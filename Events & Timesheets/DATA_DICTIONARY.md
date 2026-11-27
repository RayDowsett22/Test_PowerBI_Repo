# Data dictionary

This dictionary describes the shared **Timesheets & Events Dataset** semantic model. The checked-in model is under `Timesheets & Events Dataset.SemanticModel/definition`. Column names and Power BI data types below reflect the TMDL definition; descriptions are derived from query logic, relationships, and measure expressions. Where the source does not establish a business definition, the description is intentionally limited to what can be inferred from the model.

## Model map

The model combines the following subject areas:

- **Events:** `FactEventRegister`, with event type, status, form, project, reported-by, date, source, and categorisation dimensions.
- **Event actions and carbon:** `FactEventActionResponse`, linked to events; `FactEventActionResponseCarbonSummary`, a carbon-focused event summary.
- **Timesheets:** `FactTimeSheetHours`, linked to project, employee, date, and activity dimensions.
- **Project costs:** `EmployeeProjectCosts`, sourced from Xpedeon hire charges, linked to date, project code, employee division, and input code.
- **Targets and support:** embedded target tables, report metadata, hours help, and writeback action data.

The model uses `DimDate` for event, timesheet, and cost-date relationships. Additional auto-generated `LocalDateTable_*` tables exist for date hierarchies and are omitted below because they are Power BI implementation tables, not business-facing dimensions.

## Relationships

The listed relationships are defined in `relationships.tmdl`; keys are model join columns, not guarantees of uniqueness unless noted.

| From | To | Purpose |
|---|---|---|
| `FactEventRegister.EventTypeID` | `DimEventType.EventTypeID` | Event type |
| `FactEventRegister.EventStatusID` | `DimEventStatus.EventStatusID` | Event status |
| `FactEventRegister.FormTypeID` | `DimFormType.FormTypeID` | Form type |
| `FactEventRegister.Datekey` | `DimDate.DateID` | Event date |
| `FactEventRegister.ProjectID` | `POS.ProjectID` | Project |
| `FactEventRegister.ReportedByEntityID` | `DimReportedBy.EntityID` | Reporter |
| `FactEventRegister.SourceID` | `DimSource.SourceID` | Event source |
| `FactEventRegister.Level1CatID`–`Level4CatID` | Matching `DimLevel*Categorisation` ID | Event categorisation levels |
| `FactEventActionResponse.EventID` | `FactEventRegister.EventID` | Event action/response belongs to event |
| `FactEventActionResponse.Answer` | `RAGOrder.RAG` | Response/RAG ordering |
| `FactEventActionResponse.DateOpenedID` | `DimDate.DateID` | Inactive date relationship, activated by selected measures |
| `FactTimeSheetHours.TimeSheetDateID` | `DimDate.DateID` | Timesheet date |
| `FactTimeSheetHours.ProjectID` | `POS.ProjectID` | Project |
| `FactTimeSheetHours.EntityID` | `DimReportedBy.EntityID` | Employee/entity |
| `FactTimeSheetHours.ActivityID` | `DimActivity.ActivityID` | Activity/input |
| `DimActivity.ActivityTypeID` | `DimActivityType.ActivityTypeID` | Activity type |
| `DimReportedBy.DesignationID` | `DimDesignation.DesignationID` | Designation |
| `DimReportedBy.EntityTypeID` | `DimEntityType.EntityTypeID` | Entity type |
| `EmployeeProjectCosts.DOC_DATE` | `DimDate.DateValue` | Cost document date |
| `EmployeeProjectCosts.IE_CODE` | `POS.Project Code` | Cost project code |
| `EmployeeProjectCosts.ENTITY_DIVISION_ID` | `DimReportedBy.EntityDivisionID` | Cost employee division |
| `EmployeeProjectCosts.INPUT_CODE` | `DimActivity.InputCode` | Cost input code |
| `FactEventActionResponseCarbonSummary.EventID` | `FactEventRegister.EventID` | Carbon summary per event |
| `XpedeonWritebackEventActions.ResponseID` | `FactEventActionResponse.Response_ID` | Writeback action linked to response |

The model also relates employee/location information by `EmploymentGUID`. Date relationships to additional date columns (such as employment dates and action dates) use generated local date tables. `FactEventActionResponse.DateOpenedID` to `DimDate.DateID` is explicitly inactive; the `Number of Actions Closed` measures activate it to report by opening date. Some relationships have bidirectional filtering; consult the model definition before changing filter behavior.

## Measures

Measures are listed by their home table. “Company” measures remove selected person/project filters using specific model fields; they are not universal totals for every possible visual grouping.

### `FactTimeSheetHours`

| Measure | Definition |
|---|---|
| `Hours` | Sums `TimesheetHours` when the selected activity is flagged `Timesheet`; otherwise sums `PayrollHours` when flagged `Payroll`. |
| `Hours (ALL)` | As above, using `OriginalHours` for activity flags `Timesheet-All` or `Payroll-All`. |
| `Hours - Company` | `Hours` with project, reporter, or unpivoted-person filters removed when those specific fields are filtered. |
| `Hours (All Designation)` | `Hours` with designation description removed. |
| `Hours %` | `Hours` divided by `Hours (All Designation)`. |
| `Hours (ALL) (All Designation)` | `Hours (ALL)` with designation description removed. |
| `Hours (ALL) %` | `Hours (ALL)` divided by `Hours (ALL) (All Designation)`. |
| `External Hours` / `External Hours (ALL)` | Corresponding hours measure filtered to designation `Undefined` and entity categories other than `Other`. |
| `External Hours Est Cost` / `External Hours Est Cost (ALL)` | External hours multiplied by the fixed rate of 130. |
| `Hours + 0` / `Hours (ALL) + 0` | Corresponding measure with blank replaced by zero. |

### `FactEventRegister`

| Measure | Definition |
|---|---|
| `Number of Events` | Distinct count of `DOC_GUID`. |
| `Number of Events + 0` | Event count with blank replaced by zero. |
| `Number of Events - Closed`, `- Open`, `- Completed` | Event count filtered to the matching status in `DimEventStatus[Status]`. |
| `Number of Events - Closed %` | Closed event count divided by event count. |
| `Number of Events - Company` | Event count with the selected project, reporter, or unpivoted-person dimension removed, depending on the filtered field. |
| `Number of Events - Closed - Company` | Closed count with the same specific company-total filter behavior. |
| `Number of Events - Closed - Company %` | Company closed count divided by company event count. |
| `Frequency Rate` | Events per 100,000 `Hours`. |
| `Frequency Rate - Company` | Company event count per 100,000 company hours. |
| `% Categorisation` | Event count divided by the selected categorisation/form group total; returns blank at the group total level. |
| `Title for … graph` | Dynamic titles for event frequency, closure, and CO2 visuals. |

### `FactEventActionResponse`

| Measure | Definition |
|---|---|
| `Number of Event Actions` | Counts `RESPONSE_ID` where `REQUIRES_ACTION = "Y"`; blank count is returned as zero. |
| `Number of Event Actions + 0` | Action count with blank replaced by zero. |
| `Number of Site Support Reponses` | Distinct count of `Response_ID` (spelling as in the model). |
| `Total Number of Site Support Reponses` | Counts responses across nonblank RAG values. |
| `% of Site Support Reponses` | Response count divided by the above total. |
| `Number of Actions Closed` | Distinct response count using `DateOpenedID` as the active date relationship for the measure. Despite its name, the expression has no explicit closed-status filter. |
| `Number of Actions Closed - SLA` | Sum of `IsActionClosedInSLA` using the `DateOpenedID` relationship. |

### `FactEventActionResponseCarbonSummary`

| Measure | Definition |
|---|---|
| `Exact CO2 Added` / `Exact CO2 Saved` | Sums `ExactCO2` where `CO2Factor` is respectively `1` / `-1`. |
| `Exact CO2 Impact` | Exact CO2 added minus exact CO2 saved. |
| `0-1`, `1-10`, `>10 Tonnes CO2e Added/Saved` | Counts event responses in each range split by the sign of `CO2Factor`. |
| `CO2Added` / `CO2Saved` | Totals the three corresponding reported CO2e ranges. |

### `FactTargetRate`

| Measure | Definition |
|---|---|
| `Matching Targets Frequency Rate` | Counts target rows matching the selected event type, entity/hours type, categorisation level, and the full date context. Wildcard target values are accepted. |
| `Frequency Rate - Red/Amber/Green Target` | Returns the matching row's red maximum, amber maximum, or green minimum. |
| `Frequency Rate - RAG` / `Frequency Rate - RAG (Project)` | Returns a hex colour based on actual frequency versus target thresholds. The project version suppresses the red threshold for `QC&E TA`. |

### `EmployeeProjectCosts`

| Measure | Definition |
|---|---|
| `Employee Cost` | Sum of `COST`, formatted as GBP. |
| `Employee Cost inc External` | Employee cost plus estimated external hours cost when designation is `Undefined`. |
| `Employee Cost inc External (ALL)` | Same pattern using `External Hours Est Cost (ALL)`. |

## Table and column inventory

`int64` is a whole number, `double` a decimal number, `string` text, `boolean` true/false, and `dateTime` a date/time value. Columns listed under “Calculated” are model- or Power Query-derived. IDs and audit columns are technical fields unless their use is described otherwise.

### Report metadata and reference tables

#### `BIAbout vw_Info` — report about-page metadata

| Column | Type | Meaning |
|---|---|---|
| `ElementID` | `int64` | Report element identifier; the report definition filters this table to element 3. |
| `ElementName` | `string` | Display name. |
| `ElementDescription` | `string` | Report description. |
| `CurrentVersion` | `string` | Current report version. |
| `ISCDescription` | `string` | Information security classification description. |
| `GoliveDate` | `dateTime` | Go-live date. |
| `ShortName` | `string` | Short display name. |
| `LatestUpdateDescription` | `string` | Latest update description. |
| `ReportVersion` | `string` | Calculated label prefixed with `Report - `. |
| `ISC` | `string` | Calculated classification label prefixed with `Information Security Classification:`. |

#### `BIAbout vw_VersionHistory` — report release history

`ElementID` (`int64`), `ElementVersionID` (`int64`): identifiers; `ResourceName` (`string`): resource name; `LatestUpdateDescription` (`string`): release notes; `VersionDate` (`dateTime`): update date; `CurrentVersion` (`string`): version label.

#### `DimDate` — reporting calendar

`DateID` (`int64`): date key; `DateValue` (`dateTime`): calendar date; `MonthName` (`string`), `Year` (`int64`), `MonthID` (`int64`), `DayName` (`string`), `WeekNumber` (`int64`): calendar labels; `WeekEndDate` (`dateTime`): week-ending date; `Month Year` and `Long Month Year` (`string`): month labels. `Prior3Months`, `Prior6Months`, `Prior12Months`, `isCurrentday`, `isCurrentMth`, `isMonthEnd`, `IsBankHoliday`, `IsWorkingDay`, `IsWeekEnd`, `IsHoliday`, `DayOffset`, `WeekOffset`, `MonthOffset`, `YearOffset`, `isWeekday` (`int64`): calendar flags/offsets from the warehouse. `WeekOffetReverse` (`int64`): calculated negative week offset used for sorting. `InsertAuditID` and `UpdateAuditID` (`int64`): audit identifiers.

#### `DimActivity` — timesheet activity and hour inclusion flags

`ActivityID` (`int64`): activity key; `ActivityCategoryID` (`int64`): activity category key; `InputCode` (`string`): input code; `ActivityDescription` (`string`): source description; `Input Description` (`string`): display description, with ActivityID 1088 labelled `Site Visitor Hours`; `Active` (`string`): activity active value; `ActivityTypeID` (`int64`): activity type key. `Payroll`, `Timesheet`, `Payroll-All`, `Timesheet-All` (`boolean`): hour selection flags used by the hour measures. `ConditionalFormatAll`, `ConditionalFormatPayroll`, `ConditionalFormatTimesheet` (`string`, calculated): color values for report formatting. `InsertAuditID`, `UpdateAuditID` (`int64`): audit identifiers.

#### `DimActivityType` — activity type lookup

`ActivityTypeID` (`int64`): key; `ActivityTypeCode` and `ActivityTypeDescription` (`string`): code and label.

#### `DimDesignation` — employee designation lookup

`DesignationID` (`int64`): key; `DesignationCode`, `DesignationDescription` (`string`): code and label; `Support`, `Operation`, `Design` (`boolean`): designation flags; `ExpGroupID` (`int64`): expense group key; `InsertAuditID`, `UpdateAuditID` (`int64`): audit identifiers.

#### `DimEntityType` — entity category lookup

`EntityTypeID` (`int64`): key; `EntityTypeDescription`, `EntityTypeCode` (`string`): label and code. `ReportedByCategory` (`string`, calculated): maps Sub-Contractor, Agency, and Plant to themselves; all other entity descriptions to `Other`.

#### Event classifications and lookups

| Table | Columns |
|---|---|
| `DimEventStatus` | `EventStatusID` (`int64`), `Status` (`string`), `InsertAuditID` / `UpdateAuditID` (`int64`). |
| `DimEventType` | `EventTypeID` (`int64`), `EventTypeDescription` / `EventTypeDescription_new` (`string`), `InsertAuditID` / `UpdateAuditID` (`int64`). |
| `DimFormType` | `FormTypeID` (`int64`), `FormType` (`string`), `InsertAuditID` / `UpdateAuditID` (`int64`). |
| `DimLevel1Categorisation` | `Level1CatID` (`int64`), `Level 1 Categorisation` (`string`), `InsertAuditID` / `UpdateAuditID` (`int64`). |
| `DimLevel2Categorisation` | `Level2CatID` (`int64`), `Level 2 Categorisation` (`string`), `InsertAuditID` / `UpdateAuditID` (`int64`). |
| `DimLevel3Categorisation` | `Level3CatID` (`int64`), `Level 3 Categorisation` (`string`), `InsertAuditID` / `UpdateAuditID` (`int64`). |
| `DimLevel4Categorisation` | `Level4CatID` (`int64`), `Level 4 Categorisation` (`string`), `InsertAuditID` / `UpdateAuditID` (`int64`). |
| `DimSource` | `SourceID` (`int64`): key; `SourceDescription` (`string`): source label. |
| `RAGOrder` | `Order` (`int64`): ordering value; `RAG` (`string`): response/RAG label. |
| `DimLocation` | `EmploymentGUID` (`double`): employment identifier; `Location` (`string`): location label. |

#### `DimReportedBy` — employee/entity attributes

`EntityID`, `EntityDivisionID`, `EmploymentGUID`, `DeptID`, `DesignationID`, `EntityTypeID`, `InsertAuditID`, `UpdateAuditID`, `Deleted` (`int64`): entity, employment, department, designation, entity type, and audit/deletion identifiers. `StaffID`, `UserName`, `ShortName`, `EntityName`, `PayrollFrequency`, `Status`, `FirstName`, `LastName`, `Email`, `MMID_Code`, `ChangedEmail` (`string`): employee/account details and status. `EmploymentStartDate`, `EmploymentEndDate`, `FromDate`, `ToDate`, `DOB` (`dateTime`): employment/effective and birth dates. `isCurrent` (`boolean`): current-record flag.

### Project and helper tables

#### `POS` — project, client, framework, and project-role attributes

`ProjectID` (`int64`): project key; `Project Code` (`string`): project code; `Project` (`string`): project name; `Project Full` (`string`): combined project code and name; `ProjectTypeID` (`int64`): project type key. `Framework Contract Code`, `FrameworkContract`, `Framework Contract Full`, `Client Code`, `ClientName`, `FrameworkAgreementName`, `Active`, `Post Code`, `Company`, `Director`, `OperationsManager`, `ContractsManager`, `CommercialManager`, `QuantitySurveyor`, `SiteAgent`, `DesignManager`, `Project Leader`, `MEP Engineer`, `M&E Design Engineer`, `Unit` (`string`): project, framework, client, status, location, company, and assigned project-role labels. `Framework Agreement ID` (`int64`): framework agreement key. `ExcludeProjects` (`string`): query-derived value set to 1 for project codes starting `PRA` or `ZZ`, otherwise 0; report filters select 0. Query logic excludes project names containing `Dummy`.

#### `'POS - Unpivot'` — project role/person bridge

`ProjectID` (`int64`): project key; `Role` (`string`): project role; `Value` (`string`): person assigned to that role. The model relationship to `POS` uses bidirectional filtering.

#### `HoursHelp` — report support matrix

`Activity Category`, `Activity Type Code`, `Activity Type Description`, `Activity Input`, `Activity Description`, `Payroll Flag`, `Timesheet Flag`, `Payroll All`, `Timesheet All` (`string`): activity and inclusion labels. `Raw Timesheet`, `Processed Attendance`, `Subsistance & Lodge`, `Lodge`, `Processed Attendance Normal Hours`, `Timesheet Exceptions` (`double`): numeric support values. `People Rate Comparrison`, `Contract`, `ProjectStaffLabourCosts` (`int64`): support flags/values as represented in the source view. The source query converts true/false values for multiple numeric-looking columns into 1/0.

### Fact tables

#### `FactEventRegister` — event register records

`EventID`, `EventTypeID`, `FormTypeID`, `EventStatusID`, `ReportedByEntityID`, `OriginatorEntityID`, `ProjectID`, `Level1CatID`, `Level2CatID`, `Level3CatID`, `Level4CatID`, `Datekey`, `SourceID`, `InsertAuditID`, `UpdateAuditID`, `Form_id` (`int64`): event identifiers and dimension/audit keys. `Description`, `DOC_REF`, `ParentDocRef`, `ReviewType`, `ReportedByCategory` (`string`): event description, document references, review and reporter category. `DOC_GUID`, `ParentDocGUID`, `VersionNumber` (`double`): document GUID/version values. `LogDeleted` (`boolean`): deletion flag.

The query keeps non-deleted events from `MonthOffset -36` through 0 and excludes event type `QES New Starter`. The principal event count measure is a distinct count of `DOC_GUID`, not a raw row count.

#### `FactTimeSheetHours` — timesheet/attendance detail

`TimeSheetHoursID`, `SourceId`, `EntityID`, `TimeSheetDateID`, `ProjectID`, `ActivityID`, `InsertAuditID`, `UpdateAuditID`, `'8.5Rule'`, `46HourRule` (`int64`): row, source, employee, date, project, activity, audit, and rule identifiers/flags. `LineGUID`, `HDRDocGUID`, `SectionIDValue`, `OriginalHours`, `TimeSheet46HourRule`, `RateToApply`, `PayrollHours`, `TimesheetHours` (`double`): source GUIDs, section value, original/rule hours, rate and adjusted hours. `SubType`, `BACK_PAY`, `Division` (`string`): source category/attributes. `LogDeleted`, `isCorrection`, `Payroll`, `Timesheet`, `Timesheet-All`, `Payroll-All` (`boolean`): deletion, correction, and activity inclusion flags. `BACK_PAY_ENTERED_IN_TS_START_DATE` (`dateTime`): date associated with the back-pay entry.

The query drops deleted rows and retains the current and preceding 36 months. For activity IDs 1017/1018 it calculates whether weekly hours are at most 8.5; for activity 1061 it calculates whether weekly hours are at most 46. When a rule is not met, `PayrollHours` is zeroed for activity IDs 1021/1037, and `TimesheetHours` is zeroed for activity IDs 1070/1063. These rules and IDs are implemented in the source query and should be confirmed with the business owner before being changed.

#### `EmployeeProjectCosts` — Xpedeon hire-charge costs

`ENTITY_DIVISION_ID` (`double`): employee division identifier; `DOC_DATE` (`dateTime`): cost date; `IE_CODE` (`string`): project code; `INPUT_CODE` (`string`): input/activity code; `COST` (`double`): aggregated hire-charge amount.

The query groups `ATT_PROC_HIRE_CHARGES` by division, document date, IE code, and input code, limited to current and prior 36 months.

#### `FactEventActionResponse` — event action/question responses

`EventActionID`, `EventID`, `DateOpenedID`, `DateClosedID`, `Duration`, `IsActionClosedinSLA`, `InsertAuditID`, `UpdateAuditID`, `CO2Event`, `CO2AddedSaved`, `CO2_0_1`, `CO2_1_10`, `CO2_10` (`int64`): action/event/date keys, duration, SLA and derived carbon flags/values. `Response_ID`, `SEQ_NO`, `DOC_GUID` (`double`): response sequence/document identifiers. `Question`, `Answer`, `AnswerAction`, `Requires_Action`, `CurrentOwner`, `Status` (`string`): response content, action requirement, owner, and status. `DateOpened`, `DateClosed` (`dateTime`): action dates. `LogDeleted` (`boolean`): deletion flag.

The query limits responses to non-deleted events that are themselves non-deleted, are not `QES New Starter`, and are within the same 36-month window as the event register.

#### `FactEventActionResponseCarbonSummary` — carbon summary by event and added/saved response

`EventID`, `CO2Event`, `CO2Factor`, `CO2_0_1`, `CO2_1_10`, `CO2_10` (`int64`): event key, carbon flags/sign, and range indicators. `AddedSaved` (`string`): Added/Saved category. `ExactCO2` (`double`): exact reported CO2e amount.

The query includes non-deleted `QC&Event` events in the 36-month window, requires a `Carbon Impact = Yes` response, and groups data by event and added/saved response. Range indicators correspond to the answer bands `0-1`, `1-10`, and `>10 Tonnes of CO2e`.

#### `FactTargetRate` — event frequency-rate targets

`Event Type`, `Target type`, `HoursType`, `Level`, `Level 1 Categorisation`, `Level 2 Categorisation`, `Level 3 Categorisation`, `Documentation` (`string`): target matching attributes and notes. `Start date`, `End date` (`dateTime`): target validity range. `Green min`, `Amber max`, `Red max` (`int64`): rate thresholds. `FactTargetRateSK`, `EventTypeID`, `Level1CatID` (`int64`): model-generated key and lookup IDs.

This table is embedded in the model and transformed to add its surrogate key and lookup IDs. Target measures require the selected date context to fall entirely within a target's validity range. Model comments assume validated non-overlapping targets and warn that RAG/target measures are for monthly actuals only.

#### `FactTargetPercent` — percentage targets

`Event Type`, `Target type`, `Level`, `Level 1 Categorisation`, `Level 2 Categorisation`, `Level 3 Categorisation`, `Documentation` (`string`): target attributes and notes. `Start date`, `End date` (`dateTime`): validity range. `Green min`, `Amber max`, `Red max` (`double`): percentage thresholds. `FactTargetPercentSK`, `EventTypeID`, `Level1CatID` (`int64`): model-generated key and lookup IDs.

This table is embedded in the model and percentage thresholds are loaded with Power Query's `Percentage.Type`.

#### `XpedeonWritebackEventActions` — writeback actions (DirectQuery)

`KeyID` (`int64`): row key; `Source`, `DocRef`, `Project`, `Current Owner`, `ActionRequired`, `ActionNotes`, `InProgressNotes`, `ActionCompleted`, `Comment`, `Processed`, `ModifiedBy`, `ActionStatus` (`string`): source, document, project, action and processing attributes. `DocGUID`, `ResponseID`, `ActionID` (`double`): linked document, response, and action identifiers. `ActionOpened`, `ActionClosed`, `ModifiedDate`, `ProcessedDate` (`dateTime`): action and processing dates. `ActionCompletedFormat`, `InProgressFormat`, `ActionNotesFormat`, `ActionReqFormat` (`int64`): source formatting/status fields.

This table reads `BIReporting.XpedeonWriteBack.EventActions` through DirectQuery; `ActionBy` is renamed to `Current Owner`. It is linked to `FactEventActionResponse` by `ResponseID`.

## Model parameters and sources

| Parameter/source | Configuration in model |
|---|---|
| `Instances` | Text parameter. Default `jnbsql03.jnbentley.co.uk`; alternative `jnbskvrepo001.jnbentley.co.uk`. |
| `JNBDW` | Main warehouse for dates, timesheets, event register/actions, people, activities, projects, and dimensions. |
| `BIReporting` | Report metadata views and DirectQuery writeback actions. |
| `ExternalData` | Xpedeon `ATT_PROC_HIRE_CHARGES`, used to populate project costs. |
| Embedded tables | `FactTargetRate` and `FactTargetPercent` are model-embedded rather than read from the listed SQL databases. |

## Caveats

- The dictionary does not establish source-side uniqueness, nullability, row-level security, or refresh credentials unless encoded in the model definition.
- Several model columns are audit IDs or source GUIDs and should not be treated as measures. Use explicit measures for counts and totals.
- `FactTargetRate` and `FactTargetPercent` include manually embedded target data; update and validate these tables as part of target maintenance.
- Field spellings such as `WeekOffetReverse`, `Subsistance & Lodge`, `Comparrison`, and `Reponses` reproduce the model and should not be silently corrected in report references.
