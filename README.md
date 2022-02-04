# STAC Asset Specification <!-- omit in toc --> 

- [Overview](#overview)
- [Asset fields](#asset-fields)
  - [Additional Field Information](#additional-field-information)
    - [stac_version](#stac_version)
    - [stac_extensions](#stac_extensions)
    - [id](#id)
    - [roles](#id)
  - [Properties Object](#properties-object)
  - [Link Object](#link-object)
    - [Relation types](#relation-types)
    - [Items](#items)
- [Media Type for STAC Item](#media-type-for-stac-item)
- [Extensions](#extensions)

## Overview

This document explains the structure and content of a SpatioTemporal Asset Catalog (STAC) Asset. 
An **Asset** is a [GeoJSON](http://geojson.org/) [Feature](https://tools.ietf.org/html/rfc7946#section-3.2) 
augmented with [foreign members](https://tools.ietf.org/html/rfc7946#section-6) relevant to a 
STAC object.
An **Asset** is an object that contains a URI to data associated with the Item that can be downloaded
or streamed. 

- Examples:
  - See the [minimal example](../examples/simple-asset.json),
- [JSON Schema](json-schema/asset.json)

## Asset fields

This object describes a STAC Item. The fields `id`, `type`, `bbox`, `geometry` and `properties` are
inherited from GeoJSON.

| Field Name | Type                                                  | Description |
| ---------- | ----------------------------------------------------- | ----------- |
| type       | string                                                | **REQUIRED.** Type of the GeoJSON Object. MUST be set to `Feature`. |
| stac_version | string                                              | **REQUIRED.** The STAC version the Asset implements. |
| stac_extensions | \[string]                                        | A list of extensions the Asset implements. |
| id         | string                                                | **REQUIRED.** Provider identifier. The ID should be unique within the  [Item](../item-spec/item-spec.md) that contains the Asset. |
| links      | \[[Link Object](#link-object)]                        | **REQUIRED.** List of link objects to resources and related URLs. A link with the `rel` set to `item` is strongly recommended. |
| item | string                                                      | **REQUIRED** The `id` of the STAC Item this Asset is referenced from(see [`asset` relation type](#relation-types)). This field provides an easy way for a user to search for the Item which aggregates related Assets. Must be a non-empty string. |
| href        | string                                               | **REQUIRED.** URI to the asset object. Relative and absolute URI are both allowed. |
| title       | string                                               | The displayed title for clients and users. |
| description | string                                               | A description of the Asset providing additional details, such as how it was processed or created. [CommonMark 0.29](http://commonmark.org/) syntax MAY be used for rich text representation. |
| media_type  | string                                               | [Media type](#asset-media-type) of the asset. See the [common media types](../best-practices.md#common-media-types-in-stac) in the best practice doc for commonly used asset types. |
| roles       | \[string]                                            | The [semantic roles](#asset-roles) of the asset, similar to the use of `rel` in links. |
| geometry    | [GeoJSON Geometry Object](https://tools.ietf.org/html/rfc7946#section-3.1) \| [null](https://tools.ietf.org/html/rfc7946#section-3.2) | Defines the full footprint of the asset represented by this item, formatted according to [RFC 7946, section 3.1](https://tools.ietf.org/html/rfc7946#section-3.1). The footprint should be the default GeoJSON geometry, though additional geometries can be included. Coordinates are specified in Longitude/Latitude or Longitude/Latitude/Elevation based on [WGS 84](http://www.opengis.net/def/crs/OGC/1.3/CRS84). |
| bbox        | \[number]                                                                  | Bounding Box of the Asset |
| properties | [Properties Object](#properties-object)                                    | A dictionary of additional metadata for the Asset. |


### Additional Field Information

#### stac_version

In general, STAC versions can be mixed, but please keep the [recommended best practices](../best-practices.md#mixing-stac-versions) in mind.

#### stac_extensions

A list of extensions the Item implements.
The list consists of URLs to JSON Schema files that can be used for validation.
This list must only contain extensions that extend the Asset specification itself,
see the the 'Scope' for each of the extensions.

#### id

It is important that an Asset identifier is unique within an Item, and that the 
[Item identifier](../item-spec/item-spec.md#id) in turn is unique within a collection. Then the three can be combined to
give a globally unique identifier. Assets are *[required](#items)* to have Items.

#### roles

The `roles` field is used to describe the purpose of each asset. It is recommended to include one for every asset, to give users
a sense of why they might want to make use of the asset. There are some emerging standards that enable clients to take particular
action when they encounter particular roles, listed below. But implementors are encouraged to come up with their own terms to 
describe the role.

##### Asset Role Types

Like the Link `rel` field, the `roles` field can be given any value, however here are a few standardized role names. 

| Role Name | Description                                                                           |
| --------- | ------------------------------------------------------------------------------------- |
| thumbnail | An asset that represents a thumbnail of the Item, typically a true color image (for Items with assets in the visible wavelengths), lower-resolution (typically smaller 600x600 pixels), and typically a JPEG or PNG (suitable for display in a web browser). Multiple assets may have this purpose, but it recommended that the `type` and `roles` be unique tuples. For example, Sentinel-2 L2A provides thumbnail images in both JPEG and JPEG2000 formats, and would be distinguished by their media types. |
| overview  | An asset that represents a possibly larger view than the thumbnail of the Item, for example, a true color composite of multi-band data. |
| data      | The data itself. This is a suggestion for a common role for data files to be used in case data providers don't come up with their own names and semantics. |
| metadata  | A metadata sidecar file describing the data in this Item, for example the Landsat-8 MTL file. |

It is STRONGLY RECOMMENDED to add to each STAC Item
- a thumbnail with the role `thumbnail` for preview purposes
- one or more data file although it doesn't need to use the suggested role `data`

Note that multiple roles per asset are encouraged: pick all the ones that apply. So many should have the 'data' role, and then
another role to describe how the data is used. For more information on how to use roles see the [Asset 
Roles](../best-practices.md#asset-roles) section of the Best Practices document. It includes a [list of asset 
roles](../best-practices.md#list-of-asset-roles) that include many more ideas on roles to use. As they reach more widespread 
adoption we will include them here.

### Properties Object

Additional metadata fields can be added to the GeoJSON Object Properties. See [item-spec](https://github.com/radiantearth/stac-spec/blob/master/item-spec/item-spec.md#properties-object) for ideas.

Providers should include metadata fields that are relevant for users of STAC, but it is recommended
to [select only those necessary for search](../best-practices.md#field-selection-and-metadata-linking).
Where possible metadata fields should be mapped to the STAC Common Metadata and widely used extensions,
to enable cross-catalog search on known fields.

- [STAC Common Metadata](common-metadata.md#stac-common-metadata) - A list of fields commonly used
throughout all domains. These optional fields are included for STAC Items by default.
- [Extensions](../extensions/README.md) - Additional fields that are more specific,
such as [EO](https://github.com/stac-extensions/eo), [View](https://github.com/stac-extensions/view).
- [Custom Extensions](../extensions/README.md#extending-stac) - It is generally allowed to add custom
fields but it is recommended to add multiple fields for related values instead of a nested object,
e.g., two fields `view:azimuth` and `view:off_nadir` instead of a field `view` with an object
value containing the two fields. The convention (as used within Extensions) is for related fields 
to use a common prefix on the field names to group them, e.g. `view`. A nested data structure should
only be used when the data itself is nested, as with `eo:bands`.

### Link Object

This object describes a relationship with another entity. Data providers are advised to be liberal
with the links section, to describe things like the Item an Asset is in, or parents.
It is allowed to add additional fields such as a `title` and `type`.

| Field Name | Type   | Description |
| ---------- | ------ | ----------- |
| href       | string | **REQUIRED.** The actual link in the format of an URL. Relative and absolute links are both allowed. |
| rel        | string | **REQUIRED.** Relationship between the current document and the linked document. See chapter "Relation types" for more information. |
| type       | string | [Media type](../catalog-spec/catalog-spec.md#media-types) of the referenced entity. |
| title      | string | A human readable title to be used in rendered displays of the link. |

For a full discussion of the situations where relative and absolute links are recommended see the
['Use of links'](../best-practices.md#use-of-links) section of the STAC best practices.

#### Relation types

See [Item Relation Types](https://github.com/radiantearth/stac-spec/blob/master/item-spec/item-spec.md#relation-types)

#### Items

Assets are *required* to provide a link to a STAC Item definition.
It is important as Items provide additional information about a set of Assets, including
aggregate properties.

1. The field `item` in an Item must be filled (see section '[Asset fields](#asset-fields)'). It is the `id` of a STAC Item.
2. An Asset must also provide a link to the STAC Item using the [`item` relation type](#relation-types):
   ```js
   "links": [
     { "rel": "item", "href": "link/to/item/record.json" }
   ]
   ```

## Media Type for STAC Item

A STAC Item is a GeoJSON file ([RFC 7946](https://tools.ietf.org/html/rfc7946)), and thus should use the 
[`application/geo+json`](https://tools.ietf.org/html/rfc7946#section-12) as the [Media Type](https://en.wikipedia.org/wiki/Media_type) 
(previously known as the MIME Type). 

## Extensions

There are emerging best practices, which in time will evolve in to specification extensions for
particular domains or uses.

The [extensions page](../extensions/README.md) gives an overview about relevant extensions for STAC Items.