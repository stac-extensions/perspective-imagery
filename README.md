# Perspective Imagery Extension Specification

- **Title:** Perspective Imagery
- **Identifier:** <https://stac-extensions.github.io/perspective-imagery/v1.0.0/schema.json>
- **Field Name Prefix:** pers
- **Scope:** Item, Collection
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/extensions/README.md#extension-maturity):** Proposal
- **Owner**: @asgerpetersen

This document explains the Perspective Imagery extension to the [SpatioTemporal Asset Catalog](https://github.com/radiantearth/stac-spec) (STAC) 
specification.

The extension aims to support a broad spectrum of perspective imagery use cases: 
- photos taken with commodity cameras (SLR, mobile phone etc) or photogrammetric cameras
- photos without georeference other than an aproximate footprint, georeferenced manually with a single point and possibly a direction, by EXIF GPS, 
GPS/INS or even aerotriangulation.

The extension supports referencing camera intrinsics instead of embedding it in every Item.

- Examples:
  - [Item example](examples/item.json): Shows the basic usage of the extension in a STAC Item
  - [Collection example](examples/collection.json): Shows the basic usage of the extension in a STAC Collection
- [JSON Schema](json-schema/schema.json)
- [Changelog](./CHANGELOG.md)

## Collection Fields

| Field Name                | Type                                        | Description |
| ------------------------- | ------------------------------------------- | ----------- |
| pers:interior_orientation | [InteriorOrientation](#interiororientation) | Camera interior orientation |

## Item Properties

| Field Name                | Type                                        | Description |
| ------------------------- | ------------------------------------------- | ----------- |
| pers:interior_orientation | [InteriorOrientation](#interiororientation) | Camera interior orientation |
| pers:perspective_center   | \[number]                                   | XY and possibly Z location of the sensor at the time of exposure as either `[X, Y]` or `[X, Y, Z]` |
| pers:crs                  | string\|number\|object                      | The spatial reference system for coordinate specified in `perspective_center`. Specified as [numerical EPSG code](http://www.epsg-registry.org/), [WKT2 (ISO 19162) string](http://docs.opengeospatial.org/is/18-010r7/18-010r7.html) or [PROJJSON object](https://proj.org/specifications/projjson.html). Defaults to EPSG code 4326. |
| pers:vertical_crs         | string\|number\|object                      | The vertical spatial reference system for Z coordinate specified in `perspective_center` if this is not defined by `pers:crs`. specified as [numerical EPSG code](http://www.epsg-registry.org/), [WKT2 (ISO 19162) string](http://docs.opengeospatial.org/is/18-010r7/18-010r7.html) or [PROJJSON object](https://proj.org/specifications/projjson.html) |
| pers:rotation_matrix      | \[number]                                   | Rotation matrix as `[m11, m12, m13, m21, m22, m23, m31, m32, m33]` in row major |

**Note** Should we add an option to add omega, phi and kappa? This may be what people actually have in their data. Problem is of course that USING 
this formulation is often a pain in the behind.

### Additional Field Information

### pers:interior_orientation
The interior orientation of camera describes the physical properties of the camera allowing reconstruction of scene geometry. As these data 
typically are the same for all images recorded with the same camera it may make sense to put it on the Collection or link to a file with the 
[InteriorOrientation](#interiororientation) object using the relation type defined in [Relation types](#relation-types).

#### pers:perspective_center
Location of the sensor in the geocentric coordinate system at time of exposure. This may be specified either as `[X, Y]` or `[X, Y, Z`].

#### pers:crs
The spatial reference system for coordinate specified in `perspective_center`. Specified as [numerical EPSG code](http://www.epsg-registry.org/), 
[WKT2 (ISO 19162) string](http://docs.opengeospatial.org/is/18-010r7/18-010r7.html) or 
[PROJJSON object](https://proj.org/specifications/projjson.html). Defaults to EPSG code 4326. 

#### pers:vertical_crs
If `pers:perspective_center` is 3-dimensional and `pers:crs` does not describe the vertical component this property describes the vertical reference 
system for the Z coordinate specified in `perspective_center` if this is not defined by `pers:crs`. Specified as 
[numerical EPSG code](http://www.epsg-registry.org/), [WKT2 (ISO 19162) string](http://docs.opengeospatial.org/is/18-010r7/18-010r7.html) or 
[PROJJSON object](https://proj.org/specifications/projjson.html). Defaults to EPSG code 4326. 

#### pers:rotation_matrix
9 elements of the orthogonal matrix `[m11, m12, m13, m21, m22, m23, m31, m32, m33]` in row major order which rotates the spatial coordinate system 
(defined by `pers:crs` and `pers:vertical_crs`) to be parallel to the image record coordinate system. See section 4.1 in \[[1](#references)]. This 
requires a three dimensional `perspective_center`.

### InteriorOrientation

The purpose of the `InteriorOrientation` object is to describe as much of the camera interior orientation as is known.

| Field Name              | Type      | Description |
| ----------------------- | --------- | ----------- |
| camera_id               | string    | Unique ID of the camera used |
| camera_manufacturer     | string    | Camera manufacturer |
| camera_model            | string    | Camera model |
| sensor_array_dimensions | \[int]    | Sensor dimensions as \[number_of_columns, number_of_rows] |
| pixel_spacing           | \[number] | Distance between pixel centers in mm as \[column_spacing, row_spacing] |
| focal_length            | number    | Focal length in mm |
| principal_point_offset  | \[number] | Principal point offset in mm as \[offset_x, offset_y] |
| radial_distortion       | \[number] | Radial lens distortion koefficients as \[k0, k1, k2, k3] |
| affine_distortion       | \[number] | Affine distortion coefficients as \[a1, b1, c1, a2, b2, c2] |
| calibration_date        | string    | Date of last calibration |

#### camera_id

Unique ID of the camera used.

#### camera_manufacturer

Manufacturer ("make") of the camera. Examples: "Nikon", "Apple", "Vexcel".

#### camera_model
Camera model. Examples: "D90", "iPhone SE", "UC OM3p".

**Note:** For consumer cameras the manufacturer and model may enable estimated interior orientations by lookup. For instance using lists like 
[this](https://github.com/mapillary/OpenSfM/blob/main/opensfm/data/sensor_data_detailed.json) from mapillary.

#### sensor_array_dimensions
The number of columns and rows in the image sensor expressed as an array of two ints like `[number_of_columns, number_of_rows]`. In most cases this 
is equal to the number of pixels in the original image output from the camera.

**Note:**
This is not necessarily equal to the dimensions of the image asset at hand. If the current image has been resized then the dimensions of the 
original image must also be known.

Giving these numbers as an array makes it way more likely that the implementor switches the dimensions by accident. It should be considered either 
using an object instead maybe like `{"ncols": int, "nrows": int}` or using seperate properties like `"number_of_rows": int` and 
`"number_of_columns": int`.

#### pixel_spacing
Physical distance in the image plane (usually on the sensor chip) between centers of adjacent pixels within a column and within a row. Measured in 
`mm`. Expressed as `[column_spacing, row_spacing]`.

If unkown this may be derived from the physical width and height of the sensor chip and the sensor_array_dimensions.

#### focal_length
Effective distance from optical lens to sensor element. Measured in `mm`.

#### principal_point_offset
Principal point offset measured in `mm` and expressed as `[offset_x, offset_y]`. The principal point offset is 
the difference between the geometric center of the sensor and the optical center (see figure 12 in \[[1](#references)]).

#### radial_distortion
Radial lens distortion coefficients as defined in section 3.3.3 of \[[1](#references)] expressed as `[k0, k1, k2, k3]` where k0 is in `mm`, k1 is in 
`mm^-2`, k2 is in `mm^-4` and k3 is in `mm^-6`.

#### affine_distortion
Affine distortion coefficients as defined in section Table 1, ID 26 (and section 3.3.1) of \[[1](#references)]. Expressed as `[a1, b1, c1, a2, b2, c2]`.

#### calibration_date
Date of sensor calibration data used. Formatted according to [https://datatracker.ietf.org/doc/html/rfc3339#section-5.6](https://datatracker.ietf.org/doc/html/rfc3339#section-5.6).

## Relation types

The following types should be used as applicable `rel` types in the
[Link Object](https://github.com/radiantearth/stac-spec/tree/master/item-spec/item-spec.md#link-object).

| Type                | Description |
| ------------------- | ----------- |
| interior-orientation      | This link points to json document with an [InteriorOrientation](#interiororientation) object. |

## Best practices
- Use rotation matrix in preference of omega, phi and kappa
- Use the [view](https://github.com/stac-extensions/view) extension. Particularly the `view:azimuth` and `view:off_nadir` are useful way to describe 
important characteristics of the image. Even when the full exterior orientation is also speciffied.
- Use `datetime` defined in STAC [item metadata](https://github.com/radiantearth/stac-spec/blob/master/item-spec/item-spec.md#properties-object) for 
time of acquisition.
- Use instrument metadata from STAC item 
[common metadata](https://github.com/radiantearth/stac-spec/blob/master/item-spec/common-metadata.md#instrument) where it makes sense.
 
## References
\[1]: NGA STANDARDIZATION DOCUMENT, Frame Sensor Model Metadata Profile Supporting Precise Geopositioning, 2011-07-07 
[Link](http://www.gmljp2.work/fsmmg_standard/Frame_Formulation_Paper_Version_2.1_July2011.pdf)

## Contributing

All contributions are subject to the
[STAC Specification Code of Conduct](https://github.com/radiantearth/stac-spec/blob/master/CODE_OF_CONDUCT.md).
For contributions, please follow the
[STAC specification contributing guide](https://github.com/radiantearth/stac-spec/blob/master/CONTRIBUTING.md) Instructions
for running tests are copied here for convenience.

### Running tests

The same checks that run as checks on PR's are part of the repository and can be run locally to verify that changes are valid. 
To run tests locally, you'll need `npm`, which is a standard part of any [node.js installation](https://nodejs.org/en/download/).

First you'll need to install everything with npm once. Just navigate to the root of this repository and on 
your command line run:
```bash
npm install
```

Then to check markdown formatting and test the examples against the JSON schema, you can run:
```bash
npm test
```

This will spit out the same texts that you see online, and you can then go and fix your markdown or examples.

If the tests reveal formatting problems with the examples, you can fix them with:
```bash
npm run format-examples
```
