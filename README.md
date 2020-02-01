# My AppMaker Graveyard

For two years, ever since [AppMaker](https://appmaker.google.com/) was introduced to the edu domain, I invested both personal time and professional credibility that this was a sustainable platform. It was a joy to use and was such a compeling platform.

In Janurary 2020, Google Cloud Platform [killed it](https://gsuiteupdates.googleblog.com/2020/01/app-maker-update.html). This is where I document what I built and what I learned. Among what I learned is that one cannot trust Google's internal processes. In an effort to go from teacher to independent consultant and developer, substantial time and effort was invested here.

I hope the work speaks for itself.

(Note: Will be adding more zips and explanations over time.)

* [The projects](https://github.com/classroomtechtools/appmaker_techniques#the-projects)
    * Splash Page
    * Daily Notices
    * Student Directory
    * Enrollment Figures
    * Grade Reporter
    * ...
* [AppMaker Tips & Techniques](https://github.com/classroomtechtools/appmaker_techniques#appmaker-techniques)
    * Building widgets
    * Customizing Datastores

# The projects

## Splash Page

This is the landing page used at the school where I work. We use it to collate links to various places on our intranet that different members of the community, grouped by audience, can use to find information. The audiences are grouped by organizational units.

It has the ability for admins to publish button-like links to different audicence / OUs, and for individual users to keep their own button links. Individuals can also utilize starring items in their Google Drive to collate their buttons. It works well on both desktop and mobile.

It is basically just a bookmarking tool with some basic CMS features. What makes it interesting from the user's perspective is that anytime they need something, instead of rooting through their inboxes looking for a link to something, they just consult the Splash Page. When management or administration need to publish a link, they can do so.

From a technical perspective, what makes it really interesting is that the source of truth used for determining permissions is the `orgUnitPath` exposed via `AdminDirectory`. This is not a feature that AppMaker had by default, but because it was "low code" instead of "no code" I was able to fashion a nice solution. (AppMaker's replacement is a no code platform.)

### Installation

You need to install the Splah Page from a super user account, or one that has permissions to Admin Directory.

1. Install the OU Service first, per [these instructions](#32-install-server-side-ou-service). When you install, note the address the OU service is published to.
2. [Download the zip file](SplashPageOpenSource_MIT.zip?raw=true), make a new project, import it
3. Open the `Ou` server-side file, and change line #4 to match your url for the OU service, and the password if you changed it.
2. Wait as the database is populated with all OUs found on your domain, and your super admin user is created
3. When "Complete" is shown, go to the hamburger menu and click on "Database", scroll down to where it says "Edit Audience" and inspect the settings for each OU.
    1. Make an OU active by flipping the active state
    2. Choose a meanginful material icon to use
4. From the menu, choose "Add a global tab" and give it a name.
5. After creating the new global tab, return to the database tab, look at "Categories" find the tab you just made, and check the box of which audiences the tab should displays to. (I never got around to adding this to the above end-user interface.)
5. From the menu, choose "Add a global button" and work through the user interface to add a button
6. There is a **bug** where, under some circumstances, you will not see any buttons until you click on one of the tabs. It has to do with the app saving the location for the next time, which isn't invoked until you click on one of the tabs. (This bug is occurs once.)
7. Go to the menu and see "Starred Items" which are your Drive files that you have starred. There is also an interface there where you can bulk unstar items.
8. Go to the menu and see the "Search All" which presents an interface that lets you filter global, private, and starred buttons in one place.
6. You should then be able to publish it to the domain and other users can log in and see the global tabs and buttons you made, as well as create their own.

### Photos

Desktop Version

![Desktop](splash_desktop.png?raw=true)

Mobile version

![Mobile](splash_mobile.png?raw=true)

## Daily Notices

In the same vein as the Spash Page that has the concept of audiences determined by the organizational units, instead of links this app publishes bulletin-like notifications. Authors can choose which group of individuals the notice is displayed to.

The idea here is to avoid the "death by email" phenomenon common in schools, where multiple emails an be sent in a single day. Academic staff like having one, collated place for any announcement for the day.

Features:

* Authors can add a notice in the future, and run for up to two weeks
* Incorporates [moment library](https://momentjs.com/) to override browser timezone behaviour to be for the intended timezone
* Older notices sink to the bottom of a day

Features I was currently building:

* Ability for users to subscribe to the collated notices on a daily basis, sent to their inbox via triggers
* Ability for users to attach Drive files
* Hashtags


## Installation instructions

1. This also need the OU Service.
2. [Download the zip](DailyNotices OpenSource_MIT.zip?raw=true), make new project, import
3. Switch to "Development" pane, and manually add the following to the "Audiences" datasource: `Students`, `Parents`, and `Staff`
    * This app is hard coded to only work with these three audiences
4. You can add notices in the future, determining how long they are displayed


### Photos

![Daily Notices](daily_notices.png?raw=true)

## Student Directory

In addition to offering a search tool to find student head shots and profile information, the Student Directory is also a workflow solution. A common operation in schools for management and administration is to email a group of students, and their parents. Or to email all the teachers who teach that student.

Using this tool, users can identity the students they are interested in, and copy and paste lists of email addresses so they can bring it into their email browser, etc.

A typical operation looks like this:

1. Search for "Grade 8"
2. Tick the boxes next to their names of the ones you are intending to email
3. Go to "Selected Students"
4. Scroll down to "Student Emails" and click to copy, paste into gmail
5. Scroll down to "Parent emails" and click to copy, paste into gmail

### External API

This tool integrates with ManageBac's API as the source of truth. Using a calculated datasource and a service written with AppScripts, the app quickly displays search results based on information from that external source, including a caching mechanism.

### Photos

Using

![Student Directory](student_directory.png?raw=true)



# AppMaker Techniques

A collection of some descriptions and accompanying code on various techniques that I've learned over two years of building with Google AppMaker.

Tips are outlined as below:

1. Building Widgets
	* 1.1 [Using Dropdowns to toggle fields displayed in a table](#11-using-dropdowns-to-toggle-fields-displayed-in-a-table)

2. Customizing Datastores
    * 2.1 [Using Spreadsheet values to prototype](#21-using-spreadsheet-values-to-prototype)
    * 2.2 [Optimizing Load times for Widgets using Calculated Datastores](#22-optimizing-load-times-for-widgets-using-calculated-datastores)
    * 2.3 [Handling permissions](#23-handling-permissions)

3. Utilizing Organization Units
	* 3.1 [On AppMaker's Directory Model](#31-on-appmakers-directory-model)
	* 3.2 [Install Server-side OU Service](#32-install-server-side-ou-service)
	* 3.3 [Storing results from OU Service in Settings datastore](#33-storing-results-from-ou-service-in-settings-datastore)

## 1 Building Widgets

### 1.1 Using Dropdowns to toggle fields displayed in a table

#### Best for mobile

You have a table whose fields can be changed by the user with a dropdown. For mobile, you want to conserve space, allowing the user to hide the column completely. For example, a table that lists students, and the user can choose to see the `homeroom` field of the datastore (the default), or change it to view the `grade` field or `gender` field.

```
┌ Dropdown ───────────┐
│                     │
│ Homeroom            │◀── Changing this dropdown's value
└─────────────────────┘
┌───────────────────────────┐
│ Name              ■  HR   │ (Checkbox toggle to hide)
├ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┤
│                           │
│ Happy Student       8M    │
│                           │◀────── Changes field values
│ Happiest Student    7R    │
│                           │
└───────────────────────────┘
```

Here is the setup for solution below:

1. The Dropdown widget and Table widget share the same datastore
2. The table row has Label widgets to display the values
3. The table header row has a Checkbox that displays the field name, and can hide the field (to conserve space on mobile)
4. The Checkbox uses abbreviations to describe the field

This is how to do that in AppMaker:

1. Set the Dropdown's `options` to be a javascript binding that is just an array of all the fields except the default one in the datastore: i.e. `['grade', 'gender']`. Notice that this list does not contain `homeroom`.
2. Set the Dropdown's `names` to be a cooresponding list of the values that is to displayed to the user as what they view as options, in this example: `['Grade', 'Gender']`
3. Set the Dropdown's `No selection` value to be the default as seen by the user, here the string value "Homeroom"
4. Set the Label's value to be a binding that dynamically returns the correct value, depending on whether Dropdown's value is null — or if not, the field in the datastore: `@widget.root.descendents.Dropdown.value ? @datastore.item[ @widget.root.descendents.Dropdown.value ] : @datastore.item.homeroom`

5. Set visibility of the Label to be bound to the Checkbox's value.
6. Set the Checkbox's text value to a javascript binding that returns an object that maps Dropdown's value to the respective feild abbreviations. If the default is selected, just returns the default's abbreviation:`
@widget.root.descrendants.Dropdown.value ? {'grade': 'G', 'gender': 'M/F'}[ widget.root.descrendants.Dropdown.value ] : 'HR'`

#### Extending for Desktop

On desktop view, you could combine the Dropdown, Checkbox, and Label into one object from the user's point of view. In that way, all of those mentioned would be placed in the table header. The above solution works best on mobile, but you can use breakpoints to adjust switch between the two interfaces.


## 2 Customizing Datastores

### 2.1 Using Spreadsheet values to prototype

It can be advantageous to use a spreadsheet to build a simple application, or to prototype for a more complicated app. It is also helpful to have randomized data for such purposes.

In this example, we use a very simple solution that uses a GSheet as the data's backbone. If there an error is encountered, it just outputs the error and tries to continue reading in.

#### Limitations:

* Does not support paging
* Large datasets will have performance degradation (due to lack of paging)

#### Getting Started:

1. In a Calculated datastore, choose **Query Script**, and enter `return DataFromSpreadsheet({spreadsheetId: '<id>'})`
    * If the spreadsheet uses **random numbers** (`RAND` formula and the like), and you want them to be recalculated on each datastore load, pass `true` to the `recalc` key of the object
2. In the fields in the Calculated datastore, ensure that **its fields match the headers in the spreadsheet**. Also each column's data type should be **consistent**, and **match** the expected ones as defined in the datastore. If any of these are wrong, it is likely to produce an error message in the console.
3. Paste the below Sample Code into a server-side script.
4. Test

#### Sample Code:

```js
/**
 * DataFromSpreadsheet: Read in Spreadsheet info for a Calculated Datasource in AppMaker. Use a spreadsheet to define a datasource.
 *                      Useful for data modeling, simple Apps.
 *                      Does not support paging; sheets with large number of rows will see performance penalties
 * @param {object} params
 * @param {string} params.spreadsheetId The ID of the source spreadsheet
 * @param {string} params.sheetName The name of the source sheet
 * @param {string} params.datasource The name of the target datasource
 * @param {number} params.numHeaders How many rows are headers (default = 1)
 * @param {number} params.headerRow Which row contains the name of the field (default=params.numHeaders-1)
 * @param {bool} params.recalc True if the spreadsheet should be to be recalculated (randomized) on each read in (default=False)
 * @returns {any}
 */
function DataFromSpreadsheet(params) {
  var ss, sheet, data, model;

  // Here we are preparing the default params available to this function
  // You can just manually put the ID in order to simply calling it from the datasources
  params = params || {};
  if (!params.spreadsheetId) throw Error("No spreadsheet ID provided in params");
  if (!params.sheetName) throw Error("No spreadsheet name provided in params");
  params.datasource = params.datasource || params.sheetName;
  params.numHeaders = params.numHeaders || 1;
  params.headerRow = params.headerRow || (params.numHeaders-1);
  params.recalc = params.recalc || false;
  //

  // Read in the spreadsheet document into ss variable using Google's APIs
  // If this fails, an error will appear in the console (error automatically thrown)
  ss = SpreadsheetApp.openById(params.spreadsheetId);
  //

  // Let's get the sheet we are looking for
  // If this failes, we manually throw an error which will apear in the console
  sheet = ss.getSheetByName(params.sheetName);
  if (!sheet) throw Error("No sheet by name of " + params.sheetName);
  //

  // Before we start reading in, force the spreadsheet to recalc, so if we use any random numbers
  // they are random for each read-in
  if (params.recalc) {
    sheet.getRange(1, 1).setValue(sheet.getRange(1, 1).getValue());
    SpreadsheetApp.flush();
  }
  //

  // Read in the raw data from the chosen sheet
  // We will loop through this in order to derive the correct return values
  data = sheet.getDataRange().getValues();
  //

  // We have to use the model as given in AppMaker's server-side API framework
  // The variable 'app' is provided to us by that framework
  model = app.models[params.datasource];
  if (!model) throw Error("No datasource with name " + params.sheetName);
  //

  // The LOOP. Not including the header, we step through each row of the data
  //           ensuring that we compile a record object that has the appropriate values
  //           The javascript function Array.reduce is used to efficiently loop through
  return data.slice(params.numHeaders).reduce(
    function (acc, row, index) {
      var record, ok;

      // Use the server-side API to make a new, blank record that we will fill in the inner loop
      record = model.newRecord();
      //

      // The inner loop. Go through each item in the row (value) to set the info
      // We use javascript's Array.every to loop as it can short circuit if an unexpected error occurs
      ok = row.every(function (value, index) {
        var header;
        header = data[params.headerRow][index];  // get the header by reading the first row
        try {
          record[header] = value;
        } catch (e) {
          console.log(e.message);   // print message
          return false;
        }
        return true;
      });
      //

      // Only add it to the array if above says it's okay (no errors occurred while trying)
      if (ok) acc.push(record);
      return acc;
      //
    }, []
  );
}

```

### 2.2 Optimizing Load times for Widgets using Calculated datastores

This is a technique that improves the loading of datastore data in an AppMaker application. This technique swaps out existing widgets that were first built with Calculated datastore with a Client-side Calculated (CC) datastore. It relies on `localStorage` to hold data in the browser, while also tracking version changes in the original Calculated datastore. If the local version number is lower than the stored version number, the original Calculated datastore is consulted in order to derive the new values (and then saves that and the new version number in `localStore`).

#### Use Case

Your application's data uses a google or other external API as the source of truth, and you'd like to improve the performance, i.e. the "snappiness" of your application.

#### Getting Started

1. Prepare to switch from a Server-side Calculated Datastore to a Client-side Datastore
2. Application logic to increase version stored on server
3. Create your Client-side Calculated (CC) Datastore
4. Write the CC Query script to juggle versioning

##### 2.2.1 Prepare to switch from a Server-side Calculated Datastore to a Client-side Datastore

The Server-side Calculated Datastore is completed the "normal" way at first. Then, when you've got your panels and widgets pointing to this calculated datastore correctly, you can switch those widgets to an equivalent CC datastore instead. You still have the original datastore around, which is the source of truth for our data.

You then create a caching mechanism between these two, where the CC datastore consults the original server-side ds whenever the local version is "dirty." So, in this way, a local version is kept on the browser for speed, but the datastore on the server is used, and then saved locally, and the local version is updated. This means we need to store some numerical versioning data locally, and on the server.

We'll use `localStorage` for local versioning info and the data itself. For the versioning info on the server, we'll use `PropertiesServices` (the server data will be the original store).

We might have globally-significant data and we might have user-significant data, and this is for the developer to answer. Is the datastore info per-user or global?

So, let's make an object that helps us do this. We'll pass it `'Script'` or `'User'` depending on the above, and the key at which to store this version info. We'll interact with the `Version` object in the following manner in our AppMaker application:

```js
// Server-side:
...
makeDataDirty();
...

// Client-side:
...
google.script.run.makeDataDirty();
...
```

This is the code that implements this, save as server-side script:

```js
var context = 'Script', key = 'key';

function makeDataDirty() {
  var versioning = Versioning(context, key);
  return versioning.increment();
}

function getDataVersion() {
  var versioning = Versioning(context, key);
  return versioning.get();
}

function Versioning(which, key) {

  var defaultValue = '0';

  function getProp() {
    if (['User', 'Script'].indexOf(which) === -1) throw Error("Expecting 'User' or 'Script'");
    return PropertiesService[ 'get' + which + 'Properties' ].call(PropertiesService);
  }

  /**
   * incrementVersion: Add one to User- or Script-context at {key}
   * @returns {number}
   */
  function incrementVersion() {
    var prop, value;

    prop = getProp(which);
    value = prop.getProperty(key);
    if (!value || isNaN(value)) {
      value = defaultValue;  // first value will be '1' on server but 0 on browser
    }
    value = (parseInt(value) + 1).toString();
    prop.setProperty(key, value);
    return parseInt(value);

  }

  /**
   * getVersion: Return the raw data at {which}-Property {key}
   * @returns {number}
   */
  function getVersion() {
    var prop, value;

    prop = getProp(which);
    value = prop.getProperty(key);
    if (!value || isNaN(value)) {
      // corner case at application start-up when it doesn't exist, so create it
      value = incrementVersion();
    }
    return parseInt(value);
  }

  var ret = {};
  ret.get = getVersion;
  ret.increment = incrementVersion;
  return ret;
}
```


##### 2.2.2 Application logic to increase version stored on server

Depending on how your application works, you'll need to indicate that the server-side data is "dirty" by increasing the version information. If the data is global across the application, you send in `'Script'` to the `Versioning` object, otherwise `'User'`. Then at the right place in your application, increment the version number.

Where to do this exactly depends on your application.

##### 2.2.3 Create your Client-side Calculated Datastore

The client-side datastore needs to have the exact same fields as your server-side calculated datastore. We also need to create a versioning object used on the browser, which mirrors how the above works. Instead of an `increment` function, we have a `set` one instead.

```js
function LocalVersioning(which, key) {
  var defaultValue = '-1';

  function getProp() {
    if (!window.localStorage || !window.localStorage.getItem(key)) {
      if (window.localStorage && !window.localStorage.getItem(key)) {
        // In this case, we just make the local storage object
        // so the next time it's called we return localStorage object (with default value)
        window.localStorage.setItem(key, defaultValue);
        return localStorage;
      }

      // return a mock object with default value
      return {
        getItem: function (key) {
          return defaultValue;
        },
        setItem: function () {
          // nothing
        }
      };
    } else {
      return localStorage;
    }
  }

  /**
   * Reset
   */
  function resetVersion() {
    var prop;
    prop = getProp();
    prop.setItem(key, defaultValue);
    return defaultValue;
  }

  /**
   * incrementVersion: Set version to User- or Script-context at {key}
   * @param {number} value
   * @returns {number}
   */
  function setVersion(value) {
    var prop;

    prop = getProp(which);
    if (!value || isNaN(value)) {
      value = defaultValue;
    }
    prop.setItem(key, value.toString());
  }


  /**
   * getVersion: Return the raw data at {which}-Property {key}
   * @returns {number}
   */
  function getVersion() {
    var prop, value;

    prop = getProp(which);
    value = prop.getItem(key);
    if (!value || isNaN(value)) {
      // corner case at application start-up when it doesn't exist, so create it
      value = resetVersion();
    }
    return parseInt(value);
  }

  var ret = {};
  ret.get = getVersion;
  ret.set = setVersion;
  return ret;
}
```

##### 2.2.4 Write the CC Query script to juggle versioning

Now we need the CC datastore Query script to wrap this all up. We'll have one large function with nested functions to make it convenient. The body of this is the important part, which is discussed below:

```js
function ExecuteCCDatastore(query, factory, callback) {

  var context = 'Script', key = 'nothing111111111111';
  var localVersioning = LocalVersioning(context, key + 'Version'); // must be different from key
  var calculatedServerSide = app.datasources.C;

  /**
   * Called to allow for filter to occur. Beyond the scope of this tutorial
   */
  function filterPass(records) {
    if (!query.parameters.StringFilter) return records;
    return records.filter(function (record) {
      return record.Display.toLowerCase().indexOf(query.parameters.StringFilter.toLowerCase()) !== -1;
    });
  }

  function fromServer() {
    console.log('fromServer');
    var store, values = [];
    calculatedServerSide.load({
      success: function () {
        store = calculatedServerSide.items.map(function (item) {
          var record, value = {}, pmap;
          record = factory.create();
          Object.keys(calculatedServerSide.model.fields).filter(function (field) {
            return field[0] !== '_';  // filter out '_values'
          }).forEach(function (field) {
            record[field] = item[field];  // add to both the object to be saved and the record we need
            value[field] = item[field];
          });
          values.push(value);
          return record;
        });

        // save to browswer
        console.log(JSON.stringify(values));
        if (window.localStorage) window.localStorage.setItem(key, JSON.stringify(values));
        callback.success(filterPass(store));
      },
      failure: function (err) {
        callback.failure(err);
      }
    });


  }

  /**
   *
   */
  function fromLocal () {
    console.log('fromLocal');
    // see if anything available in store, if not, fallback to fromOriginal
    var ret, values;

    if (!window.localStorage || !window.localStorage.getItem(key)) return fromServer();

    try {
      values = JSON.parse(window.localStorage.getItem(key));
    } catch (e) {
      // issue with parsing (corruption?), reset
      localVersioning.reset();
      return fromServer();
    }
    ret = values.map(function (value) {
      var record;
      record = factory.create();
      Object.keys(value).forEach(function (field) {
        record[field] = value[field];
      });
      return record;
    });

    callback.success(filterPass(ret));
  }

  google.script.run
    .withSuccessHandler(function (serverVersion) {
       var dirty = localVersioning.get() < serverVersion;

       if (dirty) {
         // will query the server, need to load it first
         fromServer();

       } else {
         // will read in from localStorage
         // although if error is found redirects to fromServer()
         fromLocal();
       }
       localVersioning.set(serverVersion);
    })
    .getDataVersion();

}
```

On the first run, the application will use the server to get the results. On the second run, it will use the info stored in localStorage. It will use the latter until `makeDataDirty` is invoked.

### 2.3 Handling permissions

I work with schools, which require server-side handling of data so that information that is not intended for the end user is not even downloaded from the datastore. This is the manner in which I implement this feature.

The way I solve server-side permissions is to have an Audience datasource with id as primary key and Name as string. Then other datastores have many-to-many relationship to that Aduience datastore.

Then server-side, I can filter them out.

```js
query.filters.Audience.Name._in = [‘SchoolOne’];
//or
query.filters.Audience.Name._equals = ‘SchoolOne’;
```

This means that each datasource has to be populated with relevant Audience information for each item.

It also means you need a way to get “SchoolOne” to your server side, which you can do with properties (parameters). In my case, on the AppLoad section, I actually go through each datasource in the app, then it automatically feeds each of them the organizational unit info, which I have at load time.

```js
// executes during app load time:
var orgUnit = getOrgUnit();
Object.keys(app.datasources).filter(function (key) {
  return key.indexOf(‘_’) !== 0;  // weed out internal properties
}).forEach(function (name) {
  var ds;
  ds = app.datasources[name];
  if (ds .properties && ds.properties.hasOwnProperty(‘orgUnit’))
    ds.properties.orgUnit = orgUnit;
});
```

That way I can just do this in the server side script:

```
query.filters.Audience.Name._in = [query.parameters.orgUnit];
//or
query.filters.Audience.Name._equals = query.parameters.orgUnit;
```

### 3. Utilizing Organization Units

The organizational unit is a very useful thing to have enabled on your AppMaker apps. The way I added this ability was to enable permissions dependingo on the role assigned in the Google Directory Organizational Unit model.

#### 3.1 On AppMaker's Directory Model

This is a bit of rant: You would think that the build-in Directory Model available would allow for an easy way for developers to access to [Directory API's orgUnitPath field](https://developers.google.com/admin-sdk/directory/v1/reference/orgunits). The OU is such a fundamental element to access restrictions that will be available to the end user, and knowing that data point through an AppMaker instance is pretty much vital to pretty much every single app I have written.

#### 3.2 Install Server-side OU Service

In order to erect a service that AppMaker can use to know which organizational unit the current logged in user belong to, we'll launch a web service via Google Apps Scripts that works around its current lack of feature.

The following script can be installed on your domain. You'll need to have `AdminDirectory` advanced service enabled, and it'll need to be deployed as a web app with access "Anyone, even anonymous." Note that this open permission is necessary in order to work properly from an AppMaker instance.

```js
var SECRET = 'longpassword';  // ensure that you have some sort of passcode for a simple layer of security

function doGet(e) {
  var jsonString, result;

  if (!e.parameter.secret || e.parameter.secret !== SECRET || !e.parameter.user) {
  	result = '{error: "Permission denied"}';
  } else  {
	  try {
	    result = AdminDirectory.Users.get(e.parameter.user);
	  } catch (e) {
	    result = '{error: "' + e.message.replace(/"/g, ' ') + '"}';
	  }
  }
  jsonString = JSON.stringify(result);
  return ContentService.createTextOutput(jsonString).setMimeType(ContentService.MimeType.JSON);
}

function testDoGet() {
  var e = {};
  e.parameter = {user: 'useremailaddress'};
  e.parameter.secret = SECRET;
  var result = JSON.parse(doGet(e).getContent());
  Logger.log(typeof result);
  Logger.log(result.orgUnitPath);
}
```

You can use the `testDoGet` function to ensure it's working propertly.


#### 3.3 Storing results from OU Service in Settings datastore

Now that we have an OU service enabled on our domain, we utilize it in an AppMaker server-side code.

From a server-side code in AppMaker, you would need to call it with something like the following:

```js
var response = UrlFetchApp.fetch('https://script.google.com/.../exec?secret=longpassword&user=' + app.user.email);
var json = JSON.parse(response);
var mainOU = json.orgUnitPath.split('/')[1]; // get the "main" OU
```

Now that you have the org unit of the current logged in user, you can take action accordingly. For example, maintain a `Settings` datastore for every logged in user, and store the OU information there.
