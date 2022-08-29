---
title: How to build a real-time chat app with Google Sheets
categories : [Programming]
tags :  [appscript, tutorial]
description: Learn to build a real-time chat app with Google Sheets.
comments: true
math : true
---
 
Having scarce knowledge of backend techonologies, I decided to build [1bxoxb1](https://github.com/creme332/1bxoxb1), a simple anonymous chat app, with Google Sheets.  The back-end code, written entirely in Google App Script, surprisingly took less than [40 lines of code](https://github.com/creme332/1bxoxb1/blob/main/appscript/Code.gs). 

 <img src="/assets/appscript-app/website.gif" height ="500" width="600" alt = "GIF of chat app">
_A real-time no-login online chat app_

> [Github repository](https://github.com/creme332/1bxoxb1)
>
> [Live demo](https://script.google.com/macros/s/AKfycbz3_jQSeelwylADfsLTh0hqxqqkGsoYyRG19UilpUde2dDJCjv79H3tOjrJMzp_DxrpbA/exec)
{: .prompt-tip }

The goal of this post is to document the process to build this project.

# Getting started
- Basic knowledge of HTML, CSS and JS is required.
- Watch a video on the basic [features](https://www.youtube.com/watch?v=KR4cmAkB1KI&ab_channel=GoogleWorkspace) of the online AppScript IDE.
- Read about [HTML templates and scriplets](https://developers.google.com/apps-script/guides/html/templates).
- Read about the [best practices](https://developers.google.com/apps-script/guides/html/best-practices) in AppScript.
- Watch this [tutorial](https://www.youtube.com/watch?v=RRQvySxaCW0&ab_channel=LearnGoogleSheets%26ExcelSpreadsheets) on how to interact with Google Sheets. 
 
# Build project
 
## Create a Google Sheets database
 
- Go to [Google Sheets ](https://docs.google.com/spreadsheets/u/0/) to create a spreadsheet. The spreadsheet name is not important.
- Open your spreadsheet and change the first empty row to :
 
date | username | message
--|--|--|
|
|
 
## Create AppScript project
-  Go to the [AppScript dashboard](https://script.google.com/) to create and open a new appscript project. The project name is not important.
- There is currently a single file `Code.gs` in your project. Create 3 new **HTML** files to make your file structure look like this :
```
Files
│_   Code.gs
|_   Index.html
|_   Stylesheet.html
|_   JavaScript.html
```

 
## Write server-side code
 
>The server-side code is  written in the `Code.gs`{: .filepath } file.
{: .prompt-info }
 
### Add spreadsheet information
 
Add 3 **global** constants to store database information.
 
```js
const SPREADSHEET_URL = "PLACE YOUR URL HERE";
const spreadsheet = SpreadsheetApp.openByUrl(SPREADSHEET_URL);
const worksheet = spreadsheet.getSheetByName("YOUR SHEET NAME");
```
 
### Add boilerplate code
 
```js
function doGet() {
  return HtmlService
      .createTemplateFromFile('Index')
      .evaluate();
}
 
function include(filename) {
  return HtmlService.createHtmlOutputFromFile(filename)
    .getContent();
}
```
 
The `doGet()` function for templated HTML generates an `HtmlTemplate` object from the HTML file. Then `doGet()` calls its `evaluate()` method to execute the scriptlets and convert the template into an `HtmlOutput` object that the script can serve to the user.
 
The custom server-side `include()` function imports the `Stylesheet.html` and `JavaScript.html` file content into the `Index.html` file. When called using printing scriptlets, this function imports the specified file content into the current file.
 
### Update database with data from client
 
When a user sends a message, the  server-side function `addNewRowToSheet()` is called from the client-side.
 
`addNewRowToSheet()` will append the date, the name of the sender, and the message to the spreadsheet.
```js
function addNewRowToSheet(username, user_input) {
  worksheet.appendRow([new Date().toString(), username, user_input]);
}
```
>The date must be in string format so that it can be passed to the client later. Read more about legal parameters in appscript [here](https://developers.google.com/apps-script/guides/html/reference/run#myFunction(...)).
{: .prompt-warning }
 
### Send spreadsheet to client
 
`getSpreadsheetData()` will return a 2D array containing all database information to the client.
 
```js
function getSpreadsheetData() {
  //get data from first three columns
  const AllData = worksheet.getRange("A:C").getValues();
 
  //remove column heading
  AllData.shift();
 
  // remove empty rows from AllData and return result
  return AllData.filter(function (el) {
    return el[0] != "";
  });
}
```
 
## Write client-side code
 
### HTML
 
>Edit the `Index.html`{: .filepath } file.
{: .prompt-info }

>Add  `<?!= include('Stylesheet'); ?>`{: .filepath } in head tag and   `<?!= include('JavaScript'); ?>`{: .filepath } just before **end of body tag**.
{: .prompt-tip }
 
```html
<!DOCTYPE html>
<html>
<head>
  <base target="_top">
  <?!= include('Stylesheet'); ?>
</head>
<body>
  <div class="container">
    <div id="message-container"></div>
    <div class="bottom-container">
      <textarea placeholder="Type a message" id="input-container" cols="30" rows="1"></textarea>
      <button id="sendbtn"> > </button>
    </div>
  </div>
  <?!= include('JavaScript'); ?>
</body>
</html>
```

<img src="/assets/appscript-app/wireframe.png" height ="500" width="600" alt = "Wireframe of html page">
_Wireframe of the website_
 
### CSS
 
>Edit the `Stylesheet.html`{: .filepath } file.
{: .prompt-info }

The CSS code is available [here](https://github.com/creme332/1bxoxb1/blob/main/appscript/Stylesheet.html). The code itself is not important and can be modified.
 
>Remember to include `<style>`{: .filepath } tags in  `Stylesheet.html`{: .filepath }.
{: .prompt-tip }

### JavaScript
 
>Edit the `JavaScript.html`{: .filepath } file.
{: .prompt-info }

I will explain only the important parts of the code. The full JS code is available [here](https://github.com/creme332/1bxoxb1/blob/main/appscript/JavaScript.html). 

>Remember to include `<script>`{: .filepath } tags in  `Stylesheet.html`{: .filepath }.
{: .prompt-warning }
 
#### Send message to database

Each time the `send-btn` element is clicked, `saveToSpreadsheet()` is called. `saveToSpreadsheet()`takes input from the `input-container` and calls the server-side function `addNewRowToSheet()` which then saves the message to the database.

```js
function saveToSpreadsheet() {
    //ignore empty messages
    if (userInputBox.value == "") return;
   
    //send username and message to spreadsheet
    google.script.run.addNewRowToSheet(MY_USERNAME, userInputBox.value);
   
    //reset userInputBox
    userInputBox.value = "";
}
sendButton.addEventListener("click", saveToSpreadsheet)
```
#### Update messages in real-time
Every `REFRESH_RATE` milliseconds, the following line is executed :
 
```js
 google.script.run.withSuccessHandler(onSuccess).getSpreadsheetData();
```
 
- `script.run.getSpreadsheetData()` calls the server side function `getSpreadsheetData()`.
 
- The `withSuccessHandler(onSuccess)`part will call the function `onSuccess()` if the server-side function returns successfully. The return value of `getSpreadsheetData()`, which is a 2D array,  becomes the argument of the `OnSuccess()` function.
 
- `onSuccess()` will look for new messages since the last time `getSpreadsheetData()` was called. These messages are then displayed in the `message-container` element.
 
```js
  function updateMessages() {
    function onSuccess(spreadsheetDataArray) {
      //loop through new rows since last update
      const lastRowIndex = spreadsheetDataArray.length - 1;
      for (let i = currentRowIndex + 1; i <= lastRowIndex; i++) {
        let date = spreadsheetDataArray[i][0];
        let user = spreadsheetDataArray[i][1];
        let msg = spreadsheetDataArray[i][2];
        let timePosted = getTime(date);
 
        // code to add the above information to message-container
        // ....
      }
      currentRowIndex = lastRowIndex;
    }
    google.script.run.withSuccessHandler(onSuccess)
      .getSpreadsheetData();
  }
  setInterval(updateMessages, REFRESH_RATE);
```

> The number of messages currently being displayed in the message-container is  $ \text{currentRowIndex + 1} $,  where $ \text{currentRowIndex} $ is a global variable.
{: .prompt-info }
 
 >The default value of `REFRESH_RATE`{: .filepath } is 2000 milliseconds. It can be reduced to make refreshing of messages faster but in doing so, the number of concurrent users on the chat app will be reduced.
 {: .prompt-danger }

## Deploy project
- Deploy your project as a web app.
- The web app requires you to authorize access to your data.

>If you face the  “Sorry unable to open file at this moment” problem, try opening the web app in incognito. Read [this](https://www.codejam.info/2022/04/google-apps-script-unable-to-open-the-file.html) for more information.
{: .prompt-tip }

# Limitations
 
## Project
- A maximum of 30 concurrent users is allowed.
- Google Sheet API allows at most 300 requests per minute.
 
## AppScript Online IDE
- No version control system like Git.
- No keyboard shortcuts available like in VSCode.
- `Rename Symbol` option was not working at the time when I wrote my code.

