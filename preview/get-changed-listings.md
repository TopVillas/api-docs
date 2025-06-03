# **Get Changed Listing**

Below is a technical reference for the **GET** `/api/listings/changed` endpoint. It returns a list of listing IDs that have changed since a given timestamp.

---

## Endpoint

```bash
GET /api/listings/changed
```

* **Query Parameters**

  * `live-only` (boolean, optional, default: `false`)
    If `true`, only include listing IDs for listings currently marked as live.
  * `after` (string, format: `date-time`, optional)
    Only return listing IDs that have changed after this UTC timestamp (e.g., `2025-06-01T12:00:00Z`).
    *Milliseconds can be passed to this parameter, but will be ignored.*

* **Response (200 OK)**

  * Content-Type: `application/json`
  * Body: Array of integers (int32), each representing a `listingId` that has changed.

---

## Query Parameters

* **live-only**

  * **Type:** `boolean`
  * **Default:** `false`
  * **Description:** Filter results to only include listings currently live when set to `true`. If omitted or `false`, returns all changed listings regardless of live status.

* **after**

  * **Type:** `string`
  * **Format:** `date-time` (ISO 8601)
  * **Description:** Return only the IDs of listings updated after this timestamp. If omitted, behavior defaults to returning listings changed within a system-defined window (implementation-specific).

---

## Response Schema

```json
[  
  /* integer */
  12345,
  67890,
  24680
]
```

* Each element in the array is a listing ID (`integer`, `int32`) that has been modified or updated since the specified `after` timestamp. If `live-only=true`, the list is further narrowed to those with live status.

---

## Usage Notes

* The `after` parameter accepts UTC timestamps in `YYYY-MM-DDThh:mm:ssZ` format.
* If `after` is omitted, clients will receive all listing IDs, many of which will not be live.
* Typical use case: synchronize local cache of listings by periodically polling this endpoint, then fetching full details for each ID returned via `/api/listings/{listingId}`.
