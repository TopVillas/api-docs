# Guide to Maintaining an LoS Price Cache Using the Top Villas API

## Overview

This guide describes conceptually how you could design an integration with the **Top Villas API** to incrementally update and maintain a **Length of Stay Price (LoS) cache**. Its purpose is to explain the core concepts and is neither designed to be highly efficient nor failure-resilient. Instead, it aims to make explicit how the hint and query endpoints relate to one another and how they can be used to maintain a live price cache.

A production implementation may well make use of an existing queue platform such as Amazon SQS to improve reliability and scalability. Additionally, a robust implementation would need to handle transient network failures to ensure updates are not lost due to temporary connectivity issues.

The system follows these steps:

1. **Fetch pricing updates (Hint Request)**: Periodically request hints from the API to determine which properties have updated pricing.
2. **Queue pricing updates**: Store hints in a structured database format to ensure each update is processed in order.
3. **Fetch new pricing data (Query Request)**: Send queries to the API for updated pricing data based on hints.
4. **Process transaction responses**: Store or update LoS pricing in the cache based on received API responses.

For partners wishing to integrate with our API, we are happy to provide technical guidance and assistance if required.

## Database Schema

### Hint Processing Tables

```sql
CREATE TABLE hint_request (
    hint_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    timestamp TIMESTAMP WITH TIME ZONE NOT NULL
);

CREATE TABLE hint_queue (
    hint_queue_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    hint_request_id UUID REFERENCES hint_request(hint_id),
    timestamp TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE hint_queue_item (
    hint_queue_item_id BIGSERIAL PRIMARY KEY,
    hint_queue_id UUID REFERENCES hint_queue(hint_queue_id) ON DELETE CASCADE,
    property_id INTEGER NOT NULL,
    first_date DATE NOT NULL,
    last_date DATE NOT NULL
);
```

### Query and Transaction Processing Tables

```sql
CREATE TABLE query (
    query_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    hint_queue_item_id BIGINT REFERENCES hint_queue_item(hint_queue_item_id) ON DELETE CASCADE,
    first_date DATE NOT NULL,
    last_date DATE NOT NULL,
    nights SMALLINT NOT NULL,
    timestamp TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE transaction_queue (
    transaction_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    query_id UUID REFERENCES query(query_id),
    timestamp TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    response_xml XML NOT NULL
);
```

### Length of Stay Pricing Table

```sql
CREATE TABLE los_data (
    property_id INTEGER,
    checkin_date DATE,
    nights SMALLINT,
    occupancy SMALLINT,
    base_rate NUMERIC,
    tax NUMERIC,
    other_fees NUMERIC,
    last_updated TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    transaction_id UUID REFERENCES transaction_queue(transaction_id),
    PRIMARY KEY (property_id, checkin_date, nights, occupancy)
);
```

## Hint Processing Workflow

```sql
-- Step 1: Get the last hint request timestamp and ID
VAR hint_request =
    (SELECT hint_id, timestamp
     FROM hint_request
     ORDER BY timestamp DESC
     LIMIT 1);

VAR hint_request_id = hint_request.hint_id;
VAR last_change = hint_request.timestamp;

-- Step 2: Insert the next hint request (before making the API call)
INSERT INTO hint_request (timestamp)
VALUES (CURRENT_TIMESTAMP);

-- Step 3: If there's no previous hint request, return (first-time execution case)
IF hint_request_id IS NULL THEN RETURN;

-- Step 4: Send the Hint Request to the API using the last recorded timestamp
VAR hint = POST /api/xml/hint {last_change};

-- Step 5: Insert a new batch entry into hint_queue
VAR hint_queue_id = INSERT INTO hint_queue (hint_request_id, timestamp)
    VALUES (hint_request_id, CURRENT_TIMESTAMP)
    RETURNING hint_queue_id;

-- Step 6: Insert each hint response item into hint_queue_item
FOR item IN hint.items LOOP
    INSERT INTO hint_queue_item (hint_queue_id, property_id, first_date, last_date)
    VALUES (hint_queue_id, item.property_id, item.first_date, item.last_date);
END LOOP;
```

## Query Processing Workflow

```sql
-- Execute continually
LOOP

    -- Fetch the next hint_queue_item that doesn't have a corresponding query
    VAR hint_queue_item =
        (SELECT hqi.hint_queue_item_id, hqi.property_id, hqi.first_date, hqi.last_date
         FROM hint_queue_item hqi
         LEFT JOIN query q ON hqi.hint_queue_item_id = q.hint_queue_item_id
         WHERE q.hint_queue_item_id IS NULL
         ORDER BY hqi.hint_queue_item_id
         LIMIT 1);

    -- If no pending hint_queue_item is found, sleep and retry
    IF hint_queue_item IS NULL THEN
        SLEEP(5);
        CONTINUE;
    END IF;

    -- Construct the Query Request
    VAR query_payload = {
        "FirstDate": hint_queue_item.first_date,
        "LastDate": hint_queue_item.last_date,
        "Nights": 30,
        "PropertyList": [hint_queue_item.property_id]
    };

    -- Send the Query Request to the API
    VAR transaction = POST /api/xml/query {query_payload};

    -- Insert the Query into the Database
    VAR query_id =
        INSERT INTO query (hint_queue_item_id, first_date, last_date, nights, timestamp)
        VALUES (hint_queue_item.hint_queue_item_id, hint_queue_item.first_date, hint_queue_item.last_date, 30, CURRENT_TIMESTAMP)
        RETURNING query_id;

    -- Insert the Transaction into the Database
    INSERT INTO transaction_queue (query_id, timestamp, response_xml)
    VALUES (query_id, CURRENT_TIMESTAMP, transaction.xml_response);

END LOOP;
```

## Transaction Processing Workflow (LoS Data Updates)

```sql
-- Step 1: Find the last processed transaction_id
VAR last_transaction =
    (SELECT MAX(transaction_id) FROM los_data);

-- Step 2: Fetch the next unprocessed transaction
VAR transaction =
    (SELECT transaction_id, response_xml
     FROM transaction_queue
     WHERE transaction_id > last_transaction
     ORDER BY transaction_id
     LIMIT 1);

-- If no new transaction is found, sleep and retry
IF transaction IS NULL THEN
    SLEEP(5);
    CONTINUE;
END IF;

-- Step 3: Parse the XML response to extract pricing data
VAR pricing_data = PARSE_XML(transaction.response_xml);

-- Step 4: Upsert into `los_data`
FOR item IN pricing_data LOOP
    INSERT INTO los_data (
        property_id, checkin_date, nights, occupancy, base_rate, tax, other_fees, last_updated, transaction_id
    )
    VALUES (
        item.property_id, item.checkin_date, item.nights, item.occupancy,
        item.base_rate, item.tax, item.other_fees, CURRENT_TIMESTAMP, transaction.transaction_id
    )
    ON CONFLICT (property_id, checkin_date, nights, occupancy)
    DO UPDATE SET
        base_rate = EXCLUDED.base_rate,
        tax = EXCLUDED.tax,
        other_fees = EXCLUDED.other_fees,
        last_updated = CURRENT_TIMESTAMP,
        transaction_id = EXCLUDED.transaction_id;
END LOOP;
```

## Final Thoughts

One important consideration is when a new property is added to the system. In this case, an entry should be manually added to the hint queue to request all available pricing for the new property. This ensures that its pricing and availability data are captured in the cache from the start.

Conversely, when properties are removed from the platform, all associated prices will be marked as unavailable. This means there is no technical requirement to explicitly handle property removals within the integration, as the pricing data will naturally reflect the unavailability of the property.

Under this scheme when you encounter a transaction message without a corresponding property in your system, it can be safely ignored.
