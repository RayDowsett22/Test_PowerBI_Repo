# Events & Timesheets Power BI

This folder contains two Power BI reports connected to a shared semantic model:

| Item | Purpose |
|---|---|
| `Event Register v3.0.0.pbip` | Event register, event actions, event frequency and closure, categorisation, and carbon impact reporting. |
| `Timesheet Register v2.0.0.pbip` | Timesheet and attendance hours, employee/project summaries, and project labour costs. |
| `Timesheets & Events Dataset.pbip` | Shared semantic model used by both reports. |

The report definitions are thin reports connected to the published **Timesheets & Events Dataset** semantic model in the **Xpedeon Registers** Power BI workspace. The checked-in model definition is under `Timesheets & Events Dataset.SemanticModel`.

For model table and field definitions, relationships, and measure notes, see [DATA_DICTIONARY.md](./DATA_DICTIONARY.md). For the underlying SQL objects and source columns, see [SQL_SOURCE_DICTIONARY.md](./SQL_SOURCE_DICTIONARY.md).

## Reports

Both reports have **About** and **Filters** pages. Report page inventory below follows the order in the checked-in page metadata; drill-through and help pages are included.

### Event Register v3.0.0

- **Event Register**, **Event Form View**, **Event Details**: event listings and event-level detail.
- **Events by Categorisation**, **Events graph**, **Event Frequency**, **Event Frequency by Project**, **Event Frequency Graph**: event counts and frequency trends and comparisons.
- **Events per 100K hrs by person graph**, **Events per 100K hrs by project graph**: person/project event-frequency comparisons against company results and targets.
- **CO2 Impact**, **CO2 Impact by Project**, **CO2 Register**: reported carbon impact, by project and event.
- **Event Closed Frequency%**, **Form Closed Frequency by Project%**, **Closed % by person graph**, **Closed % by project graph**: closure percentages and comparisons.
- **Event Actions**, **Action Details**: action status, ownership and action-level detail.
- **QES Support**, **Hours Details**, **Payroll Group Info**, **Hours Help**: supporting views and report guidance.

### Timesheet Register v2.0.0

- **Timesheet Summary**, **Hours Info**, **All Hours Info**: timesheet and attendance summaries and detail.
- **Project Summary**, **Project Summary All**: project cost summaries.
- **Timesheet**, **Timesheets All**, **Individual Time**: employee/project timesheet drill-through and detail.
- **Processed Timesheets**, **Timesheets**: supporting processing views.
- **Hours Help**: report guidance.

The **Timesheet** page is configured as a hidden drill-through destination, accepting entity type, project, and week-ending date. Other page visibility and drill-through behavior is controlled by the report definition.

## Data and calculation behavior

- The semantic model uses Import partitions for its warehouse tables and a DirectQuery partition for `XpedeonWritebackEventActions`.
- The Power Query parameter `Instances` selects the SQL Server instance. Its checked-in default is `jnbsql03.jnbentley.co.uk`; the alternative is `jnbskvrepo001.jnbentley.co.uk`.
- Sources include the `JNBDW` and `BIReporting` databases on the selected instance, and the `ExternalData` database for hire charges. Target-rate and target-percent tables are embedded in the model.
- Event, event-action, and timesheet fact queries use `DimDate.MonthOffset BETWEEN -36 AND 0` (current month and the preceding 36 months). Event and timesheet queries also exclude deleted records. Event register queries exclude the `QES New Starter` event type. Carbon summary rows are further limited to `QC&Event` records with a confirmed carbon impact.
- Both reports contain a hidden report-level project filter for `POS.ExcludeProjects = 0`. The project query sets that flag for project codes beginning `PRA` or `ZZ`; it also removes project names containing `Dummy`.
- The `Hours` and `Hours (ALL)` measures select their values from activity flags. Payroll and timesheet adjustments are calculated in the `FactTimeSheetHours` query using the documented 8.5-hour and 46-hour rules and activity-specific exclusions. See the dictionary for details.
- Frequency-rate target and RAG measures are designed for monthly comparisons. The DAX comments caution against using them for weekly, quarterly, or yearly comparisons and assume target periods do not overlap.
- Estimated external-hour costs use a hard-coded rate of 130 in the model. Confirm this rate with the business owner before using it as a current costing assumption.

## Working with this folder

Open the relevant `.pbip` file in Power BI Desktop. Both report definitions point to the published semantic model; editing or refreshing against the SQL sources requires appropriate Power BI and database access. Changing the `Instances` parameter changes the server used by model queries.

The TMDL and report JSON files are the source-controlled definitions. Power BI-generated local date tables and the local model cache are implementation artifacts and are not documented as business tables. After changing tables, relationships, report pages, filters, or measures, update [DATA_DICTIONARY.md](./DATA_DICTIONARY.md) and this overview as applicable.

## Documentation scope

This documentation is based on the checked-in PBIP, TMDL, and report JSON definitions. It describes configured queries and calculations, not live row counts, refresh status, access permissions, or business definitions that are not present in the model. Column type names in the dictionary are Power BI model types (`int64`, `double`, `string`, `boolean`, and `dateTime`).
