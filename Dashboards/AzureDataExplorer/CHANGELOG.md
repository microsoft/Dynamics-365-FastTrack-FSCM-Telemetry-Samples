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

- **Environment Id value list ignored the dashboard time range.** The lookup was
  hardcoded to `ago(7d) .. now()`. With a time range wider than seven days, environments
  present only in the older window were missing from the picker even though the tiles
  would have filtered on them correctly. It now uses `_startTime .. _endTime`, so all
  queries in the dashboard share the dashboard time range.

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
consistent: `array` selection with `includeAllOption`, membership (`in (...)`) filters
guarded by `isempty()`, and lookup queries bound to the dashboard time range.

> **Note:** the Azure Data Explorer query editor may report a semantic error on
> `_startTime` / `_endTime` in a parameter lookup query until that query has been run
> once from within the parameter editor. The variables do resolve correctly at runtime.
