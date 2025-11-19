# Burundi Map JSON – Technical Guide

## Overview

This file, `burundi-map.json`, encodes an administrative map of Burundi. It is designed to be a **pure data source** for applications that need to:

- **Look up locations** (province, commune, zone, colline/quarter).
- **Drive UI components** such as cascading dropdowns or search filters.
- **Attach metadata** (IDs, coordinates, additional attributes) in a predictable structure.

The data is hierarchical and human-readable, making it easy to consume from web, mobile, or backend code.

---

## Data Model

Top-level structure:

```jsonc
{
  "country": "Burundi",
  "provinces": [
    {
      "name": "…",
      "capital": "…",
      "communes": [
        {
          "name": "…",
          "zones": [
            {
              "name": "…",
              "collines_quarters": ["…", "…"]
            }
          ]
        }
      ]
    }
  ]
}
```

### Fields

- **`country`**
  - Type: `string`
  - Always set to `"Burundi"`.

- **`provinces`**
  - Type: `Array<Province>`
  - Each entry describes one administrative province.

- **`Province` object**
  - **`name`**: `string`
    - Province name in uppercase (e.g. `"BUHUMUZA"`, `"BUJUMBURA"`).
  - **`capital`**: `string`
    - Capital city of the province (e.g. `"Cankuzo"`, `"Bujumbura"`).
  - **`communes`**: `Array<Commune>`
    - Communes belonging to the province.

- **`Commune` object**
  - **`name`**: `string`
    - Commune name (e.g. `"Butaganzwa"`, `"Cankuzo"`, `"Makamba"`).
  - **`zones`**: `Array<Zone>`
    - Administrative zones inside the commune.

- **`Zone` object**
  - **`name`**: `string`
    - Zone name.
  - **`collines_quarters`**: `string[]`
    - List of **collines** (rural hills / localities) or **urban quarters** for that zone.
    - Urban quarters are usually prefixed with `"Quartier "`.

---

## Design & Conventions

- **Hierarchical only**
  - No IDs or codes are embedded; the hierarchy is **purely name-based**: `province > commune > zone > colline/quarter`.
- **Strings only for leaf nodes**
  - `collines_quarters` is a simple array of strings to keep the dataset light and easy to search.
- **No geometry**
  - There are **no coordinates or polygons** in this file. It is meant as an **administrative index**, not as GeoJSON.
- **Naming**
  - Province `name` is uppercase, while communes/zones/collines use conventional casing.
  - Some province names may differ from current official spellings if the source was historical or domain‑specific.

---

## Typical Usage Patterns

### 1. Cascading selection (frontend)

Use the hierarchy to build a 4-level selector:

- **Level 1**: Province (`provinces[i].name`)
- **Level 2**: Commune (`provinces[i].communes[j].name`)
- **Level 3**: Zone (`provinces[i].communes[j].zones[k].name`)
- **Level 4**: Colline / Quarter (`provinces[i].communes[j].zones[k].collines_quarters[l]`)

You can filter arrays on the client side based on a selected parent.

### 2. Backend lookups

On the server side you can:

- Validate that a submitted location exists in the hierarchy.
- Normalize / canonicalize user input against known names.
- Attach additional metadata (IDs, external codes, GPS, etc.) in your own database keyed by province/commune/zone/colline names.

---

## Extending the Schema

If you need to enrich the data, the recommended pattern is to **add optional fields** without breaking existing consumers:

- At **province** level:
  - `code`: ISO‑like or internal code.
- At **commune / zone** level:
  - `code`, `population`, `center_coordinates`.
- At **colline / quarter** level:
  - Instead of only strings, allow objects:
    ```jsonc
    "collines_quarters": [
      { "name": "Ruyaga", "code": "RUY001", "lat": -3.123, "lng": 29.456 },
      "LegacyStringStillAllowed"
    ]
    ```

When changing the structure, keep backwards compatibility where possible (e.g. support both strings and objects for `collines_quarters`).

---

## Integration Tips

- **Parsing**
  - Any JSON-compliant runtime can load the file.
  - Keep it as a static asset (e.g. in `public/` or similar) for front‑end apps, or as a config/data file for backends.
- **Search**
  - For quick text search, flatten the hierarchy into a list of full paths like:
    - `"BUJUMBURA / Mukaza / Mukaza / Quartier Rohero"`.
- **Localization**
  - If you need multiple languages, keep this file as the **canonical data**, and maintain separate translation maps keyed by the original names.

---

## Maintenance

- When updating:
  - Preserve the overall structure (`country`, `provinces`, `communes`, `zones`, `collines_quarters`).
  - Ensure the JSON stays valid (no trailing commas, matching brackets).
- Consider adding automated tests/validators in your project that:
  - Walk the hierarchy and assert required fields exist.
  - Enforce uniqueness of `(province, commune, zone, colline/quarter)` combinations where appropriate.
