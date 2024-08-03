# AIRAC Release Prep

## Basic Description

- **Program Name**: AIRAC Release Prep
- **Executable Name**: AIRAC Release Prep.exe
- **Purpose**: Assists in preparing ARTCC facility files for upload to the vNAS Data-Admin site every AIRAC cycle.
- **GitHub Repository**: [AIRAC Release Prep](https://github.com/KSanders7070/AIRAC_RELEASE_PREP)

## General Function

The AIRAC Release Prep performs the following tasks:

1. **Directory Management**:
   - Creates preference .json files per facility ID in the `%localappdata%\Ksanders7070_AIRAC_RLS_PREP` directory.
   - A temp directory (`%temp%\ksanders7070_airac_rls_prep_temp`) may be created and is deleted at program close. If found at launch, the contents will be deleted.
   - Creates a sub-directory `UPLOAD_TO_vNAS` within the user-selected FE-Buddy_Output folder. If one already exists, advise the user that the contents of the UPLOAD_TO_vNAS folder will be deleted prior to beginning any processing of new files, allowing the user to cancel and try again if need-be.
     - Will also create another sub-directory `UPLOAD_TO_vNAS\Video Maps`
   - Moves processed and prepared files to the appropriate directories for upload to the vNAS server.
     - Combined Alias file: `FE-Buddy_Output\UPLOAD_TO_vNAS`
     - GeoJSONs/Video Maps: `FE-Buddy_Output\UPLOAD_TO_vNAS\Video Maps`

2. **Data Retrieval**:
   - Retrieves required data from the user-selected FE-Buddy Output folder.
   - Retrieves the facility's Custom Alias command file from either a URL or local directory that the user specifies.
     - If downloading from URL, the first line should be ".FeUseOnly" in order to verify the download was successful. The user should be notified of this requirement. Example of this line at the top of the custom alias command file:

       ```
       .FeUseOnly This line is here for Facility Engineer automation systems. This must be the first line of the alias file and should not be edited.

       ; ZZZ Custom Alias Command created by Joe Someone
       ; Updated 54JAN20.22
       .MyFirstActualCommand goes here and so on.
       .ThereIsAnEasierWayToDoDownloadVerification Yea, I know but screw it, I am coding this on my breaks at work and don't feel like putting in the time right now.

       ...
       ```

3. **Alias File Preparation**:
   - Prompts the user to select which alias files from the `FE-BUDDY_Output\ALIAS` folder they want included in the combined alias file.
     - Saves these selections as a preference for this facility ID for future use and automatically selects the previously selected files if they exsist.
       - If a previously selected alias file is not found in the fe-buddy output folder, still display it to the user but have it greyed out with an option to have the user delete it from the preference file, otherwise it is just ignored.
     - This program will not prompt the user to be able to select "AliasTestFile.txt" and "DUPLICATE_COMMANDS.txt".
   - Combines the custom alias command file with the user-selected alias files from the FE-Buddy folder into a single alias file `Combined_<facility ID>_Alias.txt` and outputs it to the `FE-BUDDY_Output\UPLOAD_TO_vNAS` directory.
     - The custom alias command file content will be written to the final combined file first.

4. **Video Maps Preparation**:
   - Prompts the user to select which geojson files from the `FE-BUDDY_Output\CRC` folder they want included in the UPLOAD_TO_vNAS folder.
     - Saves these selections as a preference for this facility ID for future use and automatically selects the previously selected files if they exsist.
       - If a previously selected geojson is not found in the fe-buddy output folder, still display it to the user but have it greyed out with an option to have the user delete it from the preference file, otherwise it is just ignored.
   - Copies the user-selected .geojson files from the `FE-BUDDY_Output\CRC` directory to the `FE-BUDDY_Output\UPLOAD_TO_vNAS\VIDEO_MAPS` directory.
   - Allows optional renaming of .geojson files and formatting (single-line or pretty-print).
   - The user can also add specific CRC Defaults values to each geojson file.

5. **Error Handling and Logging**:
   - Provides feedback on the status of each operation and logs errors encountered during the process.

## Desired Functionality

### General Requirements

- **Local AppData Folder**: Create a local appdata folder `%localappdata%\Ksanders7070_AIRAC_RLS_PREP` for storing preference files for this program. The preference file should be named `AIRAC_RLS_PREP_prefs.json`.
- **Preference File**:
  - When the user defines the facility they are working on and clicks Save, a section in the preference file should be created for that facility ID.
  - Preferences should only be written to the preference file when the user selects the Save button.
  - The preference file should retain the last facility ID used for automatic population of the appropriate preferences on the next launch.
  - The preference file should contain per facility ID:
    - Last update date and time.
    - What was changed or updated.
    - A detailed log of every change for the past 50 changes along with their date and time for troubleshooting purposes.
    - Alias file local directory or URL, including the token if provided.
    - Each geojson file previously selected for transfer to the UPLOAD_TO_vNAS folder along with the renamed name, single-line or pretty-print option, and any CRC Defaults data.
- **Version Management**:
  - A `currentVersion` variable should be created in a single place, such as the `AppSettings` document or similar appropriate document, and referenced where needed.
  - The application should check for the latest version available on GitHub using the GitHub API and provide the user a notice when there is an update available.
- **Preferences Management**:
  - Users should be able to save and load preferences based on Facility ID entry.
  - If the facility ID is recognized as a previously saved ID, the preferences associated with that ID should be automatically loaded.
- **Error Handling**:
  - Proper error handling for all user interactions, especially file and directory selections.
  - The FE-Buddy output folder should always begin with "FE-BUDDY_Output" and should have at least two subdirectories: "CRC" and "ALIAS".
    - If these directories are not found or the directory does not begin with "FE-BUDDY_Output", prompt the user to reselect the appropriate folder.

### GUI Design

#### General Design

- **Theme**: Dark mode theme similar to GitHub.
- **Layout**: Use tabs to organize different functionalities. Each tab should contain related settings and information.
- **Tabs**:
  - The tabs should be color-coded to highlight that there is required information that needs to be input and saved on that tab.
    - Red = missing required data, Yellow = new data has been entered but not saved.
    - If there is missing required information in the tab but also new entries that are not saved, the tab should be red rather than yellow.
- **Save Button**:
  - A button that is available to the user regardless of what tab they are in that saves all inputs to the preference file.
  - Should be greyed out or not displayed if the AIRAC Prep is in progress.
  - Red = missing required data, Yellow = new data has been entered but not saved.
    - If one of the tabs has missing data and the other tab has new entry infomration, the save button should be red rather than yellow.
- **Misc**:
  - Quitting the application without saving changed data should prompt the user to provide them the chance to save changes before exiting.

#### Tabs

1. **Home Tab**:
   - **Description**: Default tab presented on launch. Provides an overview of the program and basic user instructions.
   - **Features**:
     - Detailed description of what the program does.
     - Display the current version number (e.g., v1.0.0).
     - Display the latest version available on GitHub.
       - While waiting for the latest version to be retrieved, display a message saying that it is still searching for the latest version available.
       - If the latest version could not be found, change the message to state so.
       - If the current users version is the most up to date availble, state "you are running the most up to date version" or something similar.
     - Button/hyperlink to get the latest version if available.
     - Button to report issues (links to GitHub issues page).
     - Button to the changelog.md in the main branch GitHub page.
     - Instructions to review and fill out all applicable data in each of the tabs before conducting the AIRAC Prep process.

2. **Facility Tab**:
   - **Description**: Second tab.
   - **Features**:
     - Facility ID entry field.
       - A list of available previously saved facility IDs may be warrented.
     - Button to launch a folder browser to select the FE-BUDDY_Output folder (kept for the session only and is not written to the preferences file).

3. **Aliases Tab**:
   - **Description**: Third tab. Manage alias command files.
   - **Features**:
     - Input for the location of the custom alias command file
       - If URL: input area for the URL with optional TOKEN field.
       - If local file: open a file browser and have the user select the facility custom alias file.
     - Radio buttons to select alias files from the `FE-BUDDY_Output\ALIAS` directory for inclusion in the final/combined facility alias file
       - Exclude "AliasTestFile.txt" and "DUPLICATE_COMMANDS.txt" from the available selection of alias file from the FE-Buddy Output folder.

4. **Video Maps Tab**:
   - **Description**: Fourth tab. Manage video map files.
   - **Features**:
     - Radio buttons to select .geojson files from the `FE-BUDDY_Output\CRC` directory for inclusion in the `FE-BUDDY_Output\UPLOAD_TO_vNAS\VIDEO_MAPS` directory.
     - Option to edit the name of selected .geojson files.
     - Option to compress the output file into a single line or pretty-print (single-line by default).
     - Option to add "CRC Defaults" to the geojson.
       - The program will read the geojson file and determine what types of features are contained within the geojson and provide the user the option to assign values per type of features.
         - More than one type of feature may be contained in a geojson.
           - If SYMBOLS, define values: `bcg`, `filters`, `style`, `size`.
           - If TEXT, define values: `bcg`, `filters`, `size`, `underline`, `opaque`, `xOffset`, `yOffset`.
           - If LINES, define values: `bcg`, `filters`, `style`, `thickness`.
       - The program will then insert the appropriate CRC Defaults at the beginning of the geojson in accordance with the Understanding CRC GeoJSON.md:

5. **Prep Tab**:
   - **Description**: Fifth tab. Conduct the AIRAC prep process.
   - **Features**:
     - If all required data isn't input and saved, a summary list of required data that needs to be filled out displays for the user and the "Start prep" button is greyed out or does not display.
     - After the Start Prep button is selected, display a checklist to show the progress of the prep process as it happens.
     - The Start Prep button name should turn to "Cancel" once the process is running and selecting this cancel button should cancel the process and turn back into Start Prep.
         - Start Prep process should always clear the UPLOAD_TO_vNAS folder prior to starting to ensure any previous files are not there anymore.
     - Error handling and status updates during the prep process.
     - Once the AIRAC Prep is complete, display instructions similar to the batch script ending.
     - The program should not be able to "Start Prep" until all required data is input and saved.

#### Additional Features

- **Handling Missing Files**:
  - If the FE-Buddy output folder does not contain a previously selected .geojson or alias file, list the file as greyed out with options to edit or delete preferences for it.
- **Preferences Management**:
  - If a previously selected file's feature types have changed, prompt the user to update or delete irrelevant preferences. Examples:
    - A previous run of this program said that videomap1.geojson had LINE features and the user had values for the LINES CRC defaults for videomap1.geojson in the preferences files, but with this new run of the program, the videomap1.geojson file now has lines and symbols.
      - The user should be provided a notice what has changed and provided the opportunity to enter values for the symbols now or ignore it.
    - The same logic should be given in the situation where the previously selected geojson had LINES and TEXT and now only has LINES.
      - The user should be notified that this change has occured and prompt the user to delete that preference that was applicable to the TEXT CRC defaults or ignore it.
  - Users are permitted to not have defaults for each type of feature but still have it for some within the same file. For example:
    - If videomap2.geojson has LINES, SYMBOLS, and TEXT, the user may have CRC Defaults for one ore more of them or none at all.
  - Validate input values for all user entries, especially for CRC Defaults, to ensure correctness.
