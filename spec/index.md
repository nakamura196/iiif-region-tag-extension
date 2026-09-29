# Shared Region and Keyed Tag Extension

- **Status:** Draft (2026-09-28). Under discussion between the editors; it may change.
- **Editors:** Satoru Nakamura, Jun Ogawa
- **Base specification:** [IIIF Presentation API 4.0](https://iiif.io/api/presentation/4.0/) (4.0.0-draft), which builds on the [W3C Web Annotation Data Model](https://www.w3.org/TR/annotation-model/)

The key words MUST, MUST NOT, SHOULD, SHOULD NOT and MAY are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

## 1. Introduction

Scholarly annotation of 2D images and 3D scenes often treats the *place* being annotated as a first-class entity. Several people may annotate the same region, each with a different interpretation. The Presentation API already lets each Annotation carry its own Specific Resource as its target, but it does not say what a client should do when two Annotations point at the same Specific Resource.

Annotators also want to tag an Annotation with key–value pairs (for example, *status: Roman citizen*). The Web Annotation Data Model supports tags through the `tagging` purpose, but a tag has only a value; it has no place for the key.

This extension defines how to read both patterns. **It introduces no new properties, classes or values.** It only assigns meaning to structures that the base specifications already allow. A client that does not implement this extension still shows every Annotation correctly, one by one.

### 1.1 What is standard and what is defined here

| Feature | Defined by | Status |
|---|---|---|
| `id` on a Specific Resource | Presentation 4.0 (required) | Standard |
| Annotations sharing one Specific Resource `id` denote one region; clients group them | This extension, §3 | **Extension** |
| Each sharing Annotation repeats identical `source` and `selector` | This extension, §3.1 | **Extension** |
| `body` as an array; tag bodies are TextualBody with `purpose: "tagging"` | W3C Web Annotation | Standard |
| `tagging` in the Annotation's `motivation` array | W3C Web Annotation, Presentation 4.0 | Standard |
| `label` (language map) on a TextualBody | Presentation 4.0 | Standard |
| The `label` of a tagging body is the tag's key | This extension, §4 | **Extension** |
| Declaring use of the extension in `@context` | This extension, §2 (context defines no terms) | **Extension** |
| `creator` and `created` on an Annotation | W3C Web Annotation | Standard (recommended in §5) |

## 2. Declaring the extension

A Manifest that follows this extension SHOULD declare it by adding the context URI of this extension to its top-level `@context`, before the Presentation API context, as Presentation 4.0 requires for extensions:

```json
{
  "@context": [
    "https://nakamura196.github.io/iiif-region-tag-extension/context.json",
    "http://iiif.io/api/presentation/4/context.json"
  ]
}
```

The context document defines no terms, because this extension introduces none. Its only role is to mark that the rules below apply.

- A client SHOULD apply §3 and §4 when the context is present.
- A client MAY apply them when it is absent, for example to Presentation 3.0 Manifests from a publisher known to follow them. Grouping by `id` is safe even then, because Specific Resources with the same URI are, by definition, the same resource.

## 3. Shared regions

A **shared region** is a Specific Resource whose `id` appears as the `target` of more than one Annotation.

Presentation 4.0 requires every Specific Resource to have an HTTP(S) URI in `id`. This extension adds one rule: **Annotations whose targets have the same `id` target the same region.**

### 3.1 Requirements for publishers

- A publisher that wants several Annotations to share a region MUST give their targets the same `id`.
- A publisher MUST NOT give the same `id` to Specific Resources that describe different regions.
- Each Annotation MUST embed the full Specific Resource, including `source` and `selector`. It MUST NOT refer to the region by `id` alone. This keeps every Annotation valid and renderable in clients that do not implement this extension.
- The embedded `source` and `selector` of Specific Resources that share an `id` MUST be identical.
- A publisher SHOULD use a URI that identifies the region independently of any single Annotation, for example `https://example.org/iiif/1/region/r1`.

### 3.2 Requirements for clients

- A client SHOULD present Annotations whose targets share an `id` as a group attached to a single region. For example, a 3D viewer draws one marker and lists the grouped Annotations when the marker is selected.
- A client SHOULD draw the region from the `selector` of the first Annotation of the group, in document order.
- If the embedded selectors of a group differ, the client MAY warn the user. It SHOULD still group the Annotations.
- A client MUST NOT group Annotations only because their selectors have equal values. Grouping is by `id` alone.

## 4. Keyed tags

A **keyed tag** is a TextualBody with `purpose` set to `tagging` and a `label`. The `value` is the tag's value. The `label` is the tag's key: the axis or category the value belongs to.

```json
{
  "type": "TextualBody",
  "purpose": "tagging",
  "value": "Roman citizen",
  "label": { "en": [ "status" ] }
}
```

### 4.1 Requirements for publishers

- A keyed tag MUST put only the value in `value`. It MUST NOT join the key and the value into one string, such as `"status: Roman citizen"`. The value itself may contain a colon.
- The `label` MUST be a language map, as required by Presentation 4.0. Use the `none` language when the key has no language.
- An Annotation that carries keyed tags SHOULD include `tagging` in its `motivation`.
- The descriptive text of the Annotation, if any, SHOULD be the first item of `body`, without `purpose: "tagging"`.

### 4.2 Requirements for clients

- A client SHOULD render a tagging body that has a `label` as a key–value pair, for example `status: Roman citizen`.
- A client SHOULD render a tagging body without a `label` as a plain tag showing its `value`.
- A client MUST NOT treat a tagging body as the descriptive text of the Annotation.

## 5. Provenance (informative)

When one region collects Annotations from different people or projects, users need to tell them apart. The Web Annotation Data Model already defines `creator` and `created` on an Annotation. Publishers SHOULD use them rather than `metadata`. This extension does not define anything further.

## 6. Compatibility with Presentation 3.0

In Presentation 3.0, and in the Web Annotation Data Model, `id` on a Specific Resource is optional. A Presentation 3.0 publisher that wants to use shared regions MUST therefore add `id` to the targets that share a region. All other rules above apply unchanged.

Presentation 3.0 allows `motivation` to be a single string, following the Web Annotation Data Model. Presentation 4.0 requires an array.

## 7. Example

See [examples/manifest.json](../examples/manifest.json), a Presentation 4.0 Manifest for a 3D model of a globe. The annotation texts are illustrative. It contains:

- two Annotations by different creators on the same region `#region-uk` (§3); the second carries keyed tags and one tag without a key (§4);
- a polygon region given with a `WktSelector`;
- an Annotation that uses no feature of this extension, which every client shows as usual.

## 8. References

- IIIF Presentation API 4.0 (4.0.0-draft), Data Model: <https://iiif.io/api/presentation/4.0/model/>
- IIIF Presentation API 3.0: <https://iiif.io/api/presentation/3.0/>
- W3C Web Annotation Data Model: <https://www.w3.org/TR/annotation-model/>
