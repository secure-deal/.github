# Specification — Attribute Filtering

> Status: **Normative**.
> Companion to [attribute-schema-v2.md](./attribute-schema-v2.md), which defines the schema. This spec defines how those schemas drive product filtering at query time.
> Code: `app/backend/src/shared/category-attribute-filters.validator.ts`, `app/backend/src/core/product-categories/attribute-schema/*` (normalisation), `app/backend/src/modules/client/catalog/dto/get-catalog.dto.ts` (DTO), `app/backend/src/modules/admin/products/services/admin-products.service.ts` (`resolveAttributeFilters` reference pipeline).

## 1. Purpose

Allow clients (catalog) and operators (admin product list) to narrow product results by structured attribute values, using only the properties a category schema explicitly marks as filterable. The same input contract is used by the public catalog and by admin product search.

## 2. Input contract

The query parameter is `attribute_filters`, a URL-encoded JSON object, sent alongside a mandatory `category` parameter:

```
GET /api/client/catalog/products
    ?category=phones
    &attribute_filters=%7B%22brand%22%3A%5B%22apple%22%2C%22samsung%22%5D%2C%22ram_gb%22%3A8%2C%22color%22%3A%22space-gray%22%7D
```

Decoded:

```json
{
  "brand": ["apple", "samsung"],
  "ram_gb": 8,
  "color": "space-gray"
}
```

DTO: `attribute_filters?: Record<string, unknown>` (`get-catalog.dto.ts`). Validation is deferred to a category-aware validator (see §4) because the legal property set depends on the category schema.

### Rules

- `category` is REQUIRED whenever `attribute_filters` is present. The error code is `ValidationErrorCode.CATEGORY_REQUIRED_FOR_ATTRIBUTE_FILTERS`.
- Unknown properties (not declared in the category schema) ⇒ `ValidationErrorCode.UNKNOWN_ATTRIBUTE_FILTER`.
- Properties not marked `x-filterable: true` ⇒ `ValidationErrorCode.ATTRIBUTE_NOT_FILTERABLE`.
- Array values are accepted only when the property is marked `x-multiselectable: true`. Otherwise ⇒ `ValidationErrorCode.ATTRIBUTE_NOT_MULTISELECTABLE`.
- Empty arrays and `null` are dropped silently (treated as "no filter").
- Values are coerced and normalised per §5.

Current admin category editor exposes `x-multiselectable` as the explicit UI toggle for multi-value filters. `x-sortable` is not part of the current admin editing flow.

## 3. Discovery — which properties are filterable

The set of legal filter keys for a category is the union of `x-filterable: true` properties across all `attribute_schema` entries that apply to the category and its ancestors (categories inherit attributes).

Resolution algorithm (see `resolveAttributeFilters` in `admin-products.service.ts` for the reference implementation):

1. Resolve the category by slug; load its ancestor chain.
2. For every category in the chain, load its `attribute_schema` rows.
3. For each property in each schema, if `x-filterable === true`, merge it into the working set via `mergeFilterableAttributeProperty`. Conflicts (same key, different definition) are resolved by the deepest (most specific) category winning, with `allowed_values` unioned across levels by canonical `value`.
4. The merged map is the legal filter dictionary for the request.

The same algorithm powers the catalog facets endpoint (the UI needs to know which filters to render).

## 4. Validation pipeline

Implemented in `category-attribute-filters.validator.ts`. Consumers call `validateCategoryAttributeFilters({ category, attributeFilters, schemas })` which:

1. Asserts `attributeFilters` is a plain object.
2. For each key:
   - Reject if not in the merged filterable map.
   - Reject if value is an array and the property is not `x-multiselectable`.
   - Reject if any leaf value type does not match the schema (`string` / `number` / `boolean` / `color` / `date` according to property `type`).
   - If the property declares `allowed_values`, reject values whose canonical value is not present in that list.
3. Normalises the value (§5).
4. Returns a clean `Record<string, unknown>` ready to be JSON-stringified into the SQL parameter, plus an error array (empty on success).

All errors throw `ApiError(ValidationErrorCode.X, …)` and the controller maps them to a 422 validation response.

## 5. Value normalisation

| Property type | Normalisation |
| --- | --- |
| `string` with `allowed_values` | trim → lowercase → reject if canonical value is not in `allowed_values[*].value`. |
| `string` (free) | trim. |
| `number` | parse → reject `NaN` / non-finite. Range validation if `minimum` / `maximum` are present in the schema. |
| `boolean` | accept `true / false`, also accept `"true" / "false"` strings. |
| `color` | `normalizeColorValue` (`src/core/product-categories/attribute-schema`): trims, lowercases, maps known synonyms (e.g. `gray` ↔ `grey`), normalises `space-gray` etc.; if `allowed_values` is present, membership is checked against `allowed_values[*].value` after normalisation. |
| `date` | accept canonical date strings; if schema declares `min_date` / `max_date`, the value must stay within that inclusive range. |
| array of any of the above | element-wise normalisation, deduplicated. |

The normalisation step is deterministic and pure; tests assert that `normalize(normalize(x)) === normalize(x)`.

## 6. SQL application

The validated filter object is passed as a single JSONB parameter to the catalog SQL (pgtyped):

```sql
-- products.sql (excerpt, simplified)
SELECT p.*
FROM products p
WHERE p.category_id = ANY($categoryIds)
  AND p.status = 'active'
  AND (
    $attributeFilters::jsonb IS NULL
    OR p.attributes @> $attributeFilters::jsonb
    OR (
      -- multi-select: each key whose value is an array becomes an "any of" check
      ...
    )
  )
ORDER BY ...
LIMIT $limit OFFSET $offset
```

Notes:

- `attributes` on `products` is a JSONB column; matching uses GIN indexes.
- Multi-select expansion is performed in SQL via JSON operators; for arrays we use `EXISTS (SELECT 1 FROM jsonb_array_elements(...))` patterns. The client sends one JSON; the SQL splits it into containment + per-array `ANY` predicates.
- Numeric range filters (e.g. `price_min` / `price_max`) are handled outside `attribute_filters`; only equality / multi-equality is supported through this contract today.
- Date properties use exact-value matching in `attribute_filters`; schema-level `min_date` / `max_date` constrain allowed values, but do not introduce range-query syntax here.

## 7. Error codes

All under `ValidationErrorCode`:

- `CATEGORY_REQUIRED_FOR_ATTRIBUTE_FILTERS`
- `UNKNOWN_ATTRIBUTE_FILTER` — `details: { property }`
- `ATTRIBUTE_NOT_FILTERABLE` — `details: { property }`
- `ATTRIBUTE_NOT_MULTISELECTABLE` — `details: { property }`
- `INVALID_ATTRIBUTE_VALUE` — `details: { property, value, reason }`

Each maps to HTTP 422 in the standard error envelope.

## 8. Worked examples

### 8.1 Multi-select brand and exact RAM

```json
{ "brand": ["apple", "samsung"], "ram_gb": 8 }
```

`brand` must be `x-filterable + x-multiselectable`. `ram_gb` only `x-filterable` (single value). Both must be in the merged map for category `phones`.

### 8.2 Color synonyms

Input: `{ "color": "Grey" }` → normalised to `{ "color": "gray" }` (assuming `allowed_values` contains canonical `value: "gray"`). Without normalisation the search would miss products tagged `gray`.

### 8.3 Rejected — unknown key

Input: `{ "imei": "123" }` ⇒ `UNKNOWN_ATTRIBUTE_FILTER` (IMEI is per-product but not in the filter set).

### 8.4 Rejected — single value sent as array

`{ "ram_gb": [8, 16] }` against a non-multiselectable `ram_gb` ⇒ `ATTRIBUTE_NOT_MULTISELECTABLE`.

## 9. Performance

- The merged map is small (tens of keys) and is resolved once per request; cache on the request object.
- `products.attributes` requires a GIN index for `@>` containment to scale; the migration that introduces `attributes` enforces this.
- Multi-select expansions can degrade if many keys hold many values; the application caps array length at a reasonable bound (today: 50). Exceeding it ⇒ `INVALID_ATTRIBUTE_VALUE`.

## 10. Testing

Located under `app/backend/src/shared/tests/category-attribute-filters.validator.spec.ts` and the catalog/admin product service specs. Required cases:

- Missing category with non-empty filters.
- Each error code path.
- Color normalisation idempotence and synonym mapping.
- Multi-select expansion correctness (snapshot the SQL parameter).
- Schema inheritance: a property defined on the parent category is filterable on the child.

## 11. Future / out-of-scope

- Range filters inside `attribute_filters` (e.g. `{ "ram_gb": { "$gte": 8 } }`).
- Faceted aggregation endpoint exposing counts per filter value.
- Filter presets (saved searches).
- Cross-category meta-filters (e.g. "shippable to city X").
