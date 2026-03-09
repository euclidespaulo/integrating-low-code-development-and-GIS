# Internal Request Automation System

The system integrates **ArcGIS Survey123, Power Automate, SharePoint, and Power BI** to manage internal GIS service requests. Requests submitted through Survey123 trigger an automated workflow that records the request, routes notifications to responsible personnel, and updates monitoring dashboards.

---

# Repository Structure

```
/survey123
  - xlsform_template.xlsx
/power_automate
  - expressions.md
  - flow_logic.md
/power_bi
  - dax_measures.md
/docs
  - architecture_diagram.png
  - workflow_overview.png
```

This structure separates configuration files, automation logic, analytical models, and system documentation to support reproducibility and maintainability.

---

# Reproduction Steps

To reproduce the automated request management workflow described in the research:

1. Configure **Survey123** using the provided XLSForm template.
2. Publish the survey to **ArcGIS Online**.
3. Configure the **Power Automate** workflow to trigger on survey submission.
4. Connect the workflow to a **SharePoint or Excel repository** for storing requests.
5. Configure **Power BI** using the provided DAX measures for dashboard analytics.

These steps recreate the intake, automation, storage, and analytics pipeline described in the study.

---

# Survey123 Configuration

The intake form is implemented using the **XLSForm schema**.

## Survey Sheet

| type | name | label | required | choice_filter | notes |
|-----|-----|-----|-----|-----|-----|
| begin_group | begin | Fill out the form | no |  | Form section start |
| select_one division | department | Division | yes |  | Determines requester options |
| select_one requester | requester | Requester | yes | Division=${department} | Displays requesters belonging to the selected division |
| hidden | email | Email | no |  | Automatically populated field |
| select_one service | services | Services | yes |  | Determines available asset categories |
| select_multiple assets | asset | Assets | yes | regex(Assets,${services}) | Displays assets associated with the selected service |
| text | description | Description | yes |  | Required request description |
| file | upload | Upload file | no |  | Optional supporting attachment |
| select_one priority | priority | Priority Level | yes |  | Indicates request urgency |
| date | reportdatetime | Date and time | no |  | Submission timestamp |
| end_group | end |  | no |  | Form section end |

---

## Choices Sheet

| list_name | name | label | division | requester |
|-----------|------|-------|----------|-----------|
| division | Constructionservice | Construction Service |  |  |
| division | engineering | Engineering |  |  |
| division | governmental | Governmental |  |  |
| division | IFRWWR | IFR WWR |  |  |
| division | it | Information Technology |  |  |
| division | tnd | Transmission & Distribution |  |  |
| division | waterquality | Water Quality |  |  |
| requester | requester_1 | Requester 1 | Governmental | requester_1 |
| requester | requester_2 | Requester 2 | Engineering | requester_2 |

---

# Power Automate Expressions

## Attachment Processing

Used to determine whether a file was uploaded and generate a URL.

```
if(
empty(triggerOutputs()?['body/feature/attachments/upload']),
'N/A',
concat(
triggerOutputs()?['body/feature/attachments/upload']?[0]?['url'],
'?token=TOKEN'
)
)
```

---

## Requester Email Lookup

```
outputs('EmailList')?[trim(body('Get_the_requester_name')?['Requester Name'])]
```

`EmailList` compose stores JSON in the format:

```
{"requestername":"email"}
```

The requester name must match the Survey123 value to send the notification correctly.

---

## Assignment Notification Routing

```
outputs('Emails')?[trim(body('Get_a_row')?['Assignee'])]
```

---

# Power BI Analytical Measures

## Percentage of Requests

```
% of Internal Requests =
DIVIDE(
COUNT('InternalRequests'[Division]),
CALCULATE(
COUNT('InternalRequests'[Division]),
ALL('InternalRequests')
)
)
```

---

## Average Processing Time

```
AverageTimeToMapInternal =
AVERAGE(InternalRequests[TimeToMap])
```

---

## Monthly Requests

```
Avg request per Month =
DIVIDE(
COUNT('InternalRequests'[Requested Date]),
DISTINCTCOUNT('InternalRequests'[YearMonth])
)
```

---

## Processing Duration

```
TimeToMap =
DATEDIFF(
'InternalRequests'[Start Date],
'InternalRequests'[End Date],
HOUR
)
```

---

## Month Aggregation

```
YearMonth =
FORMAT('InternalRequests'[Requested Date], "YYYY-MM")
```

---

# Security Note

Any authentication token shown in repository examples is **illustrative only**.

Real authentication tokens used by **ArcGIS Online and Power Automate are temporary and dynamically generated** during workflow execution. These tokens should **never be stored in public repositories, configuration files, or documentation**.

All example tokens in this repository are placeholders and do not provide system access.

---

# License

This repository is provided for **research and educational purposes**.
