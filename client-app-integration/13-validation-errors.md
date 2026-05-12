# Validation and error responses

## Generic error shape

Business and validation errors follow the `ApiErrorResponse` shape:

```json
{
  "statusCode": 422,
  "statusKey": 422001,
  "message": "",
  "error": "",
  "field": "phone",
  "constraint": "matches",
  "description": "Optional backend description"
}
```

## Fields

| Field | Meaning |
| --- | --- |
| `statusCode` | HTTP status code. |
| `statusKey` | Stable project-specific error code. |
| `field` | Optional failing request field. |
| `constraint` | Optional validator/business rule key. |
| `description` | Optional backend explanation. |

## Common statuses

| Status | Meaning |
| --- | --- |
| `400` | malformed request |
| `401` | unauthenticated / expired session |
| `403` | forbidden for current user or state |
| `404` | entity not found |
| `409` | conflict with current business state |
| `415` | unsupported media type |
| `422` | validation or precondition failure |
| `429` | throttled |
| `500` | server-side fault |

## Common buyer-side examples

- invalid phone format on login;
- invalid UUID for `delivery_address_id` or `order_item_id`;
- trying to review an ineligible order item;
- trying to read or mutate an entity that does not belong to the current buyer;
- opening a dispute in an invalid order state.

## Frontend handling

1. Prefer inline field messages when `field` is present.
2. Use generic banners/toasts for non-field business errors.
3. Preserve `statusKey` in logs and telemetry.
4. Avoid hard-coding backend prose; key off status and known error codes where possible.
