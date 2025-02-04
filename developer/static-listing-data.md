# **Static Listing Data API**  

The **Static Listing Data API** provides access to regularly updated property listings in **XML format**, allowing you to synchronize your local database with the latest static listing data.  

## Endpoint

```bash
GET https://{api_host}/api/xml/listings
```

## Response Format

- The response is a **ZIP file** that contains one or more **XML files** in the **root directory**.  
- **Important:** The ZIP file **is not compressed** (i.e., it serves as a container but does not apply compression).  
- Each XML file follows the naming convention:
  
  ```bash
  listings_{firstId}_{lastId}.xml
  ```

  where `{firstId}` and `{lastId}` represent the range of listing IDs included in that file.  

## Response Headers

When making a request to this endpoint, expect the following headers in the response:  

| Header                  | Description |
|-------------------------|-------------|
| **Content-Type**        | `application/zip` |
| **Content-Length**      | Specifies the file size in bytes |
| **Last-Modified**       | Timestamp indicating the last update of the ZIP file |

### Update Frequency & Fetch Guidelines

- **Listings can be added, removed, or modified** in the dataset, so it is important to regularly update your local data to reflect these changes.
- The ZIP file is **periodically updated** to reflect changes in static listing data.  
- Clients should:
  - **Fetch the file at least once per day** to keep their database up to date.  
  - **Avoid fetching more often than once per hour** to prevent unnecessary API load.  

## Usage Recommendations

- **Extract the ZIP file** and process each XML file independently.  
- Use the **`Last-Modified`** header to determine if a new file is available before downloading as this will prevent you from unnecessarily processing a dataset that you have already seen.

## **Listing File Format**

The listing data files provided by the **Static Listing Data API** conform to the following XML schema:  
[https://gstatic.com/localfeed/local_feed.xsd](https://gstatic.com/localfeed/local_feed.xsd)

However, some optional fields defined in the schema are **always omitted**, while others are **always included** to maintain consistency.

### Example XML Structure

```xml
<?xml version="1.0" encoding="UTF-8"?>
<listings xmlns:xsi="xsi"
    xsi:noNamespaceSchemaLocation="schema_xsd">
  <language>en</language> 

  <listing> 
    <id>1</id>
    <name>Formosa Gardens 1</name>
    <address format="simple">
      <component name="addr1">2713 Formosa Blvd</component>
      <component name="city">Kissimmee</component>
      <component name="province">FL</component>
      <component name="postal_code">34747</component>
    </address>
    <country>US</country>
    <latitude>28.327206</latitude>
    <longitude>-81.60473</longitude>
    <location_precision>0</location_precision>
    <category>villa</category>
    <content>...</content>
  </listing> 
  ...
</listings>
```

### Address

The full physical location of the property.

- The `<address>` element **will always** have the `format="simple"` attribute.
- At a minimum, the address **will contain**:
  - **Street address** (`addr1`)
  - **City** (`city`)
  - **State/Province/Region** (`province`)
  - **Postal code** (`postal_code`)
- Additional optional components **may include**:
  - **Secondary address line** (`addr2`)
  - **Third address line** (`addr3`)

### Category

Each listing will include a **category** defining the type of property. Accepted values:

- `villa`
- `townhouse`
- `condo`

### Key Notes

- The **root element** is `<listings>`, which contains multiple `<listing>` elements.
- The **`<language>` tag** is always included and currently defaults to `en` (English).
- The **XML follows the provided schema**, but certain optional fields may be omitted to ensure consistency across all listings.

## Content Section Format

The `<content>` section contains various types of metadata related to a listing, including descriptions, reviews, images, and property attributes. This data helps provide comprehensive details about the listing.

### Structure Overview

```xml
<content>
  <text type="description">...</text>
  <review type="user">...</review>
  <image type="photo" url="..." width="..." height="...">...</image>
  <attributes>...</attributes>
</content>
```

### 1. Description (`<text>`)

The `<text>` element provides a link to the listing description along with key textual information.

- **Attributes:**
  - `type="description"` (Indicates that this element contains a property description.)
- **Child Elements:**
  - `<link>`: URL to the full property description page.
  - `<title>`: The listing's title.
  - `<body>`: The actual description text.

### 2. User Reviews (`<review>`)

The `<review>` element contains details about user reviews, including ratings and dates.

- **Attributes:**
  - `type="user"` (Specifies that the review is from a user.)
- **Child Elements:**
  - `<link>`: URL to the review page.
  - `<author>`: Name of the reviewer.
  - `<rating>`: Numerical rating (e.g., `5.0`).
  - `<body>`: The review content.
  - `<date>`: The date when the review was posted (attributes: `year`, `month`, `day`).
  - `<servicedate>`: The date of the stay (attributes: `year`, `month`, `day`).

### 3. Images (`<image>`)

The `<image>` element provides photo metadata, including a direct URL and descriptive information.

- **Attributes:**
  - `type="photo"` (Indicates the element contains an image.)
  - `url`: Direct link to the image file.
  - `width`: Image width in pixels.
  - `height`: Image height in pixels.
- **Child Elements:**
  - `<link>`: URL to the listing's page.
  - `<title>`: Description of the image.
  - `<date>`: The date when the image was added (attributes: `year`, `month`, `day`).

### 4. Property Attributes (`<attributes>`)

The `<attributes>` element contains multiple `<client_attr>` elements that describe property-specific details.

- **Child Elements (`<client_attr>`)**:
  - `name="ac"`: Indicates if the property has air conditioning (`Yes`/`No`).
  - `name="capacity"`: Maximum guest capacity.
  - `name="check_in_time"`: Standard check-in time.
  - `name="check_out_time"`: Standard check-out time.
  - `name="child_friendly"`: Indicates if the property is child-friendly.
  - `name="enhanced_cleaning_practices"`: Specifies adherence to enhanced cleaning protocols.
  - `name="golf"`: Indicates if golf amenities are available.
  - `name="home_theater"`: Specifies if the property has a home theater.
  - `name="host_language"`: Primary language spoken by the host.
  - `name="hot_tub"`: Indicates if there is a hot tub available.
  - `name="infinity_pool"`: Specifies if the property has an infinity pool.
  - `name="instant_bookable"`: Indicates if instant booking is available.
  - `name="internet_type"`: Type of internet access (`Free`, `Paid`).
  - `name="kitchen"`: Specifies if the property has a kitchen.
  - `name="minimum_days_advance_purchase"`: Minimum days required for advance booking.
  - `name="num_bathrooms"`: Number of bathrooms.
  - `name="num_bedrooms"`: Number of bedrooms.
  - `name="num_beds"`: Total number of beds.
  - `name="num_reviews"`: Total number of reviews for the property.
  - `name="outdoor_grill"`: Indicates if an outdoor grill is available.
  - `name="partner_hygiene_link"`: Link to partner cleaning guidelines.
  - `name="pool_cage"`: Indicates if the pool has a cage.
  - `name="pool_table"`: Specifies if there is a pool table.
  - `name="pool_type"`: Type of pool (`Indoors`, `Outdoors`).
  - `name="rating"`: Overall user rating of the property.
  - `name="room_type"`: Defines the type of room (`Entire Place`, `Private Room`).
  - `name="self_checkin_checkout"`: Specifies if self-check-in/out is available.
  - `name="smoking_free_property"`: Indicates if smoking is prohibited.
  - `name="square_footage"`: Property size in square feet.
  - `name="themed_bedrooms"`: Indicates if the property has themed bedrooms.
  - `name="tv"`: Specifies if the property has a TV.
  - `name="washer_dryer"`: Indicates if a washer and dryer are available.
  - `name="website"`: Direct URL to the listing’s page.

This structure ensures that all key listing information, including descriptions, user reviews, images, and property attributes, is properly organized and accessible.

### Example Listing

```xml
<content>
  <text type="description">
   <link>https://www.thetopvillas.com/formosa-gardens-1#description</link>
   <title>Formosa Gardens 1</title>
   <body>This sprawling luxury 9-bedroom villa ...</body>
  </text>
  <review type="user">
   <link>https://www.thetopvillas.com/formosa-gardens-1?reviews=1#review-30927</link>
   <author>Vay Kation</author>
   <rating>5.0</rating>
   <body>My family and I vacation ...</body>
   <date year="2023" month="7" day="25" />
   <servicedate year="2023" month="5" day="1" />
  </review>
  <review type="user">
   ..
  </review>
  <image type="photo" url="https://img.thetopvillas.com/1/crop/1920x1080/6166fb42e11e5.jpg" width="1920" height="1080">
   <link>https://www.thetopvillas.com/formosa-gardens-1</link>
   <title>Formosa Gardens 1: Property Exterior</title>
   <date year="2023" month="5" day="8" />
  </image>
  <image type="photo" url="https://img.thetopvillas.com/1/crop/1920x1080/1534408614d62c34f9dae05478387c14b7.jpg" width="1920" height="1080">
   <link>https://www.thetopvillas.com/formosa-gardens-1</link>
   <title>Formosa Gardens 1: Indoor Pool</title>
   <date year="2023" month="5" day="8" />
  </image>
  <attributes>
   <client_attr name="ac">Yes</client_attr>
   <client_attr name="capacity">26</client_attr>
   <client_attr name="check_in_time">16:00:00</client_attr>
   <client_attr name="check_out_time">10:00:00</client_attr>
   <client_attr name="child_friendly">Yes</client_attr>
   <client_attr name="enhanced_cleaning_practices">Yes</client_attr>
   <client_attr name="golf">Yes</client_attr>
   <client_attr name="home_theater">Yes</client_attr>
   <client_attr name="host_language">en</client_attr>
   <client_attr name="hot_tub">Yes</client_attr>
   <client_attr name="infinity_pool">Yes</client_attr>
   <client_attr name="instant_bookable">Yes</client_attr>
   <client_attr name="internet_type">Free</client_attr>
   <client_attr name="kitchen">Yes</client_attr>
   <client_attr name="minimum_days_advance_purchase">2</client_attr>
   <client_attr name="num_bathrooms">10.0</client_attr>
   <client_attr name="num_bedrooms">9</client_attr>
   <client_attr name="num_beds">17</client_attr>
   <client_attr name="num_reviews">15</client_attr>
   <client_attr name="outdoor_grill">Yes</client_attr>
   <client_attr name="partner_hygiene_link">https://www.thetopvillas.com/new-safer-cleaning-procedures</client_attr>
   <client_attr name="pool_cage">Yes</client_attr>
   <client_attr name="pool_table">Yes</client_attr>
   <client_attr name="pool_type">Outdoors</client_attr>
   <client_attr name="rating">5.0</client_attr>
   <client_attr name="room_type">Entire Place</client_attr>
   <client_attr name="self_checkin_checkout">Yes</client_attr>
   <client_attr name="smoking_free_property">Yes</client_attr>
   <client_attr name="square_footage">9500</client_attr>
   <client_attr name="themed_bedrooms">Yes</client_attr>
   <client_attr name="tv">Yes</client_attr>
   <client_attr name="washer_dryer">Yes</client_attr>
   <client_attr name="website">https://www.thetopvillas.com/formosa-gardens-1</client_attr>
  </attributes>
 </content>
 ```
 