# The Metropolitan Museum of Art Collection API Overview

The Metropolitan Museum of Art (The Met) provides a free public API that exposes metadata for more than 470,000 artworks in its collection. The service is RESTful and does not require authentication, making it easy to integrate into scripts, research tools, or public applications.

## Base URL

```
https://collectionapi.metmuseum.org/public/collection/v1/
```

No API key or headers are required beyond standard JSON requests.

## Core Endpoints

| Endpoint | Description | Example |
| --- | --- | --- |
| `GET /departments` | Returns the list of museum departments and their IDs. | `curl "https://collectionapi.metmuseum.org/public/collection/v1/departments"` |
| `GET /objects` | Returns all object IDs. Supports incremental harvest via `metadataDate` and filtering by `departmentIds` (pipe-separated list). | `curl "https://collectionapi.metmuseum.org/public/collection/v1/objects?departmentIds=11|12&metadataDate=2020-01-01"` |
| `GET /search` | Searches objects by full-text query. Optional filters: `q` (required), `isOnView`, `hasImages`, `medium`, `geoLocation`, `dateBegin`, `dateEnd`, `departmentId`, `artistOrCulture`. | `curl "https://collectionapi.metmuseum.org/public/collection/v1/search?q=monet&hasImages=true"` |
| `GET /objects/{objectID}` | Retrieves full metadata for a specific object ID, including constituent (artist) data and image URLs. | `curl "https://collectionapi.metmuseum.org/public/collection/v1/objects/436121"` |

## Common Object Fields

Object responses are verbose; the fields below are the ones most frequently used when building applications or dashboards:

- `objectID` (integer): Unique identifier used in other endpoints.
- `isPublicDomain` (boolean): Indicates whether images can be used without restriction.
- `primaryImage` and `primaryImageSmall` (string URLs): High-resolution and thumbnail images.
- `additionalImages` (array of URLs): Supplementary images.
- `constituents` (array of objects): Artist information (`role`, `name`, `constituentID`).
- `title`, `objectName`, `classification`, `culture`, `period`, `dynasty`, `reign`.
- `repository`, `department`, `accessionNumber`, `accessionYear`.
- `city`, `state`, `country`, `county`, `region`, `subregion`, `locale`.
- `medium`, `dimensions`, `creditLine`, `objectDate`, `objectBeginDate`, `objectEndDate`.
- `measurements` (array): Structured measurement data.
- `tags` (array): Controlled subject-matter tags.

## Typical Workflow

1. **Search for artworks** using the `/search` endpoint with query and filters to retrieve a list of matching `objectIDs`.
2. **Fetch detailed records** by calling `/objects/{objectID}` for the IDs of interest.
3. **Filter or cache results** locally to avoid repeated calls when scrolling through galleries or dashboards.

### Example: Find on-view Monet paintings with images

```bash
# Step 1: Search and collect IDs
curl "https://collectionapi.metmuseum.org/public/collection/v1/search?q=Monet&hasImages=true&isOnView=true"

# Step 2: Fetch details for a specific ID
curl "https://collectionapi.metmuseum.org/public/collection/v1/objects/436121"
```

## Notes and Best Practices

- The API is read-only; there are no create/update endpoints.
- Responses are JSON and generally lightweight, but `objects` calls can be large—prefer `search` to scope IDs and fetch object details individually.
- Data is updated regularly; use `metadataDate` on `/objects` to harvest only records changed after a given ISO date (e.g., `metadataDate=2023-01-01`).
- Images in the public domain are served via HTTPS. Respect copyright and license notes for any items not marked `isPublicDomain`.
- The service does not enforce strict rate limits, but polite client behavior (caching, pagination, and brief pauses in bulk scripts) is recommended.

## Further References

- API documentation: <https://metmuseum.github.io/>
- Open-access policy: <https://www.metmuseum.org/about-the-met/policies-and-documents/open-access>
