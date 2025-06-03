# **Get Listing**

Below is a technical reference for the **GET** `/api/listings/{listingId}` endpoint. It returns a **Listing** object.

## Endpoint

```bash
GET /api/listings/{listingId}
```

- **Path Parameter**
  - `listingId` (integer, int32) — Unique identifier of the listing to retrieve.

- **Response (200 OK)**
  - **The property is online.**
  - Content-Type: application/json
  - Body: a single `Listing` object.
- **Response (404 Not Found)**
  - **The property is offline or never existed.**

## Listing Schema

```json
{
  "id": 0,
  "slug": "string",
  "name": "string",
  "type": "string",
  "maxGuests": 0,
  "bedrooms": 0,
  "bathrooms": 0.0,
  "descriptionMarkdown": "string",
  "bookingPolicy": { /* BookingPolicy */ },
  "amenities": [ "string", ... ],
  "rooms": [ /* Room */, ... ],
  "images": [ /* Image */, ... ],
  "reviews": [ /* Review */, ... ],
  "address": { /* Address */ }
}
```

Below is a detailed description of each property in **Listing**, including any nested objects.

### Top-Level Properties

- **id** `(integer, int32)`
  Unique numeric identifier for this listing.

- **slug** `(string)`
  URL-friendly identifier (e.g., `"beach-villa-123"`) that is used by the Top Villas website.

- **name** `(string)`
  Human-readable name of the property (e.g., `"Oceanfront Villa"`).

- **type** `(string)`
  The booking.com [Property Class Type](https://developers.booking.com/connectivity/docs/codes-pct) code of the property.

- **maxGuests** `(integer, int32)`
  Maximum number of guests allowed to stay in the property.

- **bedrooms** `(integer, int32)`
  Number of bedrooms in the property.

- **bathrooms** `(number, double)`
  Number of bathrooms; can include fractional values (e.g., `2.5` indicates two full baths plus one half-bath).

- **descriptionMarkdown** `(string)`
  Detailed description formatted in Markdown.

- **bookingPolicy** `(BookingPolicy)`
  Nested object defining check-in/check-out times, deposit percentage, cancellation flexibility, etc.

- **amenities** `(array of string)`
  The booking.com [Room Amenity Type](https://developers.booking.com/connectivity/docs/codes-rma) and [Hotel Amenity Type](https://developers.booking.com/connectivity/docs/codes-hac) codes that describe the property amenities. In addition to the extended Booking.com amenity codes there are a number of extra codes, prefixed with `TTV`, that describe amenities that are not listed by the Booking.com but are important to the Top Villas portfolio.

    | OTA Code | Display Name               |
    | -------- | -------------------------- |
    | TTV8001  | Indoor Basketball Court    |
    | TTV8002  | Outdoor Basketball Court   |
    | TTV8003  | Home Theater               |
    | TTV8004  | Themed Bedrooms            |
    | TTV8005  | Private Chef               |
    | TTV8006  | Home Gym                   |
    | TTV8007  | Resort Membership Included |
    | TTV8008  | Shared Pool Only           |
    | TTV8009  | Fully Staffed              |
    | TTV8010  | Beach Volley Court         |
    | TTV8011  | Pets Allowed               |
    | TTV8012  | Pets Considered            |
    | TTV8013  | Over 12s Only              |
    | TTV8014  | Driver                     |
    | TTV8015  | Outdoor Kitchen            |
    | TTV8016  | Resort                     |
    | TTV8017  | Encore Resort Membership   |
    | TTV8018  | Reunion Resort Membership  |

- **rooms** `(array of Room)`
  Array of room objects, each describing a named room and its beds.

- **images** `(array of Image)`
  Array of image objects with URL and ordering.

- **reviews** `(array of Review)`
  Array of review objects left by past guests.

- **address** `(Address, nullable)`
  Nested object containing full address details (street lines, city, region, country, postal code, and geolocation). Although this field is nullable, all of our live villas have verified addresses.

---

### Nested Schemas

#### BookingPolicy

```json
{
  "checkIn": "HH:mm:ss",
  "checkOut": "HH:mm:ss",
  "depositPercentage": 0.0,
  "balanceDueDaysBeforeArrival": 0,
  "flexCancellation": 0
}
```

- **checkIn** `(string, format: time)`
  Standard check-in time (e.g., `"15:00:00"` for 3 PM).

- **checkOut** `(string, format: time)`
  Standard check-out time (e.g., `"10:00:00"` for 10 AM).

- **depositPercentage** `(number, double)`
  A decimal percentage of the total cost due upfront as a deposit (e.g., `0.25` means 25% deposit).

- **balanceDueDaysBeforeArrival** `(integer, int32)`
  Number of days prior to arrival by which the remaining balance must be paid (e.g., `60` means balance due 60 days before check-in).

- **flexCancellation** `(integer, int32, nullable)`
  If present, number of days before check-in when a flexible cancellation policy applies. `null` indicates no flexible cancellation window. Flexible cancellation indicates that the property can be cancelled according to the terms of our [flexible cancellation policy](https://www.thetopvillas.com/en_gb/flex).

---

#### Room

```json
{
  "roomName": "string",
  "beds": [ /* Bed */ ]
}
```

- **roomName** `(string)`
  Human-readable name of the room (e.g., `"Master Bedroom"`, `"Guest Bedroom"`). May be `null`.

- **beds** `(array of Bed)`
  Collection of bed objects within this room. Each element is a **Bed**.

---

#### Bed

```json
{
  "code": "string",
  "count": 0
}
```

- **code** `(string)`
  The Booking.com [Bed Type Code](https://developers.booking.com/connectivity/docs/codes-rma#ota-bed-types) that denotes the type of the bed.

- **count** `(integer, int32)`
  Number of beds of this type (e.g., `2` for two queens).

---

#### Image

```json
{
  "url": "string",
  "order": 0,
  "labels": [ "string", ... ]
}
```

- **url** `(string)`
  Publicly accessible URL of the image.

- **order** `(integer, int32)`
  Integer indicating display priority: lower values appear first and the first image is the title image. If there is a floor-plan, then it will be the final image.

- **labels** `(array of string)`
  An array of *at most one* label from the list below. The images have been tagged using a pre-trained image recognition model. As the tags are AI generated they are occasionally wrong, but for the most part they are pretty reliable.

    | Image Tag          |
    | ------------------ |
    | Bathroom           |
    | Beach              |
    | Bedroom            |
    | Floorplan          |
    | Games Room         |
    | Garden             |
    | Golf               |
    | Gym                |
    | Hallway            |
    | Home Office        |
    | Home Theatre       |
    | Indoor Dining      |
    | Kitchen            |
    | Laundry Facilities |
    | Living Room        |
    | Outdoor Dining     |
    | Outdoor Living     |
    | Outdoor Sports     |
    | Play Area          |
    | Pool               |
    | Property Exterior  |
    | Water Play         |

---

#### Review

These reviews have been submitted by our past guests through our post-stay feedback system.

```json
{
  "reviewId": 0,
  "reviewDate": "YYYY-MM-DD",
  "checkIn": "YYYY-MM-DD",
  "nights": 0,
  "reviewerFirstName": "string",
  "reviewerLastName": "string",
  "reviewerDisplayName": "string",
  "title": "string",
  "content": "string",
  "rating": 0,
  "ratingCleanliness": 0,
  "ratingComfort": 0,
  "ratingFacilities": 0,
  "ratingService": 0,
  "ratingValue": 0
}
```

- **reviewId** `(integer, int32)`
  Unique numeric ID for this review.

- **reviewDate** `(string, format: date)`
  Date when the review was posted (e.g., `"2025-05-15"`).

- **checkIn** `(string, format: date)`
  Date when the reviewer checked in (e.g., `"2025-05-01"`).

- **nights** `(integer, int32)`
  Number of nights the reviewer stayed.

- **reviewerFirstName** `(string)`
  First name of the reviewer.

- **reviewerLastName** `(string)`
  Last name of the reviewer.

- **reviewerDisplayName** `(string)`
  Public display name (e.g., `"JaneD123"`).

- **title** `(string)`
  Short title or headline for the review.

- **content** `(string, nullable)`
  Full text content of the review.

- **rating** `(integer, int32)`
  Overall rating on a scale (e.g., 1–5).

- **ratingCleanliness** `(integer, int32)`
  Sub-rating for cleanliness (scale 1–5).

- **ratingComfort** `(integer, int32, nullable)`
  Sub-rating for comfort (scale 1–5).

- **ratingFacilities** `(integer, int32, nullable)`
  Sub-rating for facilities (scale 1–5).

- **ratingService** `(integer, int32, nullable)`
  Sub-rating for service (scale 1–5).

- **ratingValue** `(integer, int32, nullable)`
  Sub-rating for value for money (scale 1–5).

---

#### Address

```json
{
  "line1": "string",
  "line2": "string",
  "locality": "string",
  "region": "string",
  "country": "string",
  "postcode": "string",
  "lat": 0.0,
  "lng": 0.0,
  "googlePlaceId": "string"
}
```

- **line1** `(string)`
  Primary street address line (e.g., `"530 Muirfield Loop"`).

- **line2** `(string, nullable)`
  Secondary street line (e.g., apartment or suite). May be `null`.

- **locality** `(string)`
  City or town (e.g., `"Kissimmee"`).

- **region** `(string)`
  State, province, or region (e.g., `"FL"`).

- **country** `(string)`
  Two-letter ISO country code (e.g., `"US"`).

- **postcode** `(string)`
  Postal code (e.g., `"34747"`).

- **lat** `(number, double)`
  Latitude coordinate for this address.

- **lng** `(number, double)`
  Longitude coordinate for this address.

- **googlePlaceId** `(string, nullable)`
  Google Places identifier for this address. May be `null`.

---

### Example Response

```json
{
    "id": 13708,
    "slug": "reunion-resort-4",
    "locationId": 97,
    "name": "Reunion Resort 4",
    "type": "Villa",
    "maxGuests": 24,
    "bedrooms": 11,
    "bathrooms": 10,
    "descriptionMarkdown": "[//]: # (BEGIN summary)\n\nThis expansive vacation ... - 27 miles\n* Beaches - 75 miles\n\n[//]: # (END description)",
    "searchRank": 1,
    "agentDiscountPercentage": 0.05,
    "lat": 28.28627586364746,
    "lng": -81.59989929199219,
    "bookingPolicy": {
        "checkIn": "16:00:00",
        "checkOut": "10:00:00",
        "depositPercentage": 0.2000,
        "balanceDueDaysBeforeArrival": 60,
        "flexCancellation": null
    },
    "amenities": [
        "RMA5110",
        "HAC5025",
        "TTV8016",
        "TTV8004",
        "TTV8007",
        "TTV8018",
        "RMA5099",
        "HAC0044",
        "TTV8006",
        "TTV8003",
        "HAC0005",
        "TTV8015",
        "HAC0230",
        "HAC0055",
        "HAC0079",
        "RMA5017",
        "HAC0066",
        "HAC0083",
        "HAC5158"
    ],
    "rooms": [
        {
            "roomName": "Bedroom 1",
            "beds": [
                {
                    "code": "RMA58",
                    "count": 1
                }
            ]
        },
        {
            "roomName": "Bedroom 10",
            "beds": [
                {
                    "code": "RMA6032",
                    "count": 1
                }
            ]
        },
        {
            "roomName": "Bedroom 11",
            "beds": [
                {
                    "code": "RMA6032",
                    "count": 2
                }
            ]
        },
        {
            "roomName": "Bedroom 2",
            "beds": [
                {
                    "code": "RMA58",
                    "count": 1
                }
            ]
        },
        {
            "roomName": "Bedroom 3",
            "beds": [
                {
                    "code": "RMA58",
                    "count": 1
                }
            ]
        },
        {
            "roomName": "Bedroom 4",
            "beds": [
                {
                    "code": "RMA58",
                    "count": 1
                }
            ]
        },
        {
            "roomName": "Bedroom 5",
            "beds": [
                {
                    "code": "RMA58",
                    "count": 1
                }
            ]
        },
        {
            "roomName": "Bedroom 6",
            "beds": [
                {
                    "code": "RMA58",
                    "count": 1
                }
            ]
        },
        {
            "roomName": "Bedroom 7",
            "beds": [
                {
                    "code": "RMA58",
                    "count": 1
                }
            ]
        },
        {
            "roomName": "Bedroom 8",
            "beds": [
                {
                    "code": "RMA58",
                    "count": 1
                }
            ]
        },
        {
            "roomName": "Bedroom 9",
            "beds": [
                {
                    "code": "RMA58",
                    "count": 1
                }
            ]
        }
    ],
    "bedsWithoutRooms": [],
    "images": [
        {
            "url": "https://img.thetopvillas.com/13708/crop/1000x667/65ca043e729da.jpg",
            "order": 1,
            "labels": [
                "Property Exterior"
            ]
        },
        {
            "url": "https://img.thetopvillas.com/13708/crop/1000x667/65e1bb9e571ec.jpg",
            "order": 2,
            "labels": [
                "Home Theatre"
            ]
        },
        {
            "url": "https://img.thetopvillas.com/13708/crop/1000x667/65ca040292833.jpg",
            "order": 3,
            "labels": [
                "Games Room"
            ]
        },
        ...
        {
            "url": "https://img.thetopvillas.com/13708/crop/1000x667/65ca03a3586ff.jpg",
            "order": 106,
            "labels": [
                "Property Exterior"
            ]
        }
    ],
    "reviews": [
        {
            "reviewId": 35873,
            "reviewDate": "2024-09-11",
            "checkIn": "2024-08-26",
            "nights": 6,
            "reviewerFirstName": "karen",
            "reviewerLastName": "ford",
            "reviewerDisplayName": "karen ford",
            "title": "Family vacation",
            "content": "My family and I enjoyed our time at Reunion 4. ... There were 24 of us and the house never seemed crowded. ",
            "rating": 5,
            "ratingCleanliness": 5,
            "ratingComfort": 5,
            "ratingFacilities": 5,
            "ratingService": 5,
            "ratingValue": 5
        },
        {
            "reviewId": 32273,
            "reviewDate": "2024-03-05",
            "checkIn": "2024-02-25",
            "nights": 3,
            "reviewerFirstName": "Deborah",
            "reviewerLastName": "Sheridan",
            "reviewerDisplayName": "Deborah Sheridan",
            "title": "5 star review",
            "content": "Our stay in the villa at Reunion was ... 5 stars all the way!",
            "rating": 5,
            "ratingCleanliness": 5,
            "ratingComfort": 5,
            "ratingFacilities": 5,
            "ratingService": 5,
            "ratingValue": 5
        },
        {
            "reviewId": 25657,
            "reviewDate": "2021-09-02",
            "checkIn": "2021-07-16",
            "nights": 11,
            "reviewerFirstName": "Larry",
            "reviewerLastName": "Gray",
            "reviewerDisplayName": "Larry Gray",
            "title": "Nice property!",
            "content": "The property is simply beautiful! It's ... outside kitchen. ",
            "rating": 5,
            "ratingCleanliness": 5,
            "ratingComfort": 5,
            "ratingFacilities": 5,
            "ratingService": 5,
            "ratingValue": 5
        },
        {
            "reviewId": 22297,
            "reviewDate": "2020-11-17",
            "checkIn": "2020-10-23",
            "nights": 4,
            "reviewerFirstName": "Kyle",
            "reviewerLastName": "Edwards",
            "reviewerDisplayName": "Kyle Edwards",
            "title": "Outstanding",
            "content": "The best vacation stay ever... everything was way more that what we expected. ",
            "rating": 5,
            "ratingCleanliness": 5,
            "ratingComfort": 5,
            "ratingFacilities": 5,
            "ratingService": 5,
            "ratingValue": 5
        },
        {
            "reviewId": 21685,
            "reviewDate": "2020-07-13",
            "checkIn": "2020-06-28",
            "nights": 4,
            "reviewerFirstName": "Devan",
            "reviewerLastName": "Hudson",
            "reviewerDisplayName": "Devan Hudson",
            "title": "AMAZING HOME",
            "content": "The pictures don't do this home ... This home is HUGE!! ",
            "rating": 5,
            "ratingCleanliness": 5,
            "ratingComfort": 5,
            "ratingFacilities": 5,
            "ratingService": 5,
            "ratingValue": 5
        },
        {
            "reviewId": 20445,
            "reviewDate": "2020-01-09",
            "checkIn": "2019-12-26",
            "nights": 10,
            "reviewerFirstName": "John ",
            "reviewerLastName": "Lang",
            "reviewerDisplayName": "John  Lang",
            "title": "Beautiful Home ",
            "content": "My wife and I and our ... The Home is Definitely a 5 star Home!!",
            "rating": 5,
            "ratingCleanliness": 5,
            "ratingComfort": 5,
            "ratingFacilities": 5,
            "ratingService": 5,
            "ratingValue": 5
        }
    ],
    "address": {
        "line1": "530 Muirfield Loop",
        "line2": null,
        "locality": "Kissimmee",
        "region": "FL",
        "country": "US",
        "postcode": "34747",
        "lat": 28.28627586364746,
        "lng": -81.59989929199219,
        "googlePlaceId": "ChIJSycDM2N53YgRRqMjoy8GI58"
    }
}
```
