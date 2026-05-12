# Catalog, search, and filtering

## Main catalog routes

| Method | Route |
| --- | --- |
| `POST` | `/api/client/catalog` |
| `GET` | `/api/client/catalog/:slug` |
| `GET` | `/api/client/catalog/:slug/reviews` |
| `GET` | `/api/client/catalog/:slug/similar` |

## Categories and attribute filters

| Method | Route |
| --- | --- |
| `GET` | `/api/client/categories/tree` |
| `GET` | `/api/client/categories/flat-list` |
| `GET` | `/api/client/categories/:slug/children` |
| `GET` | `/api/client/categories/:slug/path` |
| `GET` | `/api/client/categories/:slug/attributes` |

`attribute_filters` is passed to catalog as a JSON object.

Example:

```json
{
  "page": 1,
  "limit": 12,
  "category_slug": "smartphones",
  "search": "iphone",
  "min_price": 100,
  "max_price": 1000,
  "sort": "price_asc",
  "attribute_filters": {
    "brand": ["apple"],
    "storage": ["128gb", "256gb"]
  }
}
```

## Search helpers

| Method | Route | Notes |
| --- | --- | --- |
| `GET` | `/api/client/catalog/search/suggestions` | Authenticated recent/suggestion helper. |
| `POST` | `/api/client/catalog/search/recent` | Save recent search. |
| `DELETE` | `/api/client/catalog/search/recent` | Clear recent searches. |

## Sort values

- `newest`
- `price_asc`
- `price_desc`
- `title`
- `popular`
- `recommended`

## Validation notes

- `category_slug` and `merchant_slug` must be lowercase slug strings.
- `limit` is capped at `100`.
- `min_price` and `max_price` must be non-negative numbers.
