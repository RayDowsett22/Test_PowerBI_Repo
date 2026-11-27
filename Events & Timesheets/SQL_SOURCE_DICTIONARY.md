# SQL source tables and fields

This inventory lists SQL objects read by the semantic model and the fields that flow into the model or are referenced by its SQL queries. It includes columns used only to join, filter, or aggregate data, as well as fields selected for model tables. Field names are written as they appear at the SQL source; where Power Query renames or derives a model field, that is noted.

The `Instances` Power Query parameter selects the SQL Server instance. The database and schema names below are the model's configured sources, not credentials. Fields may be selected via `SELECT *`/Power Query navigation, so the inventory describes model/query usage and does not claim that these are every column physically present in the source object.

## BIReporting database

| SQL table/view | Fields used | Model table / usage |
|---|---|---|
| `BIReporting.BIAbout.vw_Info` | `ElementID`, `ElementName`, `ElementDescription`, `CurrentVersion`, `ISCDescription`, `GoliveDate`, `ShortName`, `LatestUpdateDescription` | `'BIAbout vw_Info'`; report about-page metadata. Power Query derives `ReportVersion` and `ISC`. |
| `BIReporting.BIAbout.vw_VersionHistory` | `ElementID`, `ElementVersionID`, `ResourceName`, `LatestUpdateDescription`, `VersionDate`, `CurrentVersion` | `'BIAbout vw_VersionHistory'`; report version-history page. |
| `BIReporting.XpedeonWriteBack.EventActions` | `KeyID`, `Source`, `DocRef`, `DocGUID`, `ResponseID`, `ActionID`, `Project`, `ActionBy`, `ActionRequired`, `ActionNotes`, `InProgressNotes`, `ActionOpened`, `ActionClosed`, `ActionCompleted`, `ActionCompletedFormat`, `Comment`, `Processed`, `ModifiedBy`, `ModifiedDate`, `ActionStatus`, `InProgressFormat`, `ActionNotesFormat`, `ActionReqFormat`, `ProcessedDate` | `XpedeonWritebackEventActions` (DirectQuery); `ActionBy` is renamed `Current Owner`, and `ResponseID` relates to event-action responses. |

## JNBDW database (`dbo` schema)

| SQL table/view | Fields used | Model table / usage |
|---|---|---|
| `JNBDW.dbo.DimActivity` | `ActivityID`, `ActivityCategoryID`, `InputCode`, `ActivityDescription`, `Active`, `InsertAuditID`, `UpdateAuditID`, `ActivityTypeID`, `Payroll`, `Timesheet`, `Payroll-All`, `Timesheet-All` | `DimActivity`; activity lookup and hour inclusion flags. `Input Description` and conditional-format columns are calculated in Power Query/DAX. |
| `JNBDW.dbo.DimActivityType` | `ActivityTypeID`, `ActivityTypeCode`, `ActivityTypeDescription` | `DimActivityType`; activity type lookup. |
| `JNBDW.dbo.DimDate` | `DateID`, `DateValue`, `MonthName`, `Year`, `MonthID`, `Prior3Months`, `Prior6Months`, `Prior12Months`, `DayName`, `isCurrentday`, `isCurrentMth`, `isMonthEnd`, `IsBankHoliday`, `IsWorkingDay`, `IsWeekEnd`, `IsHoliday`, `DayOffset`, `WeekOffset`, `MonthOffset`, `YearOffset`, `WeekNumber`, `InsertAuditID`, `UpdateAuditID`, `WeekEndDate`, `Month Year`, `Long Month Year`, `isWeekday`, `WeekOffetReverse` | `DimDate`; calendar fields. `WeekOffetReverse` is calculated; the dimension filters out rows with null `WeekEndDate`. `DateID`, `DateValue`, `MonthOffset`, and `WeekEndDate` are also used in fact-query joins and filters. |
| `JNBDW.dbo.DimDesignation` | `DesignationID`, `DesignationCode`, `DesignationDescription`, `InsertAuditID`, `UpdateAuditID`, `Support`, `Operation`, `ExpGroupID`, `Design` | `DimDesignation`; designation lookup. |
| `JNBDW.dbo.DimEntityType` | `EntityTypeID`, `EntityTypeDescription`, `EntityTypeCode` | `DimEntityType`; entity-type lookup. `ReportedByCategory` is calculated in the model. |
| `JNBDW.dbo.DimEventStatus` | `EventStatusID`, `Status`, `InsertAuditID`, `UpdateAuditID` | `DimEventStatus`; event-status lookup. |
| `JNBDW.dbo.DimEventType` | `EventTypeID`, `EventTypeDescription`, `InsertAuditID`, `UpdateAuditID`, `EventTypeDescription_new` | `DimEventType`; event-type lookup and event-query exclusion of `QES New Starter`. |
| `JNBDW.dbo.DimFormType` | `FormTypeID`, `FormType`, `InsertAuditID`, `UpdateAuditID` | `DimFormType`; form lookup. Power Query changes `External (e.g. Public)` to `External` in `FormType`. |
| `JNBDW.dbo.DimLevel1Categorisation` | `Level1CatID`, `Level 1 Categorisation`, `InsertAuditID`, `UpdateAuditID` | `DimLevel1Categorisation`; event category lookup and target matching. |
| `JNBDW.dbo.DimLevel2Categorisation` | `Level2CatID`, `Level 2 Categorisation`, `InsertAuditID`, `UpdateAuditID` | `DimLevel2Categorisation`; event category lookup. |
| `JNBDW.dbo.DimLevel3Categorisation` | `Level3CatID`, `Level 3 Categorisation`, `InsertAuditID`, `UpdateAuditID` | `DimLevel3Categorisation`; event category lookup. |
| `JNBDW.dbo.DimLevel4Categorisation` | `Level4CatID`, `Level 4 Categorisation`, `InsertAuditID`, `UpdateAuditID` | `DimLevel4Categorisation`; event category lookup. |
| `JNBDW.dbo.DimEntity` | `EntityID`, `EntityDivisionID`, `StaffID`, `EmploymentGUID`, `DeptID`, `UserName`, `ShortName`, `DesignationID`, `EntityName`, `EmploymentStartDate`, `EmploymentEndDate`, `FromDate`, `ToDate`, `PayrollFrequency`, `EntityTypeID`, `Status`, `InsertAuditID`, `UpdateAuditID`, `isCurrent`, `DOB`, `FirstName`, `LastName`, `Email`, `MMID_Code`, `ChangedEmail`, `Deleted` | `DimReportedBy`; employee/entity attributes. `EntityID` and `ShortName` are also used by the project-role query. |
| `JNBDW.dbo.DimSource` | `SourceID`, `SourceDescription` | `DimSource`; event-source lookup. |
| `JNBDW.dbo.vw_HoursHelp` | `Activity Category`, `Activity Type Code`, `Activity Type Description`, `Activity Input`, `Activity Description`, `Raw Timesheet`, `Processed Attendance`, `Payroll Flag`, `Timesheet Flag`, `Payroll All`, `Timesheet All`, `Subsistance & Lodge`, `Lodge`, `Processed Attendance Normal Hours`, `Timesheet Exceptions`, `People Rate Comparrison`, `Contract`, `ProjectStaffLabourCosts` | `HoursHelp`; report guidance/support view. Power Query converts true/false values in several support columns to 1/0. |
| `JNBDW.dbo.FactEventRegister` | `EventID`, `EventTypeID`, `FormTypeID`, `EventStatusID`, `Description`, `DOC_GUID`, `DOC_REF`, `ReportedByEntityID`, `OriginatorEntityID`, `ProjectID`, `Level1CatID`, `Level2CatID`, `Level3CatID`, `Datekey`, `VersionNumber`, `ParentDocGUID`, `ParentDocRef`, `SourceID`, `ReviewType`, `ReportedByCategory`, `InsertAuditID`, `UpdateAuditID`, `LogDeleted`, `Form_id`, `Level4CatID` | `FactEventRegister`; event rows. Joined to date and event type; filters out deleted records, `QES New Starter`, and dates outside `MonthOffset -36` through 0. |
| `JNBDW.dbo.FactEventActionResponse` | `EventActionID`, `EventID`, `Response_ID`, `Question`, `Answer`, `AnswerAction`, `SEQ_NO`, `DOC_GUID`, `Requires_Action`, `CurrentOwner`, `DateOpened`, `DateOpenedID`, `DateClosed`, `DateClosedID`, `Duration`, `Status`, `IsActionClosedinSLA`, `InsertAuditID`, `UpdateAuditID`, `LogDeleted` | `FactEventActionResponse`; event action/response detail. Also queried by the carbon summary for `EventID`, `LogDeleted`, `Question`, and `Answer`. Responses are limited to non-deleted qualifying events. SQL derives the model's `CO2Event`, `CO2AddedSaved`, and CO2 range indicator columns from response answers. |
| `JNBDW.dbo.FactTimeSheetHours` | `TimeSheetHoursID`, `SourceId`, `LineGUID`, `HDRDocGUID`, `EntityID`, `TimeSheetDateID`, `ProjectID`, `TimeSheetHours`, `TimeSheet46HourRule`, `ActivityID`, `InsertAuditID`, `UpdateAuditID`, `SectionIDValue`, `SubType`, `LogDeleted`, `BACK_PAY`, `BACK_PAY_ENTERED_IN_TS_START_DATE`, `Division`, `RateToApply`, `isCorrection` | `FactTimeSheetHours`; raw timesheet/attendance records. `TimeSheetHours` is exposed as `OriginalHours`; the query also derives `PayrollHours`, `TimesheetHours`, and rule flags. The query joins date and activity and filters deleted records and `MonthOffset -36` through 0. |
| `JNBDW.dbo.DimProjectEmployeeRole` | `ProjectID`, `RoleID`, `CurrentEmployeeID`, `IsCurrent`, `Isdeleted` | `POS`; selects current, non-deleted project-role assignments for a defined set of role IDs. |
| `JNBDW.dbo.DimRole` | `RoleID`, `Pos_name` | `POS`; role lookup used to pivot project role assignments into columns. |
| `JNBDW.dbo.DimProject` | `ProjectID`, `ProjectCode`, `ProjectName`, `ProjectTypeID`, `ContractID`, `Active`, `PostCode`, `CompanyID` | `POS`; project details and project-type join. Project name is filtered to remove names containing `Dummy`; project code contributes to `ExcludeProjects`. |
| `JNBDW.dbo.DimProjectType` | `ProjectTypeID` | `POS`; joined to project type. |
| `JNBDW.dbo.DimContract` | `ContractID`, `ContractCode`, `ContractName`, `ClientID`, `FrameWorkID` | `POS`; contract/framework/client joins and labels. |
| `JNBDW.dbo.DimClient` | `ClientID`, `ClientCode`, `ClientName` | `POS`; client join and labels. |
| `JNBDW.dbo.DimFramework` | `FrameworkID`, `Framework_Agreement_ID`, `Framework_Agreement_Name` | `POS`; framework agreement join and labels. |
| `JNBDW.dbo.DimCompany` | `CompanyID`, `CompanyName` | `POS`; company join and label. |

## ExternalData database

| SQL table | Fields used | Model table / usage |
|---|---|---|
| `ExternalData.XPEDEON.HRP_EMPLOYEE_ATTRIBUTES` | `EMPLOYMENT_ID`, `VALUE`, `EMP_ATTRIB_CODE` | `DimLocation`; selects `EMPLOYMENT_ID` as `EmploymentGUID`, `VALUE` as `Location`, filtered to `EMP_ATTRIB_CODE = 'LOC'`. |
| `ExternalData.XPEDEON.ATT_PROC_HIRE_CHARGES` | `ENTITY_DIVISION_ID`, `DOC_DATE`, `IE_CODE`, `INPUT_CODE`, `Amount` | `EmployeeProjectCosts`; `Amount` is summed into model field `COST`, grouped by division, date, project code, and input code. Joined to `JNBDW.dbo.DimDate` on document date and limited to `MonthOffset -36` through 0. |

## Non-SQL model tables

These are embedded in the semantic model rather than read from SQL:

| Model table | Contents |
|---|---|
| `FactTargetRate` | Embedded event frequency-rate targets. |
| `FactTargetPercent` | Embedded percentage targets. |
| `RAGOrder` | Embedded RAG labels and ordering. |

Auto-generated `LocalDateTable_*` and `DateTableTemplate_*` objects are Power BI date-hierarchy infrastructure, not additional SQL source tables.
