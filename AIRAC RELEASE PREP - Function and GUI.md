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
     - If downloading from URL, the first line should be ".FeUseOnly" in order to verify the download was successful. Example:
       ```
       .FeUseOnly This line is here for Facility Engineer automation systems. This must be the first line of the alias file and should not be edited.
       ```

3. **Alias File Preparation**:
   - Prompts the user to select which alias files from the `FE-BUDDY_Output\ALIAS` folder they want included in the combined alias file.
     - Saves these selections as a preference for this facility ID for future use and automatically selects the previously selected files if they exsist.
     - This program will not prompt the user to be able to select "AliasTestFile.txt" and "DUPLICATE_COMMANDS.txt".
   - Combines the custom alias command file with the user-selected alias files from the FE-Buddy folder into a single alias file `Combined_<facility ID>_Alias.txt` and outputs it to the `FE-BUDDY_Output\UPLOAD_TO_vNAS` directory.
     - The custom alias command file content will be written to the final combined file first.

4. **Video Maps Preparation**:
   - Copies selected .geojson files from the `FE-BUDDY_Output\CRC` directory to the `FE-BUDDY_Output\UPLOAD_TO_vNAS\VIDEO_MAPS` directory.
   - Allows optional renaming of .geojson files and formatting (single-line or pretty-print).
   - The user can add specific CRC Defaults values to each geojson file.

5. **Error Handling and Logging**:
   - Provides feedback on the status of each operation and logs errors encountered during the process.

## Desired Functionality

### General Requirements

- **Local AppData Folder**: Create a local appdata folder `%localappdata%\Ksanders7070_AIRAC_RLS_PREP` for storing preference files for this program. The preference file should be named `AIRAC_RLS_PREP_prefs.json`.
- **Preference File**:
   - When the user defines the facility they are working on and clicks Save, a section in the preference file should be created for that facility ID.
   - Preferences should only be written to the preference file when the user selects a Save button. Quitting the application without saving changed data should prompt the user to save changes before exiting.
   - The program should not be able to "Begin AIRAC Release Prep" until all required data is input and saved.
   - The preference file should contain the last facility ID used for automatic population of the appropriate preferences on the next launch.
   - The preference file should contain per facility ID:
     - Last update date and time.
     - What was changed or updated.
     - A detailed log of every change for the past 200 changes along with their date and time.
     - Alias file local directory or URL, including the token if provided.
     - Each geojson file previously selected for transfer to the UPLOAD_TO_vNAS folder along with the renamed name, single-line or pretty-print option, and any CRC Defaults data.
- **Version Management**:
   - A `currentVersion` variable should be created in a single place, such as the `AppSettings` document, and referenced where needed.
   - The application should check for the latest version available on GitHub using the GitHub API.
- **Preferences Management**: Users should be able to save and load preferences based on Facility ID entry. If the facility ID is recognized as a previously saved ID, the preferences associated with that ID should be automatically loaded.
- **Error Handling**: Proper error handling for all user interactions, especially file and directory selections. The FE-Buddy output folder should always begin with "FE-BUDDY_Output" and should have at least two subdirectories: "CRC" and "ALIAS". If these directories are not found or the directory does not begin with "FE-BUDDY_Output", prompt the user to reselect the appropriate folder.

### GUI Design

#### General Design

- **Theme**: Dark mode theme similar to GitHub.
- **Layout**: Use tabs to organize different functionalities. Each tab should contain related settings and information.
- **Tabs**: The tabs should be color-coded to highlight that there is required information that needs to be input and saved on that tab.

#### Tabs

1. **Home Tab**:
   - **Description**: Default tab presented on launch. Provides an overview of the program and basic user instructions.
   - **Features**:
     - Detailed description of what the program does.
     - Display the current version number (e.g., v1.0.0).
     - Display the latest version available on GitHub or a message saying that it is still probing for the latest version available and eventually a message saying that the latest version could not be found if it couldn't be found.
     - Button/hyperlink to get the latest version if available.
     - Button to report issues (links to GitHub issues page).
     - Button to the changelog.md in the main branch GitHub page.
     - Instructions to review and fill out all applicable data in each of the tabs before conducting the AIRAC Prep process.

2. **Facility Tab**:
   - **Description**: Second tab.
   - **Features**:
     - Facility ID entry field with save button.
     - Button to launch a folder browser to select the FE-BUDDY_Output folder (kept for the session only).

3. **Aliases Tab**:
   - **Description**: Third tab. Manage alias command files.
   - **Features**:
     - Input for the location of the custom alias command file (local directory or URL with optional TOKEN).
     - Radio buttons to select alias files from the `FE-BUDDY_Output\ALIAS` directory for inclusion in the final facility alias file output (Combined_<facility ID>_Alias.txt), excluding "AliasTestFile.txt" and "DUPLICATE_COMMANDS.txt".
     - Save button to write preferences.

4. **Video Maps Tab**:
   - **Description**: Fourth tab. Manage video map files.
   - **Features**:
     - Radio buttons to select .geojson files from the `FE-BUDDY_Output\CRC` directory for inclusion in the `FE-BUDDY_Output\UPLOAD_TO_vNAS\VIDEO_MAPS` directory.
     - Option to edit the name of selected .geojson files.
     - Option to compress the output file into a single line or pretty-print (single-line by default).
     - Option to add "CRC Defaults" to the geojson:
       - If SYMBOLS, define values: `bcg`, `filters`, `style`, `size`.
       - If TEXT, define values: `bcg`, `filters`, `size`, `underline`, `opaque`, `xOffset`, `yOffset`.
       - If LINES, define values: `bcg`, `filters`, `style`, `thickness`.
     - Save button to write preferences.

5. **Prep Tab**:
   - **Description**: Fifth tab. Conduct the AIRAC prep process.
   - **Features**:
     - If all required data isn't input and saved, a list of required data that needs to be filled out displays for the user and the "Start prep" button is greyed out.
     - Un-grey the Start Prep button if all required data is input and saved.
     - After the Start Prep button is selected, display a checklist to show the progress of the prep process as it happens.
     - Error handling and status updates during the prep process.
     - Instructions similar to the batch script at the end of the process that are still applicable.

#### Additional Features

- **Handling Missing Files**:
  - If the FE-Buddy output folder does not contain a previously created .geojson, list the file as greyed out with options to edit or delete preferences for it.
- **Preferences Management**:
  - If a previously selected file's feature types have changed, prompt the user to update or delete irrelevant preferences. If new feature-types are contained in the file, prompt the user to provide default data for that new type. Users are permitted to not have defaults for each type of feature but still have it for some within the same file.
  - Validate input values for all user entries, especially for CRC Defaults, to ensure correctness.
