# WikiArt API Quick Reference

The WikiArt API (v2) is a simple REST service for artists, artworks, and metadata used by the WikiArt website. Access requires developer credentials, but once authenticated the endpoints are read-only and easy to use for research or gallery-style applications.

## Base URL

```
https://www.wikiart.org/en/api/2/
```

JSON responses are returned by default.

## Authentication

1. Request credentials from WikiArt (access code and secret code).
2. Exchange the codes for a session token using the `login` endpoint.
3. Send the token on subsequent requests via the `Authorization: Bearer <SessionKey>` header.

```bash
curl "https://www.wikiart.org/en/api/2/login?accessCode=<ACCESS_CODE>&secretCode=<SECRET_CODE>"
# {"sessionKey":"...","userId":"...","expires":1699999999}
```

Tokens expire; refresh by calling `login` again.

## Core Endpoints

| Endpoint | Description | Example |
| --- | --- | --- |
| `GET /login` | Exchanges developer codes for a short-lived `sessionKey`. | `curl ".../login?accessCode=abc&secretCode=def"` |
| `GET /UpdatedArtists` | Paginates recently added or edited artists. Returns `data`, `hasMore`, and `paginationToken` for the next page. | `curl -H "Authorization: Bearer $TOKEN" ".../UpdatedArtists"` |
| `GET /UpdatedArtworks` | Paginates recently added or updated artworks. Structure mirrors `UpdatedArtists`. | `curl -H "Authorization: Bearer $TOKEN" ".../UpdatedArtworks?paginationToken=..."` |
| `GET /Artist` | Retrieves full details for a specific artist using either `id` or `url` (slug). | `curl -H "Authorization: Bearer $TOKEN" ".../Artist?url=claude-monet"` |
| `GET /Painting` | Retrieves full metadata for a painting using `id` or `url` from an update feed. | `curl -H "Authorization: Bearer $TOKEN" ".../Painting?id=57726d0fedc2cb3880b49137"` |

Additional list endpoints (styles, genres, movements, schools, etc.) follow the same pattern and respect the bearer token.

## Common Response Fields

### Artist records
- `id`, `url`: Stable identifiers; `url` is the slug used on wikiart.org.
- `artistName`, `birthDayAsString`, `deathDayAsString`, `birthPlace`: Basic biography.
- `periods` and `schools`: Arrays of movement identifiers.
- `image`: Primary portrait image URL.

### Artwork records
- `id`, `url`: Stable identifiers; the `url` slug can be used in site links.
- `title`, `artistName`, `artistUrl`: Display fields that link back to an artist.
- `year`, `completitionYear`, `style`, `genre`, `period`: Temporal and classification metadata.
- `width`, `height`, `units`, `image`: Dimensions and main image URL.
- `tags`: Thematic tags applied by WikiArt.

## Typical Client Workflow

1. **Log in** to obtain a `sessionKey`.
2. **Harvest updates** using `UpdatedArtists` and `UpdatedArtworks`, following `paginationToken` until `hasMore` is false.
3. **Hydrate records** by calling `Artist` or `Painting` for the IDs or slugs returned by the update feeds.
4. **Cache results** locally to avoid repeated paging, since update endpoints are designed for incremental syncs.

### Example: Incremental artwork sync

```bash
# Acquire a token
TOKEN=$(curl -s "https://www.wikiart.org/en/api/2/login?accessCode=$ACCESS&secretCode=$SECRET" | jq -r '.sessionKey')

# First page of updates
curl -H "Authorization: Bearer $TOKEN" "https://www.wikiart.org/en/api/2/UpdatedArtworks" > page1.json
NEXT=$(jq -r '.paginationToken' page1.json)

# Follow pagination if needed
curl -H "Authorization: Bearer $TOKEN" "https://www.wikiart.org/en/api/2/UpdatedArtworks?paginationToken=$NEXT" > page2.json

# Fetch a detail record
ART_ID=$(jq -r '.data[0].id' page1.json)
curl -H "Authorization: Bearer $TOKEN" "https://www.wikiart.org/en/api/2/Painting?id=$ART_ID" | jq .title
```

## Usage Notes

- All endpoints are read-only and subject to WikiArt rate-limiting; keep batch requests polite.
- `Updated*` endpoints are the most efficient way to keep a local cache current.
- Tokens are per-session; store them securely and refresh as needed when `401` responses occur.
- The API mirrors the public WikiArt website; some fields may be nullable or absent depending on the record.

## Further References

- Developer signup and docs: <https://www.wikiart.org/en/App/Developer>
- API changelog and FAQ (when available): <https://www.wikiart.org/en/App/ApiV2/Documentation>
- WikiArt site terms: <https://www.wikiart.org/en/terms-of-use>
