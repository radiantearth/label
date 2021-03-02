# Template Extension Specification

* **Title:** Label
* **Identifier:** <https://stac-extensions.github.io/label/v1.0.0/schema.json>
* **Field Name Prefix:** label
* **Scope:** Item, Collection
* **Extension [Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/extensions/README.md#extension-maturity):** Proposal
* **Owner**: @jisantuc

This is the place to add a short introduction.

* Examples:
  + Roads
    - [Example Roads Item](examples/spacenet-roads/roads_item.json)
    - [Example Roads Asset (labels)](examples/spacenet-roads/spacenetroads_AOI_3_Paris_img101.geojson)
    - [Example Roads Source Imagery Item](examples/spacenet-roads/roads_source.json)
    - [Example Roads Collection](examples/spacenet-roads/roads_collection.json)

  + Buildings
    - [Example Collection of Two Building Footprint Label Catalogs](examples/multidataset/catalog.json)
    - [Example SpaceNet Buildings Collection](examples/multidataset/spacenet-buildings/collection.json)
    - [Example SpaceNet Buildings (Vegas) Item](examples/multidataset/spacenet-buildings/AOI_2_Vegas_img2636.json)
    - [Example SpaceNet Buildings (Paris) Item](examples/multidataset/spacenet-buildings/AOI_3_Paris_img1648.json)
    - [Example SpaceNet Buildings (Shanghai) Item](examples/multidataset/spacenet-buildings/AOI_4_Shanghai_img3344.json)
    - [Example World Bank Zanzibar Buildings Collection](examples/multidataset/zanzibar/collection.json)
    - [Example World Bank Zanzibar Building Item 1](examples/multidataset/zanzibar/znz001.json)
    - [Example World Bank Zanzibar Building Item 2](examples/multidataset/zanzibar/znz029.json)
* Schema
  + [JSON Schema](json-schema/schema.json)

## Item Properties

| Field Name        | Type                             | Description |
| ----------------- | -------------------------------- | ----------- |
| label:properties  | \[string]\|null                  |**REQUIRED** These are the names of the property field(s) in each `Feature` of the label asset's `FeatureCollection` that contains the  classes (keywords from `label:classes` if the property defines classes). If labels are rasters, use `null` . |
| label:classes     | \[[Class Object](#class-object)] |**REQUIRED** if using categorical data. A Class Object defining the list of possible class names for each `label:properties` . (e.g., tree, building, car, hippo) |
| label:description | string                           | **REQUIRED** A description of the label, how it was created, and what it is recommended for |
| label:type        | string                           |**REQUIRED** An ENUM of either `vector` label type or `raster` label type |
| label:tasks       | \[string]                        |Recommended to be a subset of 'regression', 'classification', 'detection', or 'segmentation', but may be an arbitrary value |
| label:methods     | \[string]                        |Recommended to be a subset of 'automated' or 'manual', but may be an arbitrary value. |
| label:overviews   | \[[Label Overview Object](#label-overview-object)] |An Object storing counts (for classification-type data) or summary statistics (for continuous numerical/regression data). |

### Class Object

| Field Name | Type                 |Description |
| ---------- | -------------------- |----------- |
| name       | string\|null         |**REQUIRED** The property key within the asset's each `Feature` corresponding to class labels. If labels are raster-formatted, use null. |
| classes    | \[string]\|\[number] |**REQUIRED** The different possible class values within the property `name` . |

### Label Overview Object

| Field Name   | Type                             |  Description |
| ------------ | -------------------------------- |  ----------- |
| property_key | string                           |  The property key within the asset corresponding to class labels. |
| counts       | \[[Count Object](#count-object)] |  An object containing counts for categorical data. |
| statistics   | \[[Stats Object](#stats-object)] |  An object containing statistics for regression/continuous numeric value data. |

### Count Object

| Field Name | Type    | Name       | Description |
| ---------- | ------- | ---------- | ----------- |
| name       | string  | Class Name | The different possible classes within the property `name` . |
| count      | integer | Count      | The number of occurrences of the class. |

``` json
  {
    "property_key": "road_type",
    "counts": [
      {
        "name": "dirt",
        "count": 10
      },
      {
        "name": "paved",
        "count": 99
      }
    ]
  }
```

### Stats Object

| Field Name | Type   | Name      | Description |
| ---------- | ------ | --------- | ----------- |
| name       | string | Stat Name | The name of the statistic being reported. |
| value      | number | Value     | The value of the statistic `name` . |

``` json
  {
    "property_key": "elevation",
    "statistics": [
      {
        "name": "mean",
        "value": 100.1
      },
      {
        "name": "median",
        "value": 102.3
      },
      {
        "name": "max",
        "value": 100000
      }
    ]
  }
```

### Assets

#### labels (required)

The Label Extension requires at least one asset that uses the key "labels". The asset will contain a link to the actual label data. The asset has these requirements:

* is a GeoJSON FeatureCollection
* if `label:tasks` is tile_classification, object_detection, or segmentation, each feature should have one or more properties containing the label(s) for the class (one of `label:classes`). the name of the property can be anything (use "label" if making from scratch), but needs to be specified in the `Item` with the `label:properties` field.
* if `label:tasks` is tile_regression, each feature should have one or more properties defining the value for regression. the name of the property can be anything (use "label" if making from scratch), but needs to be specified in the `Item` with the `label:properties` field.

#### Raster Label Notes

If the labels are formatted as rasters - for example, a pixel mask with 1s where there is water and 0s where there is land - the following approach is recommended for including those data.

The raster label file (e.g. a GeoTIFF) should be included as an asset under the item. Along with the image file, a GeoJSON `FeatureCollection` asset should be included. That `FeatureCollection` should contain a single `Feature` , ideally a polygon geometry defining the extent of the raster.

#### Rendered images (optional)

The source imagery used for creating the label is linked to under `links` (see below). However the source imagery is likely to have been rendered in some way when creating the training data. For instance, a byte-scaled true color image may have been created from the source imagery. It may be useful to save this image and include it as an asset in the `Item` .

### Links: source imagery

A Label Item links to any source imagery that the AOI applys to by linking to the STAC Item representing the imagery. Source imagery is indicated by using a `rel` type of "source" and providing the link to the STAC Item.

In addition the source imagery link has a new label extension specific field:

| Field Name   | Type      | Name   | Description |
| ------------ | --------- | ------ | ----------- |
| label:assets | \[string] | Assets | The keys for the assets within the `source` item to which this label item applies. |

The `label:assets` field applies to situations where the labels may apply to certain assets inside the source imagery Item, but not others (e.g. if the labels were traced on top of RGB imagery, but the source item also contains assets for a Digital Elevation Model).
