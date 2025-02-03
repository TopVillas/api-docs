# Top Villas Vacation Rentals API

Welcome to the **Top Villas Vacation Rentals API**. This API provides access to both **static listing data** (including property names, descriptions, images, and amenities) and **live pricing and availability data** for all of our online bookable vacation rental properties. It is built as a subset of the [Google Vacation Rentals API](https://developers.google.com/hotels/vacation-rentals/dev-guide/onboarding), ensuring compatibility with Google's broader vacation rental ecosystem.

**API credentials and endpoints are issued upon request.**

## Key Features  

- **Static Listing Data** – Provides essential property details, including names, descriptions, images, and amenities. This data should be refreshed daily to ensure accuracy.  
- **Live Pricing & Availability** – Efficiently delivers real-time pricing and availability updates across our entire property inventory.

## Integration Overview

To integrate with the **Top Villas API**, follow these key steps:  

### 1. Review Documentation & Onboarding  

- Familiarize yourself with the [Google Hotel APIs](https://developers.google.com/hotels/hotel-prices) and [Google Vacation Rentals Onboarding Guide](https://developers.google.com/hotels/vacation-rentals/dev-guide/onboarding).  

### 2. Authentication & Access  

- Authentication follows a **JWT Security Token** flow:  
  - Obtain a JWT using your credentials.  
  - Include the token in all API requests.  
  - Refresh the token before expiration to maintain access.  

### 3. Periodic Static Data Synchronization  

- Ensure your system fetches the latest static property data at least once per day.

### 4. Maintain a Price & Availability Cache  

To ensure your system always has the latest pricing and availability while minimizing data overhead, maintain a local cache that is regularly updated via **Hint Requests**. By leveraging the **Changed Pricing** model, you only retrieve updates for properties with modified prices, reducing unnecessary data transfers and improving response times.  

- **Regular Updates** – Continuously refresh your cache based on **Hint Responses** to reflect the latest price and availability changes.  
- **Efficient Queries** – Use your cached data to reduce the frequency of full property list queries, relying instead on targeted updates.

## How Pricing & Availability Requests Work  

To optimize pricing updates, the API follows a **Changed Pricing** model, ensuring you only request data for properties with updated prices. This reduces data volume and improves efficiency.  

1. **Hint Request** – Your system periodically sends a **Hint Request** to our API, specifying the last time pricing data was retrieved.  
2. **Hint Response** – The API replies with a **Hint Response**, listing only the properties and itineraries that have had price or availability changes.  
3. **Query Request** – Based on the **Hint Response**, your system sends a **Query Request** to retrieve updated pricing and availability.  
4. **Transaction Response** – The API responds with a **Transaction Message**, providing real-time pricing and availability data for the requested properties.  

### Implementation Notes  

- **Query Frequency** – To maintain up-to-date pricing, send **Hint Requests** at regular intervals (recommended every 5 minutes).  
- **Data Efficiency** – Since **Query Requests** only target properties with updated prices, unnecessary data transfers are minimized.  
- **Structured XML Responses** – Pricing data is returned in XML format, following the [Google Vacation Rentals API pricing guidelines](https://developers.google.com/hotels/hotel-prices/dev-guide/delivery-mode#hints).  
