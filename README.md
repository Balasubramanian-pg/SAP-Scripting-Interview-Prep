# SAP-Scripting-Interview-Prep
Why god? Why Put me through this?


Here’s a **comprehensive SAP Scripting cheat sheet** covering SAP GUI Scripting, SAP TAO (Test Automation Framework), and SAP Fiori automation. This is tailored for SAP test automation, RPA, and repetitive task automation:

---

## **1. SAP GUI Scripting Basics**
### **Prerequisites**
- Enable scripting in SAP GUI:
  - **SAP GUI Options** → **Accessibility & Scripting** → Check **"Scripting"** and **"Enable Script Tracking"**.
- Install **SAP GUI Scripting API** (included with SAP GUI installation).
- Supported languages: **VBScript, JavaScript, Python (via `win32com`)**.

### **Connecting to SAP**
#### **VBScript Example**
```vbs
' Create SAP GUI application object
Set SapGuiAuto = GetObject("SAPGUI")
Set app = SapGuiAuto.GetScriptingEngine

' Connect to an existing session
Set connection = app.Children(0)
Set session = connection.Children(0)

' Log in to SAP
session.findById("wnd[0]/usr/txtRSYST-BNAME").Text = "USERNAME"
session.findById("wnd[0]/usr/pwdRSYST-BCODE").Text = "PASSWORD"
session.findById("wnd[0]").sendVKey 0 ' Enter key
```

#### **Python Example (using `win32com`)**
```python
import win32com.client

# Connect to SAP
SapGuiAuto = win32com.client.GetObject("SAPGUI")
app = SapGuiAuto.GetScriptingEngine
connection = app.Children(0)
session = connection.Children(0)

# Log in
session.findById("wnd[0]/usr/txtRSYST-BNAME").Text = "USERNAME"
session.findById("wnd[0]/usr/pwdRSYST-BCODE").Text = "PASSWORD"
session.findById("wnd[0]").sendVKey(0)  # Enter key
```

---
## **2. SAP GUI Object Model**
### **Hierarchy**
```
SapGuiAuto (Application)
├── Connection (e.g., app.Children(0))
├── Session (e.g., connection.Children(0))
└── Window (e.g., session.Children(0))
    ├── Toolbar
    ├── MenuBar
    ├── StatusBar
    └── GUI Elements (TextField, Button, Table, etc.)
```

### **Common Object Types**
| Object Type | Description | Example ID |
|-------------|-------------|------------|
| **TextField** | Input field | `wnd[0]/usr/txtRSYST-BNAME` |
| **PasswordField** | Password input | `wnd[0]/usr/pwdRSYST-BCODE` |
| **Button** | Clickable button | `wnd[0]/tbar[0]/btn[0]` |
| **CheckBox** | Checkbox | `wnd[0]/usr/chkFLAG` |
| **RadioButton** | Radio button | `wnd[0]/usr/radGROUP` |
| **TableControl** | SAP table (ALV Grid) | `wnd[0]/usr/cntlGRID/shellcont/shell` |
| **TabStrip** | Tab control | `wnd[0]/usr/tabsTABSTRIP` |
| **ComboBox** | Dropdown list | `wnd[0]/usr/ctxtCOMBO` |

---
## **3. Finding Element IDs**
### **Methods**
1. **Script Tracking**:
   - Enable in SAP GUI Options.
   - Perform actions in SAP; IDs are logged in `C:\Users\<user>\AppData\Local\SAP\SAP GUI\script\<session>.txt`.

2. **SAP GUI Inspector** (Press `F12` in SAP GUI):
   - Hover over elements to see their IDs.

3. **Trial and Error**:
   - Use `session.Children` to explore the object hierarchy.

### **Example: List All Windows**
```vbs
For Each window In session.Children
    MsgBox window.Name & " - " & window.Id
Next
```

---
## **4. Common SAP GUI Scripting Commands**
### **Navigation**
```vbs
' Open a transaction
session.StartTransaction "SE38"

' Enter text in a field
session.findById("wnd[0]/usr/ctxtRSCRIPT-NAME").Text = "ZTEST_PROGRAM"

' Click a button
session.findById("wnd[0]/tbar[0]/btn[0]").Press  ' Execute button

' Send a key (e.g., Enter, F8)
session.findById("wnd[0]").sendVKey 0  ' Enter key
session.findById("wnd[0]").sendVKey 8  ' F8 key
```

### **Working with Tables (ALV Grid)**
```vbs
' Get table object
Set table = session.findById("wnd[0]/usr/cntlGRID/shellcont/shell")

' Get row count
rowCount = table.RowCount

' Select a row
table.currentRow = 2  ' Select row 3 (0-based index)

' Get cell value
cellValue = table.GetCellValue(1, "MATNR")  ' Row 2, Column "MATNR"

' Set cell value
table.SetCellValue 1, "MATNR", "100-100"  ' Row 2, Column "MATNR"

' Double-click a row
table.selectedRows = "1"  ' Select row 2
table.DoubleClickCurrentCell
```

### **Handling Popups**
```vbs
' Check for popup windows
If session.Children.Count > 1 Then
    popup = session.Children(1)
    If popup.Name = "SAPLSPOP" Then  ' Standard popup
        popup.findById("btn[0]").Press  ' Click "Yes"
    End If
End If

' Handle error messages
If session.findById("wnd[1]/usr/btnSPOP-OPTION1", False) Is Nothing Then
    ' No error popup
Else
    session.findById("wnd[1]/usr/btnSPOP-OPTION1").Press  ' Click "OK"
End If
```

### **Working with Tab Strips**
```vbs
' Select a tab
session.findById("wnd[0]/usr/tabsTABSTRIP/tabpTAB1").Select

' Get active tab
activeTab = session.findById("wnd[0]/usr/tabsTABSTRIP").SelectedTab
```

---
## **5. SAP TAO (Test Automation Framework)**
### **Basics**
- **SAP TAO** is SAP’s official test automation tool (successor to **eCatt**).
- Supports **SAP GUI, Fiori, and Web Dynpro**.
- Integrates with **SAP Solution Manager**.

### **TAO Script Structure**
```xml
<!-- Example TAO Script (XML-based) -->
<TestScript>
  <Transaction code="SE38">
    <Screen id="1000">
      <Field name="RSCRIPT-NAME" value="ZTEST_PROGRAM"/>
      <Button name="BTN_EXECUTE" press="true"/>
    </Screen>
  </Transaction>
</TestScript>
```

### **Key TAO Commands**
| Command | Description |
|---------|-------------|
| `<Transaction>` | Start a transaction (e.g., `SE38`). |
| `<Screen>` | Interact with a screen (ID from `F12`). |
| `<Field>` | Set a field value. |
| `<Button>` | Click a button. |
| `<Table>` | Interact with ALV grids. |
| `<Check>` | Validate a value (e.g., `<Check field="MATNR" value="100-100"/>`). |
| `<Loop>` | Repeat actions (e.g., for each row in a table). |
| `<Call>` | Call another script. |

### **Example: Log in and Run a Report**
```xml
<TestScript>
  <Transaction code="SE38">
    <Screen id="1000">
      <Field name="RSCRIPT-NAME" value="ZSALES_REPORT"/>
      <Button name="BTN_EXECUTE" press="true"/>
    </Screen>
    <Screen id="2000">
      <Field name="BDC_OKCODE" value="/n"/>  ' Proceed
    </Screen>
  </Transaction>
  <Check>
    <Field name="STATUS" value="Completed Successfully"/>
  </Check>
</TestScript>
```

### **Running TAO Scripts**
1. **Via SAP Solution Manager**:
   - Upload script to **Test Suite**.
   - Schedule execution.
2. **Via Command Line**:
   ```bash
   tao.exe -script="C:\Scripts\test_script.xml" -user="USERNAME" -password="PASSWORD"
   ```

---
## **6. SAP Fiori Automation**
### **Tools**
- **SAP Fiori Elements**: Standardized UI patterns.
- **UI5 (SAPUI5)**: JavaScript framework for Fiori apps.
- **Selenium WebDriver**: For browser-based automation.
- **Cypress/Playwright**: Modern alternatives to Selenium.

### **Example: Automate Fiori App with Selenium (Python)**
```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

# Launch browser
driver = webdriver.Chrome()
driver.get("https://fiori.example.com")

# Log in
driver.find_element(By.ID, "USERNAME_FIELD").send_keys("USERNAME")
driver.find_element(By.ID, "PASSWORD_FIELD").send_keys("PASSWORD")
driver.find_element(By.ID, "LOGIN_BUTTON").click()

# Wait for app to load
WebDriverWait(driver, 10).until(
    EC.presence_of_element_located((By.ID, "shell-appTitle"))
)

# Click a tile
driver.find_element(By.XPATH, "//div[@title='Sales Orders']").click()

# Fill a form
driver.find_element(By.ID, "input--customerId").send_keys("1000")
driver.find_element(By.ID, "input--orderDate").send_keys("10/10/2023")
driver.find_element(By.ID, "createButton").click()
```

### **Fiori Element IDs**
- Use **Browser DevTools (F12)** to inspect elements.
- SAPUI5 controls often have IDs like:
  - `__xmlview0--input--customerId`
  - `__item0-shell-appTitle`

### **Handling Dynamic IDs**
```python
# Find by partial ID
driver.find_element(By.CSS_SELECTOR, "[id*='input--customerId']")

# Find by text
driver.find_element(By.XPATH, "//span[contains(text(), 'Save')]")
```

---
## **7. Error Handling & Debugging**
### **Common Errors**
| Error | Cause | Solution |
|-------|-------|----------|
| **"Object not found"** | Incorrect element ID. | Use `F12` to verify ID or use `session.Children` to explore. |
| **"Session not available"** | SAP GUI not open or script not connected. | Check `app.Children` for active connections. |
| **"Popup blocked"** | Unexpected popup. | Add popup handling logic (check `session.Children.Count`). |
| **"Permission denied"** | Scripting not enabled in SAP GUI. | Enable in SAP GUI Options → Accessibility & Scripting. |
| **"Invalid transaction code"** | Typo in transaction code. | Verify the transaction code (e.g., `SE38` vs. `SE80`). |

### **Debugging Tips**
- **Log everything**:
  ```vbs
  Sub Log(message)
      Dim fso, file
      Set fso = CreateObject("Scripting.FileSystemObject")
      Set file = fso.OpenTextFile("C:\Logs\SAP_Script.log", 8, True)  ' 8 = Append
      file.WriteLine Now & ": " & message
      file.Close
  End Sub
  ```
- **Use `MsgBox` for quick checks**:
  ```vbs
  MsgBox "Current screen: " & session.findById("wnd[0]").Text
  ```
- **Pause execution**:
  ```vbs
  WScript.Sleep 5000  ' Pause for 5 seconds
  ```

---
## **8. Best Practices**
1. **Use `Try-Catch` Blocks**:
   ```vbs
   On Error Resume Next
   ' Risky operation
   If Err.Number <> 0 Then
       Log "Error: " & Err.Description
       Err.Clear
   End If
   On Error GoTo 0
   ```
2. **Wait for Elements**:
   ```vbs
   Sub WaitForElement(ByVal elementId, ByVal timeout)
       Dim startTime
       startTime = Timer
       Do While Timer < startTime + timeout
           If Not session.findById(elementId, False) Is Nothing Then Exit Do
           WScript.Sleep 500
       Loop
       If Timer >= startTime + timeout Then
           Log "Timeout waiting for " & elementId
           Exit Sub
       End If
   End Sub
   ```
3. **Modularize Scripts**:
   - Split scripts into reusable functions (e.g., `Login`, `NavigateToTransaction`).
4. **Avoid Hardcoding**:
   - Use config files or variables for credentials, transaction codes, etc.
5. **Clean Up**:
   - Close popups and sessions properly:
     ```vbs
     Do While session.Children.Count > 1
         session.Children(1).Close
     Loop
     ```

---
## **9. SAP GUI Scripting vs. TAO vs. Fiori Automation**
| Feature | SAP GUI Scripting | SAP TAO | Fiori Automation |
|---------|-------------------|---------|-------------------|
| **Language** | VBScript, Python | XML | JavaScript, Python (Selenium) |
| **Use Case** | Quick automation, RPA | Test automation | Fiori/Web apps |
| **Maintenance** | High (brittle IDs) | Medium | Low (stable selectors) |
| **SAP Support** | Unofficial | Official | Official (Selenium) |
| **Recording** | No | Yes (via TAO Recorder) | Yes (Selenium IDE) |
| **Integration** | Standalone | SAP Solution Manager | CI/CD pipelines |

---
## **10. SAP Scripting for RPA (UiPath, Blue Prism, Automation Anywhere)**
### **UiPath Example**
1. **Use "SAP GUI Scripting" activity pack**.
2. **Connect to SAP**:
   - Use **"Attach to SAP Session"** activity.
3. **Interact with Elements**:
   - Use **"Set Text"**, **"Click"**, **"Get Text"** activities.
4. **Example Workflow**:
   - Log in → Navigate to `VA01` → Fill sales order → Save.

### **Blue Prism Example**
1. **Use "SAP GUI Automation" VBO**.
2. **Actions**:
   - **Connect**: Attach to SAP session.
   - **Set Text**: Enter data in fields.
   - **Press Button**: Click buttons.
3. **Error Handling**:
   - Use **Recover/Resume** stages for popups.

---
## **11. SAP BDC (Batch Data Communication)**
### **Basics**
- Used for **mass data uploads** (e.g., master data, transactions).
- **Two Methods**:
  - **Call Transaction (BDC_OKCODE)**: Simulates user input.
  - **Direct Input (BDC_INSERT)**: Faster, but limited to specific transactions.

### **BDC Scripting Example (ABAP)**
```abap
DATA: lt_bdcdata TYPE TABLE OF bdcdata,
      ls_bdcdata LIKE LINE OF lt_bdcdata.

* Fill BDC table
PERFORM fill_bdc_table USING 'XK01' 'RF02D-KNA1'.

* Call transaction
CALL TRANSACTION 'XK01' USING lt_bdcdata MODE 'A'.
```

### **BDC Recording (SHDB)**
1. Run transaction `SHDB`.
2. Record steps (e.g., creating a vendor in `XK01`).
3. Generate ABAP code for replay.

---
## **12. SAP GUI Scripting for Excel Automation**
### **Example: Export SAP Table to Excel**
```vbs
' Create Excel object
Set excel = CreateObject("Excel.Application")
Set workbook = excel.Workbooks.Add
Set sheet = workbook.Sheets(1)

' Get SAP table
Set table = session.findById("wnd[0]/usr/cntlGRID/shellcont/shell")

' Export to Excel
For i = 0 To table.RowCount - 1
    For j = 0 To table.ColumnCount - 1
        sheet.Cells(i + 1, j + 1).Value = table.GetCellValue(i, j)
    Next
Next

' Save and close
workbook.SaveAs "C:\Temp\SAP_Data.xlsx"
excel.Quit
```

---
## **13. Common SAP Transactions for Automation**
| Transaction | Description | Use Case |
|-------------|-------------|----------|
| **SE38** | ABAP Editor | Run reports/programs. |
| **SE80** | Object Navigator | Navigate ABAP objects. |
| **SE16/SE16N** | Data Browser | Query tables (e.g., `KNA1` for customers). |
| **VA01/VA02/VA03** | Sales Orders | Create/change/display sales orders. |
| **MM01/MM02/MM03** | Material Master | Create/change/display materials. |
| **XK01/XK02/XK03** | Vendor Master | Create/change/display vendors. |
| **FB01/FB02/FB03** | Financial Posting | Post accounting documents. |
| **SU01** | User Maintenance | Create/modify users. |
| **PFCG** | Role Maintenance | Assign authorizations. |
| **SM37** | Background Jobs | Monitor/schedule jobs. |
| **SM35** | Batch Input | Process BDC sessions. |

---
## **14. SAP Scripting Interview Questions**
### **Theory**
1. **What is SAP GUI Scripting?**
   - Automation of SAP GUI using VBScript/Python to interact with SAP screens.

2. **What are the limitations of SAP GUI Scripting?**
   - **Brittle**: IDs change with SAP updates.
   - **No Official Support**: SAP does not guarantee compatibility.
   - **Performance**: Slower than BDC or direct APIs.

3. **What is the difference between SAP GUI Scripting and SAP TAO?**
   - **SAP GUI Scripting**: Quick automation (unsupported).
   - **TAO**: Official test automation framework (XML-based, integrated with SAP Solution Manager).

4. **How do you handle dynamic element IDs in SAP?**
   - Use `session.Children` to explore the object hierarchy dynamically.
   - Avoid hardcoding IDs; use loops to find elements by properties (e.g., `Name`, `Type`).

5. **What is BDC, and when would you use it?**
   - **Batch Data Communication**: Used for mass data uploads (e.g., master data). Prefer over scripting for large datasets.

### **Practical**
1. **Write a script to log in to SAP and open transaction `SE38`.**
   ```vbs
   Set SapGuiAuto = GetObject("SAPGUI")
   Set app = SapGuiAuto.GetScriptingEngine
   Set connection = app.Children(0)
   Set session = connection.Children(0)

   ' Log in
   session.findById("wnd[0]/usr/txtRSYST-BNAME").Text = "USERNAME"
   session.findById("wnd[0]/usr/pwdRSYST-BCODE").Text = "PASSWORD"
   session.findById("wnd[0]").sendVKey 0

   ' Open SE38
   session.findById("wnd[0]/tbar[0]/okcd").Text = "/nSE38"
   session.findById("wnd[0]").sendVKey 0
   ```

2. **How would you automate a sales order creation in `VA01`?**
   ```vbs
   ' Start VA01
   session.findById("wnd[0]/tbar[0]/okcd").Text = "/nVA01"
   session.findById("wnd[0]").sendVKey 0

   ' Fill customer field
   session.findById("wnd[0]/usr/ctxtVBAK-KUNAG").Text = "1000"
   session.findById("wnd[0]").sendVKey 0

   ' Fill material and quantity
   session.findById("wnd[0]/usr/tblSAPMV45ATCTRLU_0100/ctxtMARA-MATNR[0,0]").Text = "MAT-001"
   session.findById("wnd[0]/usr/tblSAPMV45ATCTRLU_0100/txtVBDPA-KWMENG[0,1]").Text = "10"
   session.findById("wnd[0]").sendVKey 0

   ' Save
   session.findById("wnd[0]/tbar[0]/btn[11]").Press
   ```

3. **How do you handle a popup in SAP GUI Scripting?**
   ```vbs
   If session.Children.Count > 1 Then
       Set popup = session.Children(1)
       If popup.Name = "SAPLSPOP" Then  ' Standard popup
           popup.findById("btn[0]").Press  ' Click "Yes"
       End If
   End If
   ```

4. **Write a script to extract data from an ALV grid to Excel.**
   ```vbs
   Set table = session.findById("wnd[0]/usr/cntlGRID/shellcont/shell")
   Set excel = CreateObject("Excel.Application")
   Set workbook = excel.Workbooks.Add
   Set sheet = workbook.Sheets(1)

   For i = 0 To table.RowCount - 1
       For j = 0 To table.ColumnCount - 1
           sheet.Cells(i + 1, j + 1).Value = table.GetCellValue(i, j)
       Next
   Next

   workbook.SaveAs "C:\Temp\ALV_Data.xlsx"
   excel.Quit
   ```

5. **How would you automate a Fiori app?**
   - Use **Selenium WebDriver** with **Chrome/Firefox**.
   - Inspect elements with **DevTools (F12)**.
   - Example:
     ```python
     from selenium import webdriver
     driver = webdriver.Chrome()
     driver.get("https://fiori.example.com")
     driver.find_element_by_id("USERNAME_FIELD").send_keys("USERNAME")
     driver.find_element_by_id("PASSWORD_FIELD").send_keys("PASSWORD")
     driver.find_element_by_id("LOGIN_BUTTON").click()
     ```

---
## **15. SAP Scripting Tools & Libraries**
| Tool/Library | Description | Language |
|--------------|-------------|----------|
| **SAP GUI Scripting** | Official SAP GUI automation. | VBScript, Python (`win32com`) |
| **SAP TAO** | SAP’s test automation framework. | XML |
| **Selenium** | Web automation for Fiori. | Python, Java, C# |
| **UiPath** | RPA tool with SAP integration. | Drag-and-drop |
| **Blue Prism** | RPA tool with SAP VBO. | Visual Designer |
| **Automation Anywhere** | RPA with SAP packages. | Drag-and-drop |
| **PyWinAuto** | Windows GUI automation. | Python |
| **SikuliX** | Image-based automation. | Python/Java |

---
## **16. SAP Scripting Cheat Sheet: Quick Reference**
### **Common VBScript Snippets**
```vbs
' Get current session
Set session = app.Children(0).Children(0)

' Send Enter key
session.findById("wnd[0]").sendVKey 0

' Send F8 key
session.findById("wnd[0]").sendVKey 8

' Click a button by text (if ID is unknown)
For Each btn In session.findById("wnd[0]/tbar[0]").Children
    If btn.Text = "Save" Then btn.Press
Next

' Wait for a field to be ready
Do Until Not session.findById("wnd[0]/usr/ctxtFIELD", False) Is Nothing
    WScript.Sleep 500
Loop

' Close all popups
Do While session.Children.Count > 1
    session.Children(1).Close
Loop
```

### **Common Python Snippets (with `win32com`)**
```python
import win32com.client

# Connect to SAP
SapGuiAuto = win32com.client.GetObject("SAPGUI")
app = SapGuiAuto.GetScriptingEngine
session = app.Children(0).Children(0)

# Set field value
session.findById("wnd[0]/usr/ctxtMATNR").Text = "MAT-001"

# Click button
session.findById("wnd[0]/tbar[0]/btn[0]").Press()

# Get table data
table = session.findById("wnd[0]/usr/cntlGRID/shellcont/shell")
for i in range(table.RowCount):
    print(table.GetCellValue(i, "MATNR"))
```

### **Common SAP GUI Element IDs**
| Element | Example ID |
|---------|------------|
| **Login Screen** | `wnd[0]/usr/txtRSYST-BNAME` (Username) |
| **Transaction Bar** | `wnd[0]/tbar[0]/okcd` (Transaction code input) |
| **Standard Toolbar** | `wnd[0]/tbar[0]/btn[0]` (First button) |
| **ALV Grid** | `wnd[0]/usr/cntlGRID/shellcont/shell` |
| **Tab Strip** | `wnd[0]/usr/tabsTABSTRIP` |
| **Checkbox** | `wnd[0]/usr/chkFLAG` |
| **Dropdown** | `wnd[0]/usr/ctxtCOMBO` |
| **Status Bar** | `wnd[0]/sbar` |

---
## **17. SAP Fiori Selectors (CSS/XPath)**
| Element Type | Example Selector |
|---------------|-------------------|
| **Input Field** | `input[id*="input--customerId"]` |
| **Button** | `//button[contains(@id, "createButton")]` |
| **Table Row** | `//tr[@role="row"]` |
| **Tile** | `//div[@role="button" and contains(@title, "Sales Orders")]` |
| **Combo Box** | `//div[contains(@id, "combo--select")]` |
| **Date Picker** | `input[id*="date--input"]` |

---
## **18. Troubleshooting Guide**
| Issue | Cause | Solution |
|-------|-------|----------|
| **"Scripting not enabled"** | SAP GUI scripting disabled. | Enable in SAP GUI Options → Accessibility & Scripting. |
| **"Object not found"** | Incorrect element ID. | Use `F12` to verify ID or explore `session.Children`. |
| **"Session not available"** | SAP not open or wrong session. | Check `app.Children` for active sessions. |
| **"Permission denied"** | User lacks authorization. | Run SAP with admin rights or adjust user roles. |
| **"Popup blocked script"** | Unexpected popup. | Add popup handling (check `session.Children.Count`). |
| **"Script runs slowly"** | No delays between actions. | Add `WScript.Sleep 500` between steps. |
| **"Field not accepting input"** | Field is read-only. | Check SAP authorization or field properties. |
| **"BDC session failed"** | Incorrect data format. | Validate BDC table structure with `SHDB`. |

---
## **19. SAP Automation Workflow**
1. **Plan**:
   - Identify transactions and fields.
   - Use `SHDB` to record manual steps.
2. **Develop**:
   - Write scripts in VBScript/Python.
   - Use `F12` to find element IDs.
3. **Test**:
   - Run in a test environment.
   - Handle popups/errors.
4. **Deploy**:
   - Schedule via **Windows Task Scheduler** or **RPA tool**.
5. **Monitor**:
   - Log successes/failures.
   - Set up alerts for failures.

---
## **20. Resources**
- **SAP GUI Scripting Documentation**:
  - [SAP GUI Scripting API Reference](https://help.sap.com/doc/saphelp_nw70/7.0.3/en-US/48/0f7e3d48a111d189700000e8322d00/frameset.htm)
- **SAP TAO**:
  - [SAP TAO Guide](https://help.sap.com/viewer/product/TAO/latest/en-US)
- **Selenium for Fiori**:
  - [SAPUI5 Test Automation](https://sapui5.hana.ondemand.com/#/topic/6b75d83d7d844d6e907dac53d0e5d2d7)
- **UiPath SAP Integration**:
  - [UiPath SAP Activities](https://docs.uipath.com/activities/docs/sap)
- **Python Libraries**:
  - `win32com`: [PyWin32 Docs](https://pypi.org/project/pywin32/)
  - `selenium`: [Selenium Python](https://selenium-python.readthedocs.io/)

---
**Final Tip**: For interviews, emphasize **real-world use cases** (e.g., "I automated monthly financial reports in `FBL3N` using SAP GUI Scripting, reducing processing time by 80%"). Practice explaining how you’d **handle dynamic elements** or **debug scripting errors**.
