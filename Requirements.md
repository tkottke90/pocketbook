# Pocketbook CLI Tool

Developers are commonly using their web browsers to research varying piecies of a project.  When those developers are working on multiple projects at the same time those browser tabs can get out of hand.  This can lead to loss of useful information which requires time to recover through history searching or re-searching the same topics to find the article again.

This project has an effect on developer productivity which they could be using to design and develop the systems they were researching.

The Pocketbook ClI Tool looks to mitigate this by providing developers a place to store URLs within their repositories and open them again in the future.  

### Terminology

*CLI* - Acronym for Command Line Tool.  This is a tool used by a terminal.

*JSON* - Javascript Object Notation.  This is a file formated from a Javascript object.  This file can be read directly by Javascript using common js `require`, ecmascript 6 modules `import`, or by reading the file as plain text.

### Goals

- Provide a command line tool that developers can used to store their code 
- Users should be provided with a simple graphical interface to work with the collection of documents
- Users should be able to store data in a pockebook file (`*.pocketbook.json`) OR within a `package.json` file with the property name `pocketbook`.
- Users should be able to create configurations when one does not exist
- Users should be able to group records
- Users should be able to open individual records in their browser
- Users should be able to open groups of records in their browser

### Future Goals

- Provide a markdown output of configured records
- Better note management

## Solution

At this time I could not find a CLI level tool to complete the same task.

The Pocketbook CLI tool solution is designed to provide developers with a simple CLI tool that will interface with their existing projects as either a stand alone file or integrated with the npm `package.json` file.

Users will initiate the CLI tool within a directory and if the directory contains a valid configuration, it will be loaded and the user will be allowed to start interacting with the records.  If not, the user will be prompted to generate a new configuration.

#### Data Model

The data will be stored within a Javascript object either in a JSON file or as part of the `package.json`.  The core model of this application is the record itself.  Records can be organized into groups for ease of access later.  The application has one top level group with the name `root` which is the entrypoint for the application when it displays options to the user.

##### Records

A record represents a url that the user would like to access at a later date.

```json
{
  "path": "https://google.com"
}
```

Records require the `path` parameter which will be used by the application to open the url.  In addition the following properties can be included:

| Name | Description | Type |
| --- | --- | --- |
| `"label"` | Label of the url listed within the cli tool | string |
| `"notes"` | Array of notes related to a specific entry | string[] |

##### Groups

A group represents a collection of urls.

```json
{
  "label": "root",
  "canDelete": false,
  "records": [
   ...Records Here...
  ] 
}
```

Groups require a `label` and `records`.  These are used to display the group and it's children.  In addition the following properties can be included:

| Name | Description | Type |
| --- | --- | --- |
| `"notes"` | Array of notes related to a specific entry | string[] |
| `"canDelete"` | Determines if the group can be deleted by the application | boolean |

#### Business Logic

The core logic of the tool focuses on navigating the JSON structure to select urls and open them in the browser.  The application will need to detect the UNIX system in use and use the correct command:

| System | Command |
| --- | --- |
| darwin | `open <url>` |
| linux | `xdg-open <url>` |

Upon completion of a successful execution the application should update the JSON to reflect any changes generated within the runtime.  To ensure that changes are not missed the application should catch `SIGTERM` and `SIGKILL` commands and create a dot file copy of the configuration in the working directory.  This file should be checked for on script load.

##### Configuration

There are 2 types of configuration used by the application.  The first is directory configurations.  This is detected by the current working directory (`process.cwd()`) when the application is run.  This will be in the `package.json` or `*.pocketbook.jsson` files.

The second type of configuration will be application configuration.  These configurations are specific to the Pocketbook CLI.  These will live within the `bin/` directory adjacent to the application files themselves.  They will need to be generated if missing at runtime to avoid being added to version control.

#### Presentation

Users will navigate inquirer interfaces to interact with the the data model.

##### Navigation

The application will navigate between states based on the configuration of the JSON tree.  Users will be provided a list of options to page through using inquirer:

```
~/project
> pocketbook
? Root URL Group:
  Open All            <-- Group Option to open all
  Edit Group
  ---
  Testing [G]         <-- Group
  http://google.com
> Facebook            <-- Labeled Record 
```

Groups will be noted with a `[G]` letting users know they can drill down to see further options.

##### View: Record

When the user selects a record, the application will use the appropreate command to launch a prompt providing the user options about the link:

```
~/project
> pocketbook
? Select a record or group? Facebook
? What do you want to do:
> Open URL (https://facebook.com)
  Edit URL
  Delete URL 
  ---
  Back
```

When `Open URL` is selected the application will send a command to open the url in the users preferred browser.  After the route is selected, the application should return the user to the previous view.

When `Edit URL` is selected, the application will prompt the user to enter a new url for this record.  This will be updated in the file when the applicaton closes.

When `Delete URL` is selected, the application will prompt the user to verify that they wish to delete the record, and if confirmed remove the record.  This will be updated in the file when the application closes.  After the route is selected, the application should return the user to the previous view.

##### View: Group

When the user selects a group, the application will display a list of options related to the group, a seporator, and the list of items within that group.

```
~/project
> pocketbook
? Root URL Group:
  Open All        
  Edit Group
  Add Sub-Group
  Add URL
  Back
  ---
  Testing [G]         <-- Sub-Group
  http://google.com
> Facebook
```

When `Open All` is selected, the application will iterate over the records collection and open any records listed.  This will include any child records that live in sub-groups

When `Edit Group` is selected, the application will display the `View: Group_Details`.  This will include additional actions that can be taken on the group.

When `Create Sub-Group` is selected, the application will display the `View: New_Group`.  This will include prompts to create a new sub-group under this group.

When `Add URL` is selected, the application will display the `View: Record`.  Users can then modify the url record using the interface.

##### View: Group_Details

```
~/project
> pocketbook
? Select a record or group? Facebook
? What do you want to do:
> Edit Name
  Delete Group
  Empty Records 
  ---
  Back
```

When `Edit Name` is selected, the application will prompt the user to enter a new name for the group.

When `Delete Group` is selected the application will first check if the `canDelete` option is `true` for the group.  This protection ensures that groups are not deleted by accident.  Additionally it protects the root group.

When `Empty Records` is selected, the array of records is purged.


### Test Plan

Primary goal of the test plan will be to modularize the functionality to the extent that individual modules can be unit tested using MochaJS.  Integration testing will need to be done using Bash scripts.

### Monitoring

Users should be prompted when the application is installed if they want to opt-into usage data logging to provide data to a server for tracking and enhancement development.

### Deployment

Application will be packaged and deployed using `npm publish` to the npm registry for distrubution.

### Rollback Plan

No rollback plan will be implemented at this time.

### Alternative Solutions

Alternative solutions users could be using today include:
- Text files with a bash script
- External list of links (such as a personal wiki)
- Note taking apps such as MS Notepad or Notable

## Technical Requirements/MVP

<ol>
  <li>Application should require NodeJS 12.18 or newer</li>
  <li>Application should be installable via <code>npm link</code></li>
  <li>Application should maintain a configuration file</li>
  <ol type="1">
    <li>Application should create configuration file if it does not exist at runtime</li>
    <li>Application should prompt user to update configs when new config is created</li>
    <ol>
      <li>Users should be prompted to opt in or out of monitoring</li>
    </ol>
    <li>Application should allow users to access configuration by running <code>pocketbook config</code></li>
  </ol>
  <li>Application should detect if the current working directory contains a valid configuration in the following order:</li>
  <ol>
    <li>Application should check to see if there is a pocketbook file (<code>*.pocketbook.json</code>)</li>
    <li>Application should check to see if there is a <code>package.json</code> file and if it contains a pocketbook config</li>
  </ol>
  <li>When no configuration is detected, a new one should be created with user prompts</li>
  <ol>
    <li>User should be given the choice between saving to external file or <code>package.json</code></li>
    <li>Application should create configuration with a root group</li>
  </ol>
  <li>Application should allow users to start the application by running <code>pocketbook</code></li>
  <li>Records:</li>
  <ol>
    <li>Application should allow users to add records</li>
    <li>Application should allow users to open records</li>
    <li>Application should allow users to edit record urls</li>
    <li>Application should allow users to edit record labels</li>
    <li>Application should allow users to delete records</li>
    <li>Application should allow users to add notes to a record</li>
  </ol>
  <li>Groups</li>
  <ol>
    <li>Application should allow users to add groups</li>
    <li>Application should allow users to open all records in a group</li>
    <li>Application should allow users to edit group labels</li>
    <li>Application should allow users to delete a group</li>
    <li>Application should allow users to delete all records from a group</li>
    <li>Application should allow users to add notes to a group</li>
  </ol>
</ol>
