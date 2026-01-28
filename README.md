# Designed to work with Assets and an Event Trigger configured in UiPath Orchestrator

Orchestrator and Email activities require *manual binding* in **Studio** (for example selecting the SubjectFilter Asset when assigning configDict("SubjectFilter"), etc.).

Works with Gmail or Outlook, depending on which Integration Service connector is used.

## High-level workflow

  1. ### Main.xaml
     - Detects which email connector triggered the job (Gmail / Outlook).
  2. ### InitConfig.xaml
     - Fetches configuration Assets from Orchestrator.
     - Overrides values from the local Config Excel file. (The Config Excel acts as a *baseline*, Assets take priority.)
     - Returns a Dictionary<String, String> used as config across the workflow.
  3. ### ProcessEmail.xaml
     - Saves PDF attachments based on the configured attachment keyword.
     - Fetches the **ExtractionRules** Asset.
     - Returns the list of PDF files and extraction rules.
  4. ### ProcessFiles.xaml
     - Loops through each PDF file.
     - For each file, loops through the Extraction Rules.
     - Extracts values from the full PDF text using regEx.
     - Collects extracted values into a DataTable.
     - Writes the final results into an Excel file at the configured output path.
  5. ### Notification.xaml
      - Sends a notification to the address configured for "NotificationChannel" Asset.

## Assets – setup requirements
  - ### All 5 **Assets** must be of type Text.
  - ### Recommended Asset names:
    - SubjectFilter
    - AttachmentKeyword
    - OutputExcelPath
    - NotificationChannel
    - ExtractionRules
  - ### Example configuration scenario
    - **Goal**: Trigger the process when an email is received with "Invoices" in the subject. Process PDF attachments that contain "Invoice" in the filename. Extract Invoice Number, Invoice Date, and Total Amount from each PDF. Save the results into an Excel file locally.
      - **Asset** values
        - SubjectFilter → Invoices
        - AttachmentKeyword → Invoice
        - OutputExcelPath → C:\exports\Invoices.xlsx
        - NotificationChannel → john@doe.com
        - ExtractionRules → InvoiceDate||Date||DATE; InvoiceNumber||Invoice No||TEXT; TotalAmount||Total||CURRENCY
  - ### ExtractionRules format
    - Each rule follows this structure:
      - DictionaryKey || AnchorText || Type
    - Example (DATE rule):
      - InvoiceDate || Date || DATE
    - Explanation:
      - DictionaryKey: Used internally and becomes the DataTable column name / Excel header.
      - AnchorText: The label text found in the PDF (used to locate nearby values).
      - Type: Determines which regEx strategy is used.
    - Supported types
      - **DATE** – date-formatted values
      - **TEXT** – characters and numbers
      - **CURRENCY** – numeric values followed by a 3-letter currency code (e.g. EUR, USD)

### You can test the workflow locally by: 
  - Creating the Assets and an Event Trigger in Orchestrator.
  - Sending a test email you want to work with that would normally trigger the event.
  - Copy the "UiPathEventObjectId" and "UiPathEventConnector" from the logs of the job.
  - Paste the Id into the relevant "Get Email by ID" activity in the first switch-case of the Main.xaml, and assign the Connector to the "UiPathEventConnector" variable.
  - You can now run the workflow locally without relying on external trigger.
