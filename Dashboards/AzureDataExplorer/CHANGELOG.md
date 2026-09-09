# Changelog — Azure Data Explorer sample dashboards

All notable changes to the sample dashboards in this folder are documented here.

Entries are grouped by dashboard and listed newest first.

## Batch — `Batch/ADE-Dashboard-D365FO-Monitoring-Batch.json`

### Parameter filtering corrections

#### Fixed

- **Environment Id filters never matched any data.** Every tile compared
  `tostring(customDimensionsParsed.EnvironmentId)`, but the field emitted in
  `customDimensions` is `environmentId` (lowercase `e`) — the same casing the
  parameter's own lookup query reads. Selecting an environment matched nothing and
  blanked all 13 tiles. Filters now read `customDimensionsParsed.environmentId`.
  The `EnvironmentId` output column alias in the *Throttled batch tasks*
  projection and the alias in the lookup query are display names, not lookups, and
  were intentionally left unchanged.

- **Batch Infolog Errors query failed outright.** The Environment Id filter on the
  *Batch Infolog Errors* table tile referenced `cvustomDimensionsParsed.EnvironmentId`,
  an undefined identifier, so the query errored rather than simply returning no rows.

- **Batch Job Id filter blanked the tasks-spread pie.** *Batch server #tasks execution
  spread* read its job id from `customDimensionsParsed.batchJobId` (lowercase `b`),
  while the field emitted on `BatchTaskStart` is `BatchJobId` — the casing already used
  by the parameter lookup query and the other eight filtering tiles. Selecting a batch
  job emptied this tile.

- **Environment Id did not support multiple selections.** The parameter is declared
  `selectionType: array` with `includeAllOption`, but all 13 tiles filtered it with
  scalar equality (`== _environmentId`). This works for a single selection, which is why
  it appeared correct, but breaks as soon as more than one environment is picked. All
  filters now use `in (_environmentId)`, matching the membership pattern already used by
  the other parameters.

- **Activity Id parameter had no effect.** `_activityId` was declared but referenced by
  no tile, so selecting an activity did nothing anywhere on the dashboard. It is now
  applied to the *Batch execution history* tile, which already surfaced an `ActivityId`
  column, following the same `isempty()` guard convention as the other parameters:

  ```kusto
  | where isempty(_activityId) or ActivityId in (_activityId)
  ```

- **Activity Id value list came from the wrong table.** The lookup query sourced values
  from `pageViews` — client/UI telemetry that does not carry batch activity ids — so the
  picker offered values that could never match the batch tiles. It now reads batch
  `customEvents`, drops empty values, and sorts ascending.

- **Batch Job Id value list raised a semantic error.** Its lookup query referenced
  `_startTime` and `_endTime`. Those variables are declared by the Time range parameter
  but are only available to tile queries — parameter lookup queries cannot resolve them,
  so the query failed to compile. The lookup now uses a fixed `ago(7d) .. now()` window,
  matching the workaround already used by the Environment Id lookup.

#### Changed

- **Batch Job Id now supports multiple selections.** The parameter was
  `selectionType: scalar` while the other two were `array`, so only one batch job could
  be inspected at a time. It is now `array`. No query changes were needed — the nine
  filtering tiles already used `in (_batchJobId)`, which is valid for both forms. Note
  that *Batch server #tasks execution spread* and *Batch execution history* now aggregate
  across every selected job.

- **Activity Id value list covers job-level events.** The lookup previously considered
  only the task-level events `BatchTaskStart`, `BatchTaskFinished` and `BatchTaskFailure`,
  so activity ids appearing only on job-level events were absent from the picker.
  `BatchJobStart` and `BatchJobFinished` were added to the filter.

#### Result

All three parameters — Activity Id, Batch Job Id and Environment Id — are now
consistent: `array` selection with `includeAllOption`, and membership (`in (...)`)
filters guarded by `isempty()`.

> **Note:** parameter lookup queries cannot reference the Time range parameter's
> `_startTime` / `_endTime` variables — those resolve only in tile queries. All three
> lookups therefore use a fixed `ago(7d) .. now()` window to populate their value lists,
> independently of the time range selected on the dashboard.
