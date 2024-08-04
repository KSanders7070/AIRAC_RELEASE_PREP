# CRC GeoJSON File Structure and Rules

## File Composition

- Consolidated Radar Client (CRC) utilizes GeoJSONs ([RFC-7946 format](https://datatracker.ietf.org/doc/html/rfc7946)) to represent various types of spatial data elements.
- In this document, we will cover LineString (including MultiLineString), Symbol (Point), and Text (Point) features. CRC is capable of rendering polygon features but only under certain circumstances and we will not cover that here.

## Feature Types and Associated Properties

- **Line**: Represented by `"type":"LineString"` or `"type":"MultiLineString"`. Properties may include:
  - `bcg`
  - `filters`
  - `style`
  - `thickness`
    - Example:

      ```json
      {"type":"FeatureCollection","features":[{"type":"Feature","geometry":{"type":"LineString","coordinates":[[-117.02,32.6],[-116.99,32.57]]},"properties":{"bcg":3,"filters":[3],"style":"Solid","thickness":1}}]}
      ```

- **Symbol**: Represented by `"type":"Point"`. Properties may include:
  - `bcg`
  - `filters`
  - `style`
  - `size`
    - Example:

      ```json
      {"type":"FeatureCollection","features":[{"type":"Feature","geometry":{"type":"Point","coordinates":[-176.64,51.88]},"properties":{"bcg":3,"filters":[3],"style":"airwayIntersections","size":1}}]}
      ```

- **Text**: Represented by `"type":"Point"`. Properties may include:
  - `bcg`
  - `filters`
  - `text`
  - `size`
  - `underline`
  - `xOffset`
  - `yOffset`
  - `opaque`
    - Example:

      ```json
      {"type":"FeatureCollection","features":[{"type":"Feature","geometry":{"type":"Point","coordinates":[-176.64,51.88]},"properties":{"bcg":3,"filters":[3],"text":["PADK","ADAK"]"size":1,"underline":false,"opaque":false,"xOffset":12,"yOffset":0}}]}
      ```

## CRC Defaults and Structure

- CRC Defaults are features within the geojson file that define the property values to be assigned to all other features in that file that do not have defined properties themselves. Properties in a non-CRC Defaults feature may be referred to as "Overriding Properties".
- CRC Defaults are defined in specific `"type":"Point"` features with property key/values of `"isLineDefaults":true`, `"isSymbolDefaults":true`, and `"isTextDefaults":true`. These are properties assigned to the other features but are not rendered by CRC.
- Though, not required, CRC Default features should be placed at the beginning of the GeoJSON file prior to any rendered features for clarity and organizational purposes.
- These CRC Default features are often just referred to as "isDefaults" or "CRC Defaults".
- All of the following examples will display in a CRC-ERAM window:
  - Line Features + CRC Defaults

    ```json
    {"type":"FeatureCollection","features":[{"type":"Feature","geometry":{"type":"Point","coordinates":[90,180]},"properties":{"isLineDefaults":true,"bcg":3,"filters":[3],"style":"Solid","thickness":1}},{"type":"Feature","geometry":{"type":"LineString","coordinates":[[-117.02,32.69],[-116.99,32.57]]},"properties":{}},{"type":"Feature","geometry":{"type":"LineString","coordinates":[[-117.04,32.62],[-117.15,32.72]]},"properties":{}},{"type":"Feature","geometry":{"type":"LineString","coordinates":[[-117.25,32.86],[-117.38,33.16]]},"properties":{}}]}
    ```

  - Symbol Features + CRC Defaults

    ```json
    {"type":"FeatureCollection","features":[{"type":"Feature","geometry":{"type":"Point","coordinates":[90,180]},"properties":{"isSymbolDefaults":true,"bcg":3,"filters":[3],"style":"airwayIntersections","size":1}},{"type":"Feature","geometry":{"type":"Point","coordinates":[-116.9527,32.540442]},"properties":{"style":"vor"}},{"type":"Feature","geometry":{"type":"Point","coordinates":[-117.021344,32.600622]},"properties":{"style":"airwayIntersections"}},{"type":"Feature","geometry":{"type":"Point","coordinates":[-117.225421,32.782208]},"properties":{"style":"vor"}}]}
    ```

  - Text Features + CRC Defaults

    ```json
    {"type":"FeatureCollection","features":[{"type":"Feature","geometry":{"type":"Point","coordinates":[90,180]},"properties":{"isTextDefaults":true,"bcg":3,"filters":[3],"size":1,"underline":false,"opaque":false,"xOffset":12,"yOffset":0}},{"type":"Feature","geometry":{"type":"Point","coordinates":[-116.9527,32.540442]},"properties":{"text":["TIJ"]}},{"type":"Feature","geometry":{"type":"Point","coordinates":[-117.021344,32.600622]},"properties":{"text":["TEYON"]}},{"type":"Feature","geometry":{"type":"Point","coordinates":[-117.225421,32.782208]},"properties":{"text":["MZB"]}}]}
    ```

- When the CRC Defaults nor the feature properties define a required properties key/value, CRC assigns the following by automatically:
  - LINE:
    - Filters = (not assigned)
    - BCG = 1
    - Style = solid
    - Thickness = 1
  - SYMBOL:
    - Filters = (not assigned)
    - BCG = 1
    - Style = vor
    - Size = 1
  - TEXT:
    - Filters = (not assigned)
    - BCG = 1
    - Text = (not assigned)
    - Size = 1
    - Underline = false
    - xOffset = 0
    - yOffset = 0
    - Opaque = false
- For geojson data to display on an ERAM window within CRC, there must be at least a defined Filters value (and text value for text features) in either the CRC Defaults feature (covered below) or the individual overriding feature properties (covered below).

## Handling Multiple Types

- Files often contain mixed feature types which can, at times, complicate management. Utility scripts like [this one](https://github.com/KSanders7070/Split_CRC_GeoJSON_Feature_Types) can split a single GeoJSON into multiple files based on feature type, each retaining relevant defaults at the top.

## Detailed Property Values for GeoJSON Features

- **General Properties for All Features**:
  - `color`: A hexadecimal color code (e.g., "#FFFFFF").
  - `bcg`: Brightness control group, integer range [1, 40].
  - `filters`: Array of integers, representing a filter group, range [0, 40].
- **Line Properties**:
  - `style`: Possible values include "solid", "shortDashed", "longDashed", "longDashShortDash".
  - `thickness`: Integer values range from [1, 3].
- **Symbol Properties**:
  - `style`: Possible values include "obstruction1", "obstruction2", "heliport", "nuclear", "emergencyAirport", "radar", "iaf", "rnavOnlyWaypoint", "rnav", "airwayIntersections", "ndb", "vor", "otherWaypoints", "airport", "satelliteAirport", "tacan".
  - `size`: Integer values range from [1, 4].
- **Text Properties**:
  - `text`: An array of strings, each string represents a line of text.
  - `size`: Integer values range from [0, 5].
  - `underline`: Boolean, true or false.
  - `xOffset`: Positive integer.
  - `yOffset`: Positive integer.
  - `opaque`: Boolean, true or false.

## Overriding CRC Defaults Per Feature

- Though, there may be CRC Defaults contained within the .geojson that apply values to all features, each feature may contain one or more overriding properties that will apply to that feature alone.
- If there is missing overriding properties in a feature, the missing value is automatically assigned per the CRC Default.

## Usage of Defaults

- Defaults are specified in specific 'Point' features marked by properties like `isSymbolDefaults`, `isLineDefaults`, and `isTextDefaults`. These defaults apply to subsequent features of the same type unless explicitly overridden by individual feature properties.  
- Coordinates for isDefaults should always equal: 90.0,180.0

## Example GeoJSONs

- Attached you will find some example geojson files. Here is the breakdown for each of them:
  - Example_Lines.geojson
    - Contains nothing but Line features with the exception of the isDefault feature.
    - All line features will display in a ERAM window within CRC.
  - Example_Symbols.geojson
    - Contains nothing but Symbol features with the exception of the isDefault feature.
    - All Symbol features will display in a ERAM window within CRC.
  - Example_Text.geojson
    - Contains nothing but Text features with the exception of the isDefault feature.
    - All TExt features will display in a ERAM window within CRC.
  - Example_Lines_Symbols_Text.geojson
    - Contains Line, Symbol, and Text features and each has the associated isDefault at the beginning of the file.
    - All line, symbol, and text features will display in a ERAM window within CRC.
  - Example_Lines_With_Overriding_Properties.geojson
    - Contains Line features but has overriding properties in one of the features.
    - All line features will display in a ERAM window within CRC with the values of `bcg=3, filters=3, style=Solid, thickness=1` except the one feature that has overriding properties which will display: `bcg=3, filters=4, style=Dashed, thickness=3`
      - You may notice that the feature with the overriding properties does not define the `bcg` property but will still display with `bcg=3`, that is because when the overriding properties do not define a certain property but that property is defined in the isDefault, it still takes on the characteristics laid out in the isDefault value.
  - Example_Lines_and_Text_but_no_isDefault_for_Text.geojson ???
    - Contains Line and text features but the text features have no isDefaults and therefore the text will not display in an ERAM window within CRC.
  - Example_Lines_and_Text_but_no_isDefault_for_Text_But_Has_Overriding_Properties.geojson ???
    - Contains Line and text features but the text features have no isDefaults and therefore the text will not display in an ERAM window within CRC except for the text feature that has overriding properties.
