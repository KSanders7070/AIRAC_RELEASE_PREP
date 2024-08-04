# GeoJSON File Structure and Rules for CRC Software

## File Composition

- GeoJSON files represent various types of spatial data elements for VATSIM air traffic control simulations using the CRC software.
- In this document, we will cover LineString (including MultiLineString), Symbol (Point), and Text (Point) features. CRC is capable to rendering polygon features but only under certain circumstances and we will not cover that here.

## Feature Types and Properties

- **Line**: Represented by `LineString` or `MultiLineString`. Properties may include `style` (e.g., solid, dashed), `thickness`, `bcg`, and `filters`.
  - Example:

    ```json
    {"type":"FeatureCollection","features":[{"type":"Feature","geometry":{"type":"LineString","coordinates":[[-117.02,32.6],[-116.99,32.57]]},"properties":{"bcg":3,"filters":[3],"style":"Solid","thickness":1}}]}
    ```

- **Symbol**: Represented by `Point`. Properties can be `style` (denoting different navigational symbols like VOR, NDB), `size`, and occasionally, `color`.
  - Example:

    ```json
    {"type":"Feature","geometry":{"type":"Point","coordinates":[90.0,180.0]},"properties":{"isSymbolDefaults":true,"bcg":18,"filters":[18],"style":"airport","size":1}}
    ```

- **Text**: Represented by `Point`. Properties include `text` (an array of strings for multiline text), and optionally `size`, `underline`, `xOffset`, `yOffset`, and `opaque`.
  - Example:

    ```json
    {"type":"Feature","geometry":{"type":"Point","coordinates":[90.0,180.0]},"properties":{"isTextDefaults":true,"bcg":18,"filters":[18],"size":1,"underline":false,"opaque":false,"xOffset":12,"yOffset":0}}
    ```

## Defaults and Structure

- Defaults for lines, symbols, and text are defined in specific `Point` features with properties like `isLineDefaults`, `isSymbolDefaults`, and `isTextDefaults`. These are not rendered but set default properties for subsequent features.
- Default features should be placed at the beginning of the GeoJSON file for clarity and organizational purposes.
- These defaults are often referred to as "isDefaults" or "CRC Defaults".
- When the CRC Defaults nor the feature properties define a required properties key/value, CRC assigns the following by automatically:
  - **LINE:**
    - Filters = (not assigned)
    - BCG = 1
    - Style = solid
    - Thickness = 1
  - **SYMBOL:**
    - Filters = (not assigned)
    - BCG = 1
    - Style = vor
    - Size = 1
  - **TEXT:**
    - Filters = (not assigned)
    - BCG = 1
    - Text = (not assigned)
    - Size = 1
    - Underline = false
    - xOffset = 0
    - yOffset = 0
    - Opaque = false

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
    - All line features will display in a ERAM window within CRC with the values of `bcg=3, filters3, style=Solid, thickness=1` except the one feature that has overriding properties which will display: `bcg=3, filters=4, style=Dashed, thickness=3`
      - You may notice that the feature with the overriding properties does not define the `bcg` property but will still display with `bcg=3`, that is because when the overriding properties do not define a certain property but that property is defined in the isDefault, it still takes on the characteristics laid out in the isDefault value.
  - Example_Lines_and_Text_but_no_isDefault_for_Text.geojson ???
    - Contains Line and text features but the text features have no isDefaults and therefore the text will not display in an ERAM window within CRC.
  - Example_Lines_and_Text_but_no_isDefault_for_Text_But_Has_Overriding_Properties.geojson ???
    - Contains Line and text features but the text features have no isDefaults and therefore the text will not display in an ERAM window within CRC except for the text feature that has overriding properties.
