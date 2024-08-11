# CRC GeoJSON File Structure and Rules

## General

The Consolidated Radar Client (CRC), built for VATSIM Network-VATUSA Division Virtual Air Traffic Controllers, utilizes GeoJSONs ([RFC-7946 format](https://datatracker.ietf.org/doc/html/rfc7946)) to display various types of spatial data elements onto their virtual radar display.

In this document, we will cover LineString (including MultiLineString), Symbol (Point), and Text (Point) features, how CRC makes use of this data, and common issues or non-standard situations that may present themselves.

- CRC is capable of rendering polygon features for "Tower-CAB" windows but that will not be covered here.

## GeoJson Feature Types & Associated Keys/Values

For CRC to know how to display the data appropriately within a ERAM window, certain keys and values must be defined. Defining these values are done using multiple methods (isDefaults, Overriding Properties, or CRC Auto-Assigned) that will be discussed later.

- **Keys and Values Applicable to All Features**:
  - `bcg`
    - Brightness control group, integer range [1, 40].
  - `filters`
    - Array of integers, representing an ERAM filter group, range [0, 40].
    - Value is required to be assigned in the geojson (not auto-assigned by CRC) if this data is meant for a CRC-ERAM display.

- **Line Features**: Represented by `"type":"LineString"` or `"type":"MultiLineString"`. Properties may include the following keys and values:
  - `style`
    - Possible values include "solid", "shortDashed", "longDashed", "longDashShortDash".
  - `thickness`
    - Integer values range from [1, 3].
  - Example:

    ```json
    {"type":"FeatureCollection","features":[{"type":"Feature","geometry":{"type":"LineString","coordinates":[[-117.02,32.6],[-116.99,32.57]]},"properties":{"bcg":3,"filters":[3],"style":"Solid","thickness":1}}]}
    ```

- **Symbol Features**: Represented by `"type":"Point"`. Properties may include the following keys and values:
  - `style`
    - One of the following: "obstruction1", "obstruction2", "heliport", "nuclear", "emergencyAirport", "radar", "iaf", "rnavOnlyWaypoint", "rnav", "airwayIntersections", "ndb", "vor", "otherWaypoints", "airport", "satelliteAirport", or  "tacan".
  - `size`
    - Integer values range from [1, 4].
  - Example:

    ```json
    {"type":"FeatureCollection","features":[{"type":"Feature","geometry":{"type":"Point","coordinates":[-176.64,51.88]},"properties":{"bcg":3,"filters":[3],"style":"airwayIntersections","size":1}}]}
    ```

- **Text Features**: Represented by `"type":"Point"`. Properties may include the following keys and values:
  - `text`
    - An array of strings, each string represents a line of text.
    - Multi-Line text is suported.
      - Example: `{"text":["TED","ANCHORAGE VOR/DME"]}`
    - Value is required to be assigned in the geojson (not auto-assigned by CRC) if this data is meant for a CRC-ERAM display.
  - `size`
    - Integer values range from [0, 5].
  - `underline`
    - Boolean, true or false.
  - `xOffset`
    - Positive integer.
  - `yOffset`
    - Positive integer.
  - `opaque`
    - Boolean, true or false.
  - Example:

    ```json
    {"type":"FeatureCollection","features":[{"type":"Feature","geometry":{"type":"Point","coordinates":[-176.64,51.88]},"properties":{"bcg":3,"filters":[3],"text":["PADK","ADAK"]"size":1,"underline":false,"opaque":false,"xOffset":12,"yOffset":0}}]}
    ```

## Key/Value Assignments - General

When the user wishes to display the data contained within a geoJson file on their CRC ERAM window, it first requires defining certain Keys/Values to be assigned to this data. This is done via one, or a combination of the following:

- Default Features
  - A feature within the geojson that defines the properites that will be applied to the other non-default features within the file that does not have overriding properties.
  - Default Features may be refered to as "isDefaults".
  - Details discussed later in the "Key/Value Assignments - isDefaults (Defaults) Features" section.

- Default Overrides
  - In a non-isDefaults feature, if the properties contains the appropriate keys/values for the feature-type, those values will be assigned to that feature and that feature alone.
    - These values will override ones contained in a Default Feature, if there is one.
  - Details discussed later in the "Key/Value Assignments - Default Overrides" section.

- CRC-Assigned Defaults
  - If the geojson does not have a valid isDefault or Default Overrides, CRC will assign certain properties to each feature within the geojson.
  - Details discussed later in the "Key/Value Assignments - CRC Auto-Assigned Defaults" section.

## Key/Value Assignments - isDefaults (Defaults) Features

isDefaults are defined in specific `"type":"Point"` features with property key/value of `"isLineDefaults":true`, `"isSymbolDefaults":true`, and `"isTextDefaults":true`. These are keys/values that CRC reads and assigns to the other features but are not rendered to the display.

Though, not required, it is best-practice to place the isDefaults at the beginning of the FeatureCollection prior to any rendered features for clarity and organizational purposes. Multiple isDefaults for the same type (line, text, symbol) is discouraged and we will speak about the handling of this situation later in the "Multiple isDefaults Features for Same Type" section.

Though not required, utilizing `90.0,180.0` for the Coordinates will result in a standard accross facilities and for US localized data, it will ensure the point is not rendered in geojson viewers in the same area as your video map data.

Examples:

- isLineDefaults Feature

  ```json
  {"type":"Feature","geometry":{"type":"Point","coordinates":[90,180]},"properties":{"isLineDefaults":true,"bcg":3,"filters":[3],"style":"Solid","thickness":1}}
  ```

- FeatureCollection with isLineDefaults & Line Features

  ```json
  {"type":"FeatureCollection","features":[{"type":"Feature","geometry":{"type":"Point","coordinates":[90,180]},"properties":{"isLineDefaults":true,"bcg":3,"filters":[3],"style":"Solid","thickness":1}},{"type":"Feature","geometry":{"type":"LineString","coordinates":[[-117.02,32.69],[-116.99,32.57]]},"properties":{}},{"type":"Feature","geometry":{"type":"LineString","coordinates":[[-117.04,32.62],[-117.15,32.72]]},"properties":{}},{"type":"Feature","geometry":{"type":"LineString","coordinates":[[-117.25,32.86],[-117.38,33.16]]},"properties":{}}]}
  ```

- isSymbolDefaults Feature

  ```json
  {"type":"Feature","geometry":{"type":"Point","coordinates":[90,180]},"properties":{"isSymbolDefaults":true,"bcg":3,"filters":[3],"style":"airwayIntersections","size":1}}
  ```

- FeatureCollection with isSymbolDefaults & Symbol Features

  ```json
  {"type":"FeatureCollection","features":[{"type":"Feature","geometry":{"type":"Point","coordinates":[90,180]},"properties":{"isSymbolDefaults":true,"bcg":3,"filters":[3],"style":"airwayIntersections","size":1}},{"type":"Feature","geometry":{"type":"Point","coordinates":[-116.95,32.54]},"properties":{"style":"vor"}},{"type":"Feature","geometry":{"type":"Point","coordinates":[-117.02,32.6]},"properties":{"style":"airwayIntersections"}},{"type":"Feature","geometry":{"type":"Point","coordinates":[-117.22,32.78]},"properties":{"style":"vor"}}]}
  ```

- isTextDefaults Feature

  ```json
  {"type":"Feature","geometry":{"type":"Point","coordinates":[90,180]},"properties":{"isTextDefaults":true,"bcg":3,"filters":[3],"size":1,"underline":false,"opaque":false,"xOffset":12,"yOffset":0}}
  ```

- FeatureCollection with isTextDefaults & Text Features

  ```json
  {"type":"FeatureCollection","features":[{"type":"Feature","geometry":{"type":"Point","coordinates":[90,180]},"properties":{"isTextDefaults":true,"bcg":3,"filters":[3],"size":1,"underline":false,"opaque":false,"xOffset":12,"yOffset":0}},{"type":"Feature","geometry":{"type":"Point","coordinates":[-116.95,32.54]},"properties":{"text":["TIJ"]}},{"type":"Feature","geometry":{"type":"Point","coordinates":[-117.02,32.6]},"properties":{"text":["TEYON"]}},{"type":"Feature","geometry":{"type":"Point","coordinates":[-117.22,32.78]},"properties":{"text":["MZB"]}}]}
  ```

- FeatureCollection with isLineDefaults, isSymbolDefaults, isTextDefaults + Line, Symbols, & Text Features

  ```json
  {"type":"FeatureCollection","features":[{"type":"Feature","geometry":{"type":"Point","coordinates":[90,180]},"properties":{"isLineDefaults":true,"bcg":3,"filters":[3],"style":"Solid","thickness":1}},{"type":"Feature","geometry":{"type":"Point","coordinates":[90,180]},"properties":{"isSymbolDefaults":true,"bcg":3,"filters":[3],"style":"airwayIntersections","size":1}},{"type":"Feature","geometry":{"type":"Point","coordinates":[90,180]},"properties":{"isTextDefaults":true,"bcg":3,"filters":[3],"size":1,"underline":false,"opaque":false,"xOffset":12,"yOffset":0}},{"type":"Feature","geometry":{"type":"LineString","coordinates":[[-117.02,32.6],[-116.99,32.57]]},"properties":{}},{"type":"Feature","geometry":{"type":"LineString","coordinates":[[-117.04,32.62],[-117.15,32.72]]},"properties":{}},{"type":"Feature","geometry":{"type":"LineString","coordinates":[[-117.25,32.86],[-117.38,33.16]]},"properties":{}},{"type":"Feature","geometry":{"type":"Point","coordinates":[-116.95,32.54]},"properties":{"style":"vor"}},{"type":"Feature","geometry":{"type":"Point","coordinates":[-117.02,32.6]},"properties":{"style":"airwayIntersections"}},{"type":"Feature","geometry":{"type":"Point","coordinates":[-117.22,32.78]},"properties":{"style":"vor"}},{"type":"Feature","geometry":{"type":"Point","coordinates":[-116.95,32.54]},"properties":{"text":["TIJ"]}},{"type":"Feature","geometry":{"type":"Point","coordinates":[-117.02,32.6]},"properties":{"text":["TEYON"]}},{"type":"Feature","geometry":{"type":"Point","coordinates":[-117.22,32.78]},"properties":{"text":["MZB"]}}]}
  ```

## Key/Value Assignments - Default Overrides

Though, there may be isDefaults contained within the .geojson that apply values to all respective features, each feature may contain one or more overriding key/value that will apply to that feature alone.

The following is an example of a geojson with Line features and an isLineDefaults feature but the 2nd rendered (non-isDefault) feature has Default Override values of `filters = 4`, `style = Dashed`, and `thickness = 3`, overriding the isDefault values of `filters = 3`, `style = Solid`, and `thickness = 1`.

```json
{"type":"FeatureCollection","features":[{"type":"Feature","geometry":{"type":"Point","coordinates":[90,180]},"properties":{"isLineDefaults":true,"bcg":3,"filters":[3],"style":"Solid","thickness":1}},{"type":"Feature","geometry":{"type":"LineString","coordinates":[[-117.02,32.6],[-116.99,32.57]]},"properties":{}},{"type":"Feature","geometry":{"type":"LineString","coordinates":[[-117.04,32.62],[-117.15,32.72]]},"properties":{"filters":[4],"style":"Dashed","thickness":3}},{"type":"Feature","geometry":{"type":"LineString","coordinates":[[-117.25,32.86],[-117.38,33.16]]},"properties":{}}]}
```

Table for clarificaiton:
| **Feature Number** | **Properties**                                        | **Effective Values**                                                   | **Notes**                                 |
|--------------------|-------------------------------------------------------|------------------------------------------------------------------------|-------------------------------------------|
| 1 (isLineDefaults)     | `bcg: 3`<br>`filters: 3`<br>`style: Solid`<br>`thickness: 1`    | n/a                               | The `isLineDefaults` contains values for<br>all keys in the properties, so there<br>is nothing for CRC to auto-assign.   |
| 2                  | `{}`                              | `bcg: 3`<br>`filters: 3`<br>`style: Solid`<br>`thickness: 1`                               | The properties section is blank (no overriding<br> defaults) and therefore, inherits values<br>from the isLineDefaults.        |
| 3                  | `filters: 4`<br>`style: Dashed`<br>`thickness: 3`           | `bcg: 3`<br>`filters: 4`<br>`style: Dashed`<br>`thickness: 3`    | The properties override the defaults with<br>new values for `filters`, `style`, and<br>`thickness`, but `bcg` retains the<br>value defined in the `isLineDefaults` feature.   |
| 4                  | `{}`                              | `bcg: 3`<br>`filters: 3`<br>`style: Solid`<br>`thickness: 1`                               | The properties section is blank (no overriding<br> defaults) and therefore, inherits values<br>from the isLineDefaults.        |

*Note: Scripts like [this one](https://github.com/KSanders7070/Clean_CRC_GEOJSON_PROPERTIES) can assist in "clearning up" the properties section of each feature by removing all Overriding Defaults, while retaining the isDefaults feature property values.*

## Key/Value Assignments - CRC Auto-Assigned Defaults

When the required keys/values are not defined via either isDefaults or Default Overrides, CRC assigns the following automatically:  
*Note: Filters and Text keys/values are not eligable to be auto-assigned by CRC therefore, they must be defined via another method or that feature will not display on an ERAM window.*

- LINE:
  - BCG = 1
  - Style = solid
  - Thickness = 1
- SYMBOL:
  - BCG = 1
  - Style = vor
  - Size = 1
- TEXT:
  - BCG = 1
  - Size = 1
  - Underline = false
  - xOffset = 0
  - yOffset = 0
  - Opaque = false

In this following example geojson, there are Lines, Symbols, & Text features along with the isDefaults for Lines and Symbols but the isTextDefaults feature is missing. Though CRC auto-assigns the text features for most of the properties it needs, the lack of the assigned Filter default property will result in the text features not displaying in the CRC-ERAM window but, the Lines and Symbols features will. To correct this issue, the geojson manager must either create a isTextDefaults feature with a minium of a Filter property defined or place a Default Override property in each of the Text features defining the Filter value.

```json
{"type":"FeatureCollection","features":[{"type":"Feature","geometry":{"type":"Point","coordinates":[90,180]},"properties":{"isLineDefaults":true,"bcg":3,"filters":[3],"style":"Solid","thickness":1}},{"type":"Feature","geometry":{"type":"Point","coordinates":[90,180]},"properties":{"isSymbolDefaults":true,"bcg":3,"filters":[3],"style":"airwayIntersections","size":1}},{"type":"Feature","geometry":{"type":"LineString","coordinates":[[-117.02,32.6],[-116.99,32.57]]},"properties":{}},{"type":"Feature","geometry":{"type":"LineString","coordinates":[[-117.04,32.62],[-117.15,32.72]]},"properties":{}},{"type":"Feature","geometry":{"type":"LineString","coordinates":[[-117.25,32.86],[-117.38,33.16]]},"properties":{}},{"type":"Feature","geometry":{"type":"Point","coordinates":[-116.95,32.54]},"properties":{"style":"vor"}},{"type":"Feature","geometry":{"type":"Point","coordinates":[-117.02,32.6]},"properties":{"style":"airwayIntersections"}},{"type":"Feature","geometry":{"type":"Point","coordinates":[-117.22,32.78]},"properties":{"style":"vor"}},{"type":"Feature","geometry":{"type":"Point","coordinates":[-116.95,32.54]},"properties":{"text":["TIJ"]}},{"type":"Feature","geometry":{"type":"Point","coordinates":[-117.02,32.6]},"properties":{"text":["TEYON"]}},{"type":"Feature","geometry":{"type":"Point","coordinates":[-117.22,32.78]},"properties":{"text":["MZB"]}}]}
```

## Multiple Feature Types in a GeoJSON

If the manager of the geojsons finds that having more than one type of feature (lines, symbols, and text) in a geojson file is complicating management of the file and hindering flexibility, scripts like [this one](https://github.com/KSanders7070/Split_CRC_GeoJSON_Feature_Types) can split a single GeoJSON into multiple files based on feature type, each retaining relevant defaults at the top.

## Multiple isDefaults Features for Same Type

If multiple isDefaults features for the same type is found in the geojson file, the latest defined key/value is replaced as the assigned "defaults".

Example:

| Line Defaults       |  `bcg`  | `filters`  |  `style`  | `thickness` |
|:------------------:|:-------:|:----------:|:---------:|:-----------:|
| 1st isLineDefault  |    3    |    3     |   Solid   |      1      |
| 2nd isLineDefault  |    2    |    1     |  Dashed   |      2      |
| 3rd isLineDefault  |    1    |   2,5    |   Solid   |      (not defined)      |
| Ending result    |    1    |   2,5    |   Solid   |      2      |
