# Favorites and reviews

## Favorites

Routes:

| Method | Route |
| --- | --- |
| `POST` | `/api/client/favorites/products/:slug` |
| `DELETE` | `/api/client/favorites/products/:slug` |
| `GET` | `/api/client/favorites/products` |

Notes:

- Favorites are product-slug based at the API edge.
- Favorite listing is authenticated and paginated.
- Product cards can reflect `is_favorite` if the current client session is known.

## Reviews

Routes:

| Method | Route |
| --- | --- |
| `GET` | `/api/client/catalog/:slug/reviews` |
| `POST` | `/api/client/reviews` |

Create review body:

```json
{
  "order_item_id": "uuid",
  "rating": 5,
  "title": "Great product",
  "content": "Arrived in good condition",
  "media_urls": ["https://cdn.example.com/review-1.jpg"]
}
```

## Review constraints

- `order_item_id` must be a UUID.
- `rating` must be an integer from `1` to `5`.
- `media_urls` accepts up to `10` strings.
- Review creation is tied to buyer/order ownership and business rules from the backend.

## Frontend guidance

- Keep favorites separate from reviews in state even if both live on product detail.
- Refresh product review list after successful review creation.
- For favorite toggle UX, optimistic update is fine when rollback is simple.
