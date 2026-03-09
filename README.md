# Internal Request Automation System

This repository contains implementation artifacts for an automated
internal service request management workflow.

The system integrates:

- ArcGIS Survey123 for request intake
- Microsoft Power Automate for workflow orchestration
- Microsoft SharePoint / Excel for request storage
- Microsoft Power BI for operational analytics

This repository provides the configuration resources required to reproduce
the automation workflow and analytical metrics described in the associated
research publication.

---

## Repository Structure

internal-request-automation/
│
├── forms
│   └── survey123_xlsform.xlsx
│
├── automation
│   ├── attachment_expression.txt
│   ├── routing_expression.txt
│   └── email_templates.md
│
├── analytics
│   └── powerbi_measures.dax
│
├── sample_data
│   └── internal_requests_sample.csv
│
└── README.md

---

## Survey Configuration

The intake form is implemented using the XLSForm schema.

Example structure:

type,name,label
select_one division,department,Division
select_one requester,requester,Requester
select_one service,services,Service Type
select_multiple assets,asset,Asset Category
text,description,Description
file,upload,Attachment
select_one priority,priority,Priority Level

---

## Power Automate Expressions

### Attachment Processing

Used to determine whether a file was uploaded and generate a URL.


if(
empty(triggerOutputs()?['body/feature/attachments/upload']),
'N/A',
concat(
triggerOutputs()?['body/feature/attachments/upload']?[0]?['url'],
'?token=TOKEN'
)
)


---

### Requester Email Lookup


outputs('EmailList')?[trim(body('Get_the_requester_name')?['Requester Name'])]


---

### Assignment Notification Routing


outputs('Emails')?[trim(body('Get_a_row')?['Assignee'])]


---

## Power BI Analytical Measures

### Percentage of Requests


% of Internal Requests =
DIVIDE(
COUNT('InternalRequests'[Division]),
CALCULATE(
COUNT('InternalRequests'[Division]),
ALL('InternalRequests')
)
)


---

### Average Processing Time


AverageTimeToMapInternal =
AVERAGE(InternalRequests[TimeToMap])


---

### Monthly Requests


Avg request per Month =
DIVIDE(
COUNT('InternalRequests'[Requested Date]),
DISTINCTCOUNT('InternalRequests'[YearMonth])
)


---

### Processing Duration


TimeToMap =
DATEDIFF(
'InternalRequests'[Start Date],
'InternalRequests'[End Date],
HOUR
)


---

### Month Aggregation


YearMonth =
FORMAT('InternalRequests'[Requested Date], "YYYY-MM")


---

## Sample Dataset

The repository includes a synthetic dataset that replicates the schema used
by the system.

Location:

sample_data/internal_requests_sample.csv

This dataset enables testing of the analytics workflow without exposing
sensitive organizational data.

---

## Reproducing the Workflow

1. Deploy the XLSForm in ArcGIS Survey123.
2. Configure the Excel repository in SharePoint.
3. Import the workflow expressions into Power Automate.
4. Connect the dataset to Power BI.
5. Refresh the dashboard to generate operational metrics.

---

## License

This repository is provided for research and educational purposes.
