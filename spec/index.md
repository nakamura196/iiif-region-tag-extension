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
| An Image body (`type: "Image"`) in `body`, with an optional `service` and `homepage` | W3C Web Annotation, Presentation 4.0 | Standard |
| `purpose` on a body (`describing`, `linking`, `identifying`) | W3C Web Annotation | Standard |
| An Image body is shown as an image with the Annotation, not as its text; `describing` shows the place, `linking` is a related resource | This extension, §5 | **Extension** (reading rule only) |
| A body that is an external resource (a Wikidata entity, a Wikipedia article, a reference), with `purpose` | W3C Web Annotation | Standard |
| Wikidata entities are given with `identifying`, Wikipedia articles and references with `linking`; clients show them as related resources, apart from the text and the tags | This extension, §6 | **Extension** (reading rule only) |
| A reference without a web address, given as a TextualBody with `purpose: "linking"` | This extension, §6.1 | **Extension** (reading rule only) |
| How to read the related resources that the annotation editor currently writes in `seeAlso` | This extension, §7 | Informative (for clients) |
| `creator` and `created` on an Annotation | W3C Web Annotation | Standard (recommended in §8) |

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

- A client SHOULD apply §3 to §6 when the context is present. §7 is for Manifests that do not declare it.
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

## 5. Images in an annotation

Annotators often attach images to an annotation. An image may show the annotated place itself, such as a historical photograph of a building. It may also be a related resource, such as a comparable object held elsewhere. The Web Annotation Data Model already allows an Annotation to have several bodies, a body may be an Image, and each body may state its `purpose`. This extension only says how to read such bodies. **It adds no terms.**

The relationship between the image and the annotated region is given by the body's `purpose`, using values defined by the Web Annotation Data Model:

| `purpose` of the Image body | Meaning (W3C definition) | Example |
|---|---|---|
| `describing`, or none | The image describes the target: it shows the annotated place itself | A photograph of the building at this address |
| `linking` | A resource related to the target | A comparable relief in another museum |

Related resources that are not images, such as Wikidata entities, Wikipedia articles and references, follow the same pattern. They are described in §6.

```json
{
  "type": "Annotation",
  "motivation": [ "commenting" ],
  "body": [
    { "type": "TextualBody", "value": "<p>A relief of a dog carved on the stele</p>", "format": "text/html", "label": { "en": [ "Relief of a dog" ] } },
    {
      "id": "https://example.org/iiif/photo1/full/max/0/default.jpg",
      "type": "Image",
      "purpose": "describing",
      "format": "image/jpeg",
      "label": { "en": [ "Photograph of the relief (1938)" ] },
      "service": [ { "id": "https://example.org/iiif/photo1", "type": "ImageService3", "profile": "level1" } ],
      "homepage": [ { "id": "https://example.org/record/1", "type": "Text", "format": "text/html", "label": { "en": [ "Catalogue record" ] } } ]
    },
    {
      "id": "https://example.org/images/comparable-relief.jpg",
      "type": "Image",
      "purpose": "linking",
      "format": "image/jpeg",
      "label": { "en": [ "A comparable dog relief, British Museum" ] }
    },
    { "id": "http://www.wikidata.org/entity/Q144", "type": "Dataset", "purpose": "identifying" }
  ],
  "target": { "...": "a Specific Resource as in §3" }
}
```

### 5.1 Requirements for publishers

- An image attached to an annotation MUST be given as a body with `type` set to `Image` and `id` set to the URI of the image itself. It SHOULD NOT be given in `seeAlso`, which Presentation 4.0 defines for machine-readable descriptions, nor by a property that is not defined there, such as `source` on a `seeAlso` item.
- An Image body that shows the annotated place SHOULD have `purpose` set to `describing`. An Image body that is a related resource MUST have `purpose` set to `linking`.
- The body SHOULD have a `label` (language map) that identifies the image, such as its title and date.
- When the image is served by a IIIF Image API service, the body SHOULD carry it in `service`, so that clients can request a thumbnail instead of the full image.
- The page that describes the image at its holding institution SHOULD be given in `homepage` on the Image body.
- The descriptive text SHOULD remain the first item of `body` (§4.1).

### 5.2 Requirements for clients

- A client SHOULD show the Image bodies of an Annotation as images together with the Annotation, for example as thumbnails in its list entry.
- A client SHOULD present Image bodies with `purpose: "linking"` apart from those that show the place, for example under a separate heading, so that users do not take a related resource for a picture of the annotated region.
- A client MUST NOT treat an Image body as the descriptive text or as a tag of the Annotation.
- A client SHOULD use the image `service`, when present, to request a thumbnail, and SHOULD link the image to its `homepage`, when present.

## 6. Related resources

Annotators link a region to resources outside the Manifest: an entity in Wikidata that tells what the region depicts, a Wikipedia article for background, and the literature that discusses it. Each of these is a body of the Annotation with a `purpose`, as for images in §5. **This section adds no terms.**

| Resource | Body | `purpose` |
|---|---|---|
| Wikidata entity | `id`: the entity URI; `type`: `Dataset` | `identifying` |
| Wikipedia article | `id`: the article URL; `type`: `Text`; `format`: `text/html` | `linking` |
| Reference with a web address (a DOI, a catalogue record, a PDF) | `id`: that address; `type`: `Text`; `format`; `label`: the citation | `linking` |
| Reference without a web address | a TextualBody whose `value` is the citation; `format`: `text/plain` | `linking` |
| Other media: a IIIF Manifest, a video, a page of a 3D model | `id`: its address; `type`: `Manifest`, `Video`, `Model` or `Text` | `linking`, or `describing` when it shows the place itself (§5) |

`identifying` is defined by the Web Annotation Data Model for identifying "what is being depicted or described in the Target". `linking` is for "a resource related to the Target".

```json
{
  "type": "Annotation",
  "motivation": [ "commenting" ],
  "body": [
    { "type": "TextualBody", "value": "<p>A relief of a dog carved on the stele</p>", "format": "text/html", "label": { "en": [ "Relief of a dog" ] } },
    { "id": "http://www.wikidata.org/entity/Q144", "type": "Dataset", "purpose": "identifying", "label": { "en": [ "dog" ], "ja": [ "イヌ" ] } },
    { "id": "https://en.wikipedia.org/wiki/Dog", "type": "Text", "format": "text/html", "language": [ "en" ], "purpose": "linking", "label": { "en": [ "Wikipedia: Dog" ] } },
    { "id": "https://doi.org/10.0000/example", "type": "Text", "format": "text/html", "purpose": "linking", "label": { "none": [ "Author, A. (2003). An example article. Journal 1: 1–10." ] } },
    { "type": "TextualBody", "value": "Author, B. (1998). A book without a web address. Publisher.", "format": "text/plain", "purpose": "linking" }
  ],
  "target": { "...": "a Specific Resource as in §3" }
}
```

### 6.1 Requirements for publishers

- A related resource MUST be given as a body with a `purpose`. It SHOULD NOT be given in `seeAlso`. Presentation 4.0 defines `seeAlso` for a machine-readable resource, such as an RDF description, related to the resource that carries it, here the Annotation. A web page or a citation is not such a resource, and `seeAlso` cannot say how a resource relates to the region.
- A Wikidata entity SHOULD have `purpose` set to `identifying`. Its `id` SHOULD be the concept URI (`http://www.wikidata.org/entity/Q144`). The page URL (`https://www.wikidata.org/wiki/Q144`) names the page about the entity, not the entity.
- A Wikipedia article SHOULD have `purpose` set to `linking`, and SHOULD state the language of the article in `language`.
- A reference SHOULD have `purpose` set to `linking`. Its `id` SHOULD be an address that opens the work or its record: a DOI as `https://doi.org/…` where one exists, otherwise a catalogue record or a page. Its `label` SHOULD be the citation text.
- A reference that has no web address MAY be given as a TextualBody whose `value` is the citation, with `format` set to `text/plain` and `purpose` set to `linking`.
- The `id` of a body MUST be the address of the resource itself. A publisher MUST NOT use as `id` a URI that does not resolve and only names the link inside the publisher's own records. Such a URI belongs in the publisher's own data, not in the body (see §7.3).
- The descriptive text SHOULD remain the first item of `body` (§4.1).

### 6.2 Requirements for clients

- A client SHOULD show bodies with `purpose` set to `identifying` or `linking` as the related resources of the Annotation, apart from its descriptive text and its tags.
- A client MUST NOT treat such a body as the descriptive text or as a tag. This includes a TextualBody with `purpose: "linking"`, which is a citation.
- A client SHOULD show `identifying` bodies before `linking` bodies: first what the region is, then where to read more.
- A client SHOULD open the `id` of a related resource as a link. It SHOULD show a TextualBody with `purpose: "linking"` as the text of a reference, without a link.
- A client MAY fetch the label of a Wikidata entity in the reader's language when the body has no `label`.
- A client SHOULD show a resource only once when the same address is given more than once.

## 7. Reading the current output of the annotation editor (informative)

The annotation editor [3D-annotation-viewer](https://github.com/junjun7613/3D-annotation-viewer) does not yet write related resources as bodies. It writes them in the `seeAlso` of the Annotation, and it publishes a fuller description of them in RDF. This section describes that output as observed on 2026-11-01, in its source code and in the manifests it serves, and says how a client can read it into the same list as §5 and §6. It is informative. It does not make `seeAlso` a recommended form.

The editor serves a Manifest at `https://3-d-annotation-viewer.vercel.app/api/3/<key>/manifest`, where `<key>` is the URL of the original Manifest encoded in base64url. Its `@context` is Presentation 3.0, and it does not declare this extension.

### 7.1 Where the related resources are

- The `seeAlso` of each Annotation lists the resources linked to that Annotation.
- The top-level `seeAlso` of the Manifest lists the resources linked to the object as a whole, and one item with `format` set to `text/turtle`. That item points to the RDF description (`…/api/3/<key>/rdf`, Turtle, readable from other origins).

### 7.2 The items in the `seeAlso` of an Annotation

| What it is | `type` | `format` | `id` | Address of the resource | Read as (§5, §6) |
|---|---|---|---|---|---|
| Image | `Image` | `image/jpeg` | `<base>/media/<uuid>` | `source` | Image body; see 7.4 for its `purpose` |
| IIIF resource | `Manifest` | `application/ld+json` | `<base>/media/<uuid>` | `source` | `Manifest` body, `linking` |
| Video | `Video` | `text/html` | `<base>/media/<uuid>` | `source` | `Video` body, `linking` |
| 3D model on Sketchfab | `Model` | `text/html` | `<base>/media/<uuid>` | `source` (the model page) | `Text` body, `linking` |
| Wikidata entity | `Dataset` (`profile`: `https://www.wikidata.org`) | `application/ld+json` | the page URL `https://www.wikidata.org/wiki/Q…` | `id` | `Dataset` body, `identifying` |
| Wikipedia article | `Text` | `text/html` | the article URL | `id` | `Text` body, `linking` |
| Reference | `Text` | `text/plain` | `<base>/bibliography/<uuid>` | none in the Manifest; see 7.4 | TextualBody, `linking`; its `value` is the `label` |
| Reference with a PDF | `Text` | `application/pdf` | `<base>/bibliography/<uuid>` | `source` (the PDF) | `Text` body, `linking` |

Notes for clients:

- `source` is not defined on a `seeAlso` item by Presentation 3.0 or 4.0. In the editor's output it holds the address of the resource, while `id` does not resolve (7.3). A client reading this output SHOULD take the address from `source` and SHOULD NOT open `id`.
- The editor writes one Wikipedia item for each Wikidata entity that has an article, with the label `Wikipedia: <label of the entity>`.
- The label of a reference is the citation, made as `<author> (<year>). <title>`.
- Every `label` is tagged `en`, whatever its language, for example `{"en": ["イヌ"]}`. A client SHOULD NOT choose among these labels by language.

### 7.3 The `id`s under `/media/`, `/bibliography/` and `/region/`

- `<base>` is the directory of the **original** Manifest that was loaded into the editor, not the editor's own address. For an original Manifest at `https://example.org/iiif/3/87/manifest`, the image of an Annotation has the `id` `https://example.org/iiif/3/87/media/<uuid>`, and the region has `https://example.org/iiif/3/87/region/<uuid>`.
- These URIs do not resolve. The editor does not serve them, and the host of the original Manifest does not know them. All that were tried on 2026-11-01 returned 404.
- They are the same URIs as the subjects in the RDF description (7.4). They can therefore be used to look up a resource in the RDF, but not as links.
- The region `id`s still serve §3, which compares `id`s and does not dereference them.
- The RDF names the region differently: `<base>/annotation/region/<uuid>`, where the Manifest has `<base>/region/<uuid>`. To join the two, a client SHOULD use the `id` of the Annotation (`<base>/annotation/<id>`), which is the same in both.

### 7.4 What only the RDF description gives

The RDF uses CIDOC CRM, schema.org, Dublin Core, BIBO, and the editor's own vocabulary (the prefix `:` for `https://www.example.com/vocabulary/`). A client that fetches it can add the following:

- The address of a reference: `schema:url` (a page), `bibo:doi`, and `schema:contentUrl` (a PDF). With one of these, a reference can become a `Text` body with that `id`. A DOI is opened as `https://doi.org/<doi>`.
- For a Wikidata entity: a picture (`foaf:depiction`), its class in the editor's vocabulary (for example `:Object`, `:Person`), and coordinates (`geo:lat`, `geo:long`).
- How each resource relates to the Annotation, for example `<image> :concept_contextualization <annotation>` or `<annotation> :associated_with <wikidata entity>`.

The RDF does not give the address of an image file. It is only in `source` in the Manifest, so a client needs both.

The editor's relations are finer than `purpose`. The following correspondence is a **proposal** for discussion with the editor's author:

| Relation in the editor's RDF | Resource | `purpose` |
|---|---|---|
| `:depicts`, `:depicts_part`, `:documents`, `:measures`, `:reproduces`, `:illustrates` | Media | `describing` (it shows the place) |
| `:contextualizes` and its sub-properties, `:compares_with`, `:provides_typology`, `:illustrates_typology` | Media | `linking` |
| any relation, or none | Reference | `linking` |
| `:identifies`, `:depicts` | Wikidata entity | `identifying` |
| `:associated_with`, `:classified_as`, `:has_type`, `:written_in_language`, `:uses_script`, `:created_by`, `:discovered_by`, `:discovered_at` | Wikidata entity | `identifying` (§6.1). To be discussed: an entity such as the maker of the object does not tell what the region depicts |
| none (older data, written as `crm:P67_refers_to`) | any | as in §6.1; for media, `linking` |

Without the RDF, a client cannot tell whether an image shows the place or is a related resource. It SHOULD then present the image as a related resource (`linking`), so that users do not take a related resource for a picture of the annotated region (§5.2).

## 8. Provenance (informative)

When one region collects Annotations from different people or projects, users need to tell them apart. The Web Annotation Data Model already defines `creator` and `created` on an Annotation. Publishers SHOULD use them rather than `metadata`. This extension does not define anything further.

## 9. Compatibility with Presentation 3.0

In Presentation 3.0, and in the Web Annotation Data Model, `id` on a Specific Resource is optional. A Presentation 3.0 publisher that wants to use shared regions MUST therefore add `id` to the targets that share a region. All other rules above apply unchanged.

Presentation 3.0 allows `motivation` to be a single string, following the Web Annotation Data Model. Presentation 4.0 requires an array.

## 10. Example

See [examples/manifest.json](../examples/manifest.json), a Presentation 4.0 Manifest for a 3D model of a globe. The annotation texts are illustrative. It contains:

- two Annotations by different creators on the same region `#region-uk` (§3); the first links to Wikidata, Wikipedia and a reference (§6); the second carries keyed tags and one tag without a key (§4);
- a polygon region given with a `WktSelector`;
- an Annotation that uses no feature of this extension, which every client shows as usual.

## 11. References

- IIIF Presentation API 4.0 (4.0.0-draft), Data Model: <https://iiif.io/api/presentation/4.0/model/>
- IIIF Presentation API 3.0: <https://iiif.io/api/presentation/3.0/>
- W3C Web Annotation Data Model: <https://www.w3.org/TR/annotation-model/>
- Wikidata, Linked data interface (concept URIs): <https://www.wikidata.org/wiki/Wikidata:Data_access>
- 3D-annotation-viewer (the annotation editor of §7): <https://github.com/junjun7613/3D-annotation-viewer>
