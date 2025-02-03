# Live Pricing & Availability

Our pricing and availability data is designed to be **fast, accurate, and easy to use**. Here’s what makes it stand out:

## Comprehensive & Pre-Calculated Pricing

- Pricing is provided in a **Length of Stay (LOS)** format.  
- You receive **exact prices**, including all taxes and fees, for **every possible booking** in our system.  
- No need for post-processing or price calculations—simply match the customer’s search dates to our data and return the price.  

## Optimized for Performance

- This is a **large dataset**, so you will need to maintain a **local cache** for efficient lookups.  
- Our **incremental update protocol** ensures that you only fetch changes, keeping your requests **small** and your cache  **always up-to-date**.  

## Real-Time Updates with Eventual Consistency

- Updates follow an **eventually consistent hint and query scheme**.  
- Your system **polls for changes every few minutes**, retrieving only the necessary updates.  
- This ensures your cache stays **synchronized and responsive** without excessive data transfers.  

With this approach, you get **instant, accurate pricing** without the complexity of real-time calculations.

## How Pricing & Availability Requests Work  

To optimize pricing updates, the API follows a **Changed Pricing** model, ensuring you only request data for properties with updated prices. This reduces data volume and improves efficiency.  

1. **Hint Request** – Your system periodically sends a **Hint Request** to our API, specifying the last time pricing data was retrieved.  
2. **Hint Response** – The API replies with a **Hint Response**, listing only the properties and itineraries that have had price or availability changes.  
3. **Query Request** – Based on the **Hint Response**, your system sends a **Query Request** to retrieve updated pricing and availability.  
4. **Transaction Response** – The API responds with a **Transaction Message**, providing real-time pricing and availability data for the requested properties.  

### Implementation Notes  

- **Query Frequency** – To maintain up-to-date pricing, send **Hint Requests** at regular intervals (recommended every 5 minutes).  
- **Data Efficiency** – Since **Query Requests** only target properties with updated prices, unnecessary data transfers are minimized.  

## Google References

- [Google Vacation Rentals Changed Pricing Delivery Mode](https://developers.google.com/hotels/hotel-prices/dev-guide/delivery-mode#hints).
- [Hint and Query Messages](https://developers.google.com/hotels/hotel-prices/xml-reference/queries)
- [Hint Request XSD](./xsd/hint_request.xsd)
- [Hint XSD](https://www.gstatic.com/ads-travel/hotels/api/hint.xsd)
- [Query XSD](https://www.gstatic.com/ads-travel/hotels/api/query.xsd)
- [Transaction Messages](https://developers.google.com/hotels/hotel-prices/xml-reference/transaction-messages)
- [Transaction XSD](https://www.gstatic.com/ads-travel/hotels/api/transaction.xsd)