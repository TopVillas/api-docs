# Integration Overview

Before you integrate with the **Top Villas API** we recommend that you familiarize yourself with the [Google Hotel APIs](https://developers.google.com/hotels/hotel-prices) and [Google Vacation Rentals Onboarding Guide](https://developers.google.com/hotels/vacation-rentals/dev-guide/onboarding). The following sections are of particular relevance:

- [Hotel List XML Specification](https://developers.google.com/hotels/hotel-prices/dev-guide/hlf)
- [Vacation Rental Hotel List XML Extensions Specification](https://developers.google.com/hotels/vacation-rentals/dev-guide/vr-attributes)
- [Changed Pricing Delivery Mode](https://developers.google.com/hotels/hotel-prices/dev-guide/delivery-mode#hints)

## Authentication & Access  

Authentication follows a **JWT Security Token** flow:  

- Obtain a JWT using your credentials.  
- Include the token in all API requests.  
- Refresh the token before expiration to maintain access.  

## **Periodic Static Data Synchronization**  

To maintain the highest level of accuracy and consistency in your property listings, your system should **fetch the latest static property data at least once per day**.  

### Why This Matters

- **Keep Listings Up to Date** – Ensure property details, descriptions, and images always reflect the latest updates.  
- **Enhance User Experience** – Display the most current amenities, room configurations, and customer reviews to potential travelers.  
- **Prevent Data Staleness** – Stay aligned with any changes in property availability, features, or descriptions without manual intervention.  

This daily sync ensures your platform always presents **reliable and high-quality** property information to users.  


## **Maintain a Price & Availability Cache**  

To deliver **fast and accurate pricing** while minimizing unnecessary data transfers, your system should maintain a **local cache** of price and availability data, updated incrementally via **Hint Requests**.  

### How It Works

- **Incremental Updates with Hints**  
  - Instead of repeatedly fetching the entire dataset, use **Hint Requests** to identify which properties have had pricing or availability changes.  
  - Only request updates for properties flagged as **Changed Pricing**, significantly reducing bandwidth usage.  

- **Continuous Synchronization**  
  - Regularly process **Hint Responses** to keep your cache aligned with the latest pricing and availability changes.  
  - This ensures that your system always reflects **real-time booking conditions** without excessive API calls.  

- **Optimized Query Efficiency**  
  - Rely on cached data for the majority of queries rather than pulling full property lists frequently.  
  - This approach enhances performance, **reduces API load**, and ensures a smoother user experience.  

By following this caching strategy, you **maximize efficiency, reduce API strain, and ensure real-time pricing accuracy**, providing a seamless experience for your users.  
