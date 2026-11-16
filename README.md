# tmforum-schemas

JSON schemas that extend TM Forum entities. The TM Forum API validates an entity against the schema its
`@schemaLocation` points to, so these files are read by URL while the marketplace is running.

## Schemas

| Family | Version | File | Properties |
| --- | --- | --- | --- |
| Offering (TMF620 ProductOffering) | 1 | [`schemas/v1/offering.json`](schemas/v1/offering.json) | `externalId`, `offerKind`, `pricingLogicAlgorithm`, `relatedParty` |

[`schemas/external-id-billed.json`](schemas/external-id-billed.json) is the original path of the offering version 1. It is kept
because the offers already stored and the deployments point to it, and it always has the same content as
`schemas/v1/offering.json`: change both together and check them with `cmp`. Do not move or rename it.

## Versioning

- **The folder is the version and the file is the family**: `schemas/v<N>/<family>.json`. A folder only has the families that
  changed in that version.
- **Changes that cannot invalidate an existing entity are made in the current version**: a new optional property. An entity
  written before the change stays valid, and so does the one that was written with the change.
- **Any other change is a new version** (a new folder): removing or renaming a property, making one required, narrowing a
  type or a list of values, or changing `additionalProperties`. The TM Forum keeps the `@schemaLocation` an entity was written
  with and checks it again on every update, so such a change would break entities that were valid. The new version repeats
  everything of the previous one, it does not `$ref` it: with `additionalProperties: false` the properties of a referenced
  schema are not counted.
- **Each release is tagged** `<family>-v<version>.<revision>` (`offering-v1.1` is the second state of version 1, the one that
  added `offerKind`). A tag is the immutable address to give to a service that needs exactly that state:
  `.../refs/tags/offering-v1.1/schemas/v1/offering.json`. `refs/heads/main` follows the latest state.
- **Rolling out a change:** publish it first, then make the services that write the new property use it, then migrate the
  entities that need it. A service that writes a property before its schema is published is refused by the TM Forum.

### Making a change

1. Edit `schemas/v<N>/<family>.json` if the change is one that stays in the version, or copy it to `schemas/v<N+1>/` if not.
2. Keep `schemas/external-id-billed.json` identical to `schemas/v1/offering.json`.
3. Check it against the TM Forum: serve the file, create an entity that uses it as `@schemaLocation`, and check that valid
   values are stored, invalid ones refused and, if the property is filterable, that the filter works.
4. Merge it and tag the release.
