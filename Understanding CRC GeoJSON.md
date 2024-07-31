### GeoJSON File Structure and Rules for CRC Software:

#### File Composition:
- GeoJSON files represent various types of spatial data elements, including Lines, Symbols, and Texts, for VATSIM air traffic control simulations using the CRC software.

#### Feature Types and Properties:
- **Line**: Represented by `LineString` or `MultiLineString`. Properties may include `style` (e.g., solid, dashed), `thickness`, `color`, `bcg`, and `filters`.
  - Example: 
    ```json
    {"type":"Feature","geometry":{"type":"LineString","coordinates":[[90.0,180.0],[91.0,181.0]]},"properties":{"isLineDefaults":true,"bcg":1,"filters":[1],"style":"Solid","thickness":3}}
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

#### Defaults and Structure:
- Defaults for lines, symbols, and text are defined in specific `Point` features with properties like `isLineDefaults`, `isSymbolDefaults`, and `isTextDefaults`. These are not rendered but set default properties for subsequent features.
- Default features are placed at the beginning of the GeoJSON file for clarity and organizational purposes.
- These defaults are often referred to as "isDefaults".

#### Handling Multiple Types:
- Files often contain mixed feature types which can complicate management. Utility scripts can split a single GeoJSON into multiple files based on feature type, each retaining relevant defaults at the top.

#### Detailed Property Values for GeoJSON Features:
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

#### Usage of Defaults:
- Defaults are specified in specific 'Point' features marked by properties like `isSymbolDefaults`, `isLineDefaults`, and `isTextDefaults`. These defaults apply to subsequent features of the same type unless explicitly overridden by individual feature properties.  
- Coordinates for isDefaults should always equal: 90.0,180.0
