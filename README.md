# Etsy Taxonomy Dataset

A structured snapshot of Etsy's seller taxonomy — categories, attributes, and all possible values.

## Files

### `categories.json`
All Etsy leaf categories. Each entry includes:
- `id` — Etsy taxonomy ID
- `name` — category name
- `path` — full path from root, e.g. `["Weddings", "Gifts & Mementos", "Wedding Favors"]`
- `type` — `physical_only`, `digital_only`, or `physical_or_digital`
- `attribute_set` — key referencing the category's attribute set in `attribute-sets.json`

### `attribute-sets.json`
Unique attribute sets (many categories share the same set). Each attribute includes:
- `id` / `property_id` — IDs needed for Etsy API calls
- `name` — specific internal name (e.g. `"Primary color"`)
- `label` — display name shown to sellers (e.g. `"Color"`)
- `group` — semantic grouping (colors, materials, styles, etc.)
- `required` / `multi_valued` / `max_values`
- `supports_variations` — whether this attribute can drive product variants
- `input_type` — `select`, `text`, `integer`, `decimal`, or `measurement`
- `scales` — sizing systems for size attributes (US, EU, UK, etc.)
- `units` — unit options for measurement attributes (cm, in, mm, etc.)
- `value_ids` — list of value IDs; look up full value objects in `values.json`
- `hint` — Etsy's guidance text for the attribute (where available)

### `values.json`
Deduplicated value lookup keyed by value ID. Each entry includes:
- `id` — value ID
- `name` — display name
- `scale` — scale system if applicable (e.g. US, EU sizes)
- `synonyms` — alternate names for AI matching (where available)

### `index.json`
Lightweight version — categories with attribute names but no values. Suitable for AI agents doing category selection and listing planning without loading the full dataset.

## Meta

Each file contains a `meta` block:
```json
{
  "taxonomy_version": "fb819b0",
  "scraped_at": "2026-05-04T03:27:56Z"
}
```

`taxonomy_version` is Etsy's internal taxonomy hash. If this changes between releases, the dataset has been updated.

## Usage example

```python
import json

categories = json.load(open("categories.json"))["data"]
attr_sets  = json.load(open("attribute-sets.json"))["data"]
values     = json.load(open("values.json"))["data"]

# Look up a category
cat = categories["1671"]
# → {"name": "Wedding Favors", "path": [...], "attribute_set": "4493e903debd"}

# Get its attributes
attrs = attr_sets[cat["attribute_set"]]
# → [{"id": 357, "name": "Material multi", "value_ids": [638, 639, ...], ...}, ...]

# Resolve values for an attribute
resolved = [values[str(vid)] for vid in attrs[0]["value_ids"]]
# → [{"id": 638, "name": "Aluminum", "synonyms": [...]}, ...]
```

