# Statistics

Read this theme before answering with OEE, pace, stop-reason totals, or production history.

## Current production OEE

With no range, `get_oee` returns Current production OEE. Current production OEE is the Account-local calendar day from midnight through now — not a live snapshot. It is not the current work shift. `get_oee` does not take a work shift. Current production OEE has no time-point series. The series appears only when `start` is set. Optional ISO-8601 `start` switches to production OEE in that time range. Optional ISO-8601 `end` requires `start` and is omitted so the API can default it when only `start` is set. An `end` later than the current Account wall time is cut back to that time. Production OEE in a time range includes the overall percentages and a series of time points. One time point is not the range OEE.

Instants are Account wall time and have no offset. Calendar dates such as 09/11/2026 are rejected. Returned date-times are Account wall time and have no zone designator. Pass those digits back unchanged; do not convert them to UTC. The clock rules are defined in the time theme.

## Scope and Resource production pace

Optional Resource id or exact Name scopes OEE to that Resource and includes Resource production pace. It is sugar for `entityType` Resource. Combining it with a different entity type, or with a different Resource id, is a validation error: The Resource id or Name and the entity filter must identify the same Resource.

Optional `entityType` plus `entityId` scopes OEE to ResourceGroup, WorkCenter, WorkCell, Resource, or AnalysisGroup. Provide both `entityType` and `entityId`, or omit both. Never a bare group. `entityId` is a non-empty GUID, not a Name.

Resource production pace requires an open ManufacturingAction of type Production on that Resource. Setup, PlannedStop, UnplannedStop, Idle, and OutOfShift omit it, even when the Production order is Running. Several open Production actions still return one pace row. Do not invent a second. Pace is omitted when no Resource is identified, or when the Resource is not in Production. When the Resource is not in Production, the tool reports: Resource production pace is omitted because the Resource is not in Production. Do not invent a pace. OEE is still returned when pace is omitted. `paceOmittedReason` appears only when a Resource was identified and is not in Production. It is not a failed call. With no Resource, pace and `paceOmittedReason` are both absent. Keep the OEE numbers.

A ResourceGroup statistics scope keeps actions whose Operation is constrained by that Resource group. WorkCenter, WorkCell, and AnalysisGroup keep actions on the member Resources. AnalysisGroup is a statistics-only grouping of Resources. It has no scenario and no APS role. Do not apply the Resource group rule to AnalysisGroup.

## Stops and history

`get_stops` returns stop reasons in a time range. `start` is required. Omitted `end` stays omitted so the API defaults it to the current Account wall time. An `end` later than the current Account wall time is cut back to that time. Each stop reason includes name, Planned, duration as `totalTimeInMilliseconds`, and stop count as `numberOfStops`. `totalTimeInMilliseconds` is the clipped overlap of those stops inside the window, not the ManufacturingAction duration. An open stop is clipped to the window end and still counts. A stop whose clipped overlap is 0 still counts as one stop. Planned and unplanned stay distinct. This tool does not return production history.

`get_history` returns production history in a time range. `start` is required. Omitted `end` stays omitted so the API defaults it to the current Account wall time. An `end` later than the current Account wall time is cut back to that time. It returns general time buckets for the whole window (production, setup, planned stop, unplanned stop, out of shift, idle) plus a page of production-order operation history, size 20. The production, setup, planned stop, and unplanned stop buckets are clipped overlap inside the window, not the ManufacturingAction duration. Out of shift and idle are the calendar remainder in that window, not Resource status samples. Planned stop time is kept separate from unplanned stop time. This tool does not return stop-reason statistics.

`get_stops` and `get_history` use the same `entityType` and `entityId` rules as `get_oee`. Names are not accepted there. How Id and exact Name work on shop-floor tools is defined in the identity theme.

`goodQuantity` on a history row is that operation's total. `goodQuantityInRange` is postings whose posting time falls inside the window. The same split applies to scrap and rework. A page of those rows is not the Production order's produced quantity. That quantity is defined in the quantity theme.
