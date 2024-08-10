# CRC GeoJSON File Structure and Rules

## General

The Consolidated Radar Client (CRC), built for VATSIM Network-VATUSA Division Virtual Air Traffic Controllers, utilizes GeoJSONs ([RFC-7946 format](https://datatracker.ietf.org/doc/html/rfc7946)) to display various types of spatial data elements onto their virtual radar display.

In this document, we will cover LineString (including MultiLineString), Symbol (Point), and Text (Point) features, how CRC makes use of this data, and common issues or non-standard situations that may present themselves.

- CRC is capable of rendering polygon features for "Tower-CAB" windows but that will not be covered here.

## GeoJson Feature Types & Associated Keys

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
  - Default Features may be refered to as "isDefaults"
  - Details discussed later in this document.

- Default Overrides
  - In a non-isDefaults feature, if the properties contains the appropriate keys/values for the feature-type, those values will be assigned to that feature and that feature alone.
    - These values will override ones contained in a Default Feature, if there is one.
  - Details discussed later in this document.

- CRC-Assigned Defaults
  - If the geojson does not have a valid isDefault or Default Overrides, CRC will assign certain properties to each feature within the geojson.
  - Details discussed later in this document.

## Key/Value Assignments - isDefaults (Defaults) Features

isDefaults are defined in specific `"type":"Point"` features with property key/value of `"isLineDefaults":true`, `"isSymbolDefaults":true`, and `"isTextDefaults":true`.

These are keys/values assigned to the other features but are not rendered to the display.

Though, not required, it is best-practice to place the isDefaults at the beginning of the GeoJSON file prior to any rendered features for clarity and organizational purposes.

Multiple isDefaults for the same type (line, text, symbol) is discouraged and we will speak about the handling of this situation later in this document.

Though not required, utilizing `90.0,180.0` for the Coordinates will result in a standard accross facilities and increases the chance that geojson readers will display that point away from most other focus data.

Examples:

- Line Features + isDefaults

  ```json
  {"type":"FeatureCollection","features":[{"type":"Feature","geometry":{"type":"Point","coordinates":[90,180]},"properties":{"isLineDefaults":true,"bcg":3,"filters":[3],"style":"Solid","thickness":1}},{"type":"Feature","geometry":{"type":"LineString","coordinates":[[-117.02,32.69],[-116.99,32.57]]},"properties":{}},{"type":"Feature","geometry":{"type":"LineString","coordinates":[[-117.04,32.62],[-117.15,32.72]]},"properties":{}},{"type":"Feature","geometry":{"type":"LineString","coordinates":[[-117.25,32.86],[-117.38,33.16]]},"properties":{}}]}
  ```

- Symbol Features + isDefaults

  ```json
  {"type":"FeatureCollection","features":[{"type":"Feature","geometry":{"type":"Point","coordinates":[90,180]},"properties":{"isSymbolDefaults":true,"bcg":3,"filters":[3],"style":"airwayIntersections","size":1}},{"type":"Feature","geometry":{"type":"Point","coordinates":[-116.95,32.54]},"properties":{"style":"vor"}},{"type":"Feature","geometry":{"type":"Point","coordinates":[-117.02,32.6]},"properties":{"style":"airwayIntersections"}},{"type":"Feature","geometry":{"type":"Point","coordinates":[-117.22,32.78]},"properties":{"style":"vor"}}]}
  ```

- Text Features + isDefaults

  ```json
  {"type":"FeatureCollection","features":[{"type":"Feature","geometry":{"type":"Point","coordinates":[90,180]},"properties":{"isTextDefaults":true,"bcg":3,"filters":[3],"size":1,"underline":false,"opaque":false,"xOffset":12,"yOffset":0}},{"type":"Feature","geometry":{"type":"Point","coordinates":[-116.95,32.54]},"properties":{"text":["TIJ"]}},{"type":"Feature","geometry":{"type":"Point","coordinates":[-117.02,32.6]},"properties":{"text":["TEYON"]}},{"type":"Feature","geometry":{"type":"Point","coordinates":[-117.22,32.78]},"properties":{"text":["MZB"]}}]}
  ```

## Key/Value Assignments - Default Overrides

Though, there may be isDefaults contained within the .geojson that apply values to all features, each feature may contain one or more overriding key/value that will apply to that feature alone.

If there is missing Default Overrides keys/values in a feature, the missing value is automatically assigned per the isDefaults.

The following is an example of a geojson with Line features and an isDefault feature but the 2nd rendered (non-isDefault) feature has Default Overrides values of `filters = 4`, `style = Dashed`, and `thickness = 3`, overriding the isDefault values of `filters = 3`, `style = Solid`, and `thickness = 1`.

- For clarificaiton: the 1st and 3rd rendered features will display in ERAM with the values `filters = 3`, `style = Solid`, and `thickness = 1`, but the 2nd renedered feature will display with `filters = 4`, `style = Dashed`, and `thickness = 3`.

```json
{"type":"FeatureCollection","features":[{"type":"Feature","geometry":{"type":"Point","coordinates":[90,180]},"properties":{"isLineDefaults":true,"bcg":3,"filters":[3],"style":"Solid","thickness":1}},{"type":"Feature","geometry":{"type":"LineString","coordinates":[[-117.02,32.6],[-116.99,32.57]]},"properties":{}},{"type":"Feature","geometry":{"type":"LineString","coordinates":[[-117.04,32.62],[-117.15,32.72]]},"properties":{"filters":[4],"style":"Dashed","thickness":3}},{"type":"Feature","geometry":{"type":"LineString","coordinates":[[-117.25,32.86],[-117.38,33.16]]},"properties":{}}]}
```

## Key/Value Assignments - CRC Auto-Assigned Defaults

When the required keys/values are not defined via either isDefaults or Default Overrides, CRC assigns the following by automatically:  
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

## Multiple Feature Types in single GeoJSON

Though supported, having more than one type of feature (lines, symbols, and text) in a single geojson file can complicate management of the file and hinder flexibility. Scripts like [this one](https://github.com/KSanders7070/Split_CRC_GeoJSON_Feature_Types) can split a single GeoJSON into multiple files based on feature type, each retaining relevant defaults at the top.

## Multiple isDefaults Features for Same Type

If multiple isDefaults features for the same type is found in the geojson file, the latest defined key/value is replaced as the assigned "defaults".

Example:

- 1st isLineDefault in the geojson:
  - `bcg` = 3 `filters` = [3] `style` = Solid `thickness` = 1

- 2nd isLineDefault in the geojson:
  - `bcg` = 2 `filters` = [1] `style` = Dashed `thickness` = 2

- 3rd isLineDefault in the geojson:
  - `bcg` = 1 `filters` = [2,5] `style` = Solid

- What is assigned to all line features:
  - `bcg` = 1 `filters` = [2,5] `style` = Solid `thickness` = 2
