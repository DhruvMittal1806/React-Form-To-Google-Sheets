# Submit Your React Form Responses to Google Sheets
How to submit React forms to Google Sheets using Google Script API.
This is an extension to **submiting HTML forms to google sheets** made by [Jamie Wilson](https://github.com/jamiewilson).
You can check that out at [Form To Google Sheets](https://github.com/jamiewilson/form-to-google-sheets).

## Setting up google sheets
### Creating Google Sheet
1. Create a new blank sheet on [Google Sheets](https://docs.google.com/spreadsheets)
2. Put the following header into first row: 


| | A  | B | C |
| --- | ------------- | ------------- | ------------- |
| 1 | timestamp | Name | Email  |

### Creating Google App Script
1. click on `Tools > Script Editor`
2. Rename the script title and wait for it to save.
3. Instead of `function myFunction()` paste the following code in `code.gs` tab.
  
  ```js
  var scriptProp = PropertiesService.getScriptProperties()

  function intialSetup () {
    var activeSpreadsheet = SpreadsheetApp.getActiveSpreadsheet()
    scriptProp.setProperty('key', activeSpreadsheet.getId())
  }

  function doPost (e) {
    var lock = LockService.getScriptLock()
    lock.tryLock(10000)

    try {
      var doc = SpreadsheetApp.openById(scriptProp.getProperty('key'))
      var sheet = doc.getSheetByName(e.parameters.sheetName); //Adds support for multiple forms.

      var headers = sheet.getRange(1, 1, 1, sheet.getLastColumn()).getValues()[0]
      var nextRow = sheet.getLastRow() + 1

      var newRow = headers.map(function(header) {
        return header === 'timestamp' ? new Date() : e.parameter[header]
      })

      sheet.getRange(nextRow, 1, 1, newRow.length).setValues([newRow])

      return ContentService
        .createTextOutput(JSON.stringify({ 'result': 'success', 'row': nextRow }))
        .setMimeType(ContentService.MimeType.JSON)
    }

    catch (e) {
      return ContentService
        .createTextOutput(JSON.stringify({ 'result': 'error', 'error': e }))
        .setMimeType(ContentService.MimeType.JSON)
    }

    finally {
      lock.releaseLock()
    }
  }
  ```
  
  > If you want to get an in-depth understanding of this checkout [form-script-commented.js](https://github.com/jamiewilson/form-to-google-sheets/blob/master/form-script-commented.js) from [Jamie Wilson](@jamiewilson) repo. In the above code I have just added support for multiple forms.

  
  4. **Run** the setup function and authorise with the google account used to setup the sheet.
  5. You might see a dialog box asking for permission by the `Script Title you earlier renamed`. Click on allow and move further.
  
  ### Adding a new Trigger
  1. On the left side toolbar Click on `Trigger`(Alarm Clock Icon) and click on `Add Trigger`
  2. In **Choose which function to run** choose `doPost` and in *Select event type* choose `on form submit` and click **save**.
  
  ### Deploying the Script
  1. Click on `Deploy > New Deployment`.
  2. On **Select Type** click on settings icon and choose `web app`.
  3. Add a suitable description (optional).
  4. Leave the **execute as** field to `Me (your email address)`.
  5. Change the **Who has access** to `Anyone`.
  6. Click on `Deploy` and copy the `web app URL` from the popup and click on Okay.

  ## Setting up your Form
  Paste the following code in your form component.
  
  ```jsx
  import { useState, React } from React;
  
  export default function form() {
    const scriptURL = '' // Paste web app URL copied earlier

    const [status, setStatus] = useState("submit");
    const handleSubmit = async (e) => {
      e.preventDefault();
      setStatus("Sending...");

      let formData = new FormData();

      const { Name, Email } = e.target.elements;
      let details = {
        Name: Name.value,
        Email: Email.value,
      };
      // Similarly add other fields above

      for(var key in details) {
        formData.append(key, details[key]);
      }

      fetch(scriptURL, {method: 'POST', body: formData})
      .then(response => {
        console.log('Success!', response);
        setStatus("Form Submitted");
      })
      .catch(error => {
        console.error('Error!', error.message);
        setStatus("Retry");
      })
    };

    return (
      <form name="submit-to-google-sheet" onSubmit={handleSubmit}>
        <input type="text" name="sheetName" value="Sheet1" style={{ display: `none` }}> // Sheet Name where you want this form to submit too
        <input type="text" name="Name" placeholder="Name" id="Name" required />
        <label for="Name">Name</label>
        <input type="email" name="Email" placeholder="Email" id="Email" required />
        <label for="Email">Email</label>
        <!-- Add Other Fields -->
        <button type="submit">{status}</button>
      </form>
    )
  }
  ```
  
  ### Adding Additonal Fields and Forms
  #### Additional Fields
  1. Add aditional headers in Google Sheets in the first row of the Google Sheet
  2. Add new fields in the form in the similar way described in the code above, just remeber to use the exact same name (Case Sensitive) for the field you used in Google Sheet's header.

  #### Additional Forms
  1. You should make a seperate class for handling form responses.
  2. Create another sheet in the same google sheets file and similarly add the headers in the first row.
  3. Then create a new form similar to what we created the first time and change the sheetName value to the Google sheet's name you created in the previous step (Case Sensitive) and call the handleSubmit() function on submit btn.
  

  
  
  
  
  


