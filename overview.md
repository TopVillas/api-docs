# Integration Overview

Before integrating with the **Top Villas API**, we recommend familiarizing yourself with the following:  

- [Google Hotel Prices API](https://developers.google.com/hotels/hotel-prices)  
- [Google Vacation Rentals Extensions](https://developers.google.com/hotels/vacation-rentals/dev-guide/onboarding)  

Our API is a **subset of Google's API**, which is quickly becoming the **de facto standard** for communicating vacation rental data. Understanding Google's specifications will help ensure a smooth integration with the Top Villas API.  

The following sections are of particular relevance:

- [Hotel List XML Specification](https://developers.google.com/hotels/hotel-prices/dev-guide/hlf)
- [Vacation Rental Hotel List XML Extensions Specification](https://developers.google.com/hotels/vacation-rentals/dev-guide/vr-attributes)
- [Changed Pricing Delivery Mode](https://developers.google.com/hotels/hotel-prices/dev-guide/delivery-mode#hints)

## Authentication & Access

Authentication follows a **JWT Security Token** flow:  

- Obtain a JWT using your credentials.  
- Include the token in all API requests.  
- Refresh the token before expiration to maintain access.

[Auth Documentation](./auth.md)

## **Periodic Listing Data Synchronization**  

To maintain the highest level of accuracy and consistency in your property listings, your system should **fetch the latest listing data at least once per day**.  

This daily sync ensures your platform always presents **reliable and high-quality** property information to users.

[Listing Documentation](./listing.md)

## **Maintain a Price & Availability Cache**  

To deliver **fast and accurate pricing** while minimizing unnecessary data transfers, your system should maintain a **local cache** of price and availability data via incremental updates.  

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

[Pricing Documentation](./pricing.md)

[Integration Guide](./guide.md)
