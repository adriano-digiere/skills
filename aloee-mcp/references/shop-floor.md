# Shop floor

Read this theme before naming a status, a stop, or an open ManufacturingAction. Resource status, Production order progress, Operation progress, and ManufacturingAction type are four vocabularies. A shared word does not make them the same fact.

## Resource status

`get_resource_shop_floor` returns Resource status as the enum string. The values are Production, Setup, PlannedStop, UnplannedStop, OutOfShift, and Idle. Status is not mapped to free, producing, or stopped.

Resource status PlannedStop is not Production order progress Stopped.

## Production order progress

`search_production_orders` filters Production order progress: NotInitiated, Stopped, Running, and Ended. This is Production order progress, not Resource status. Running means at least one Operation is Production or Setup. Stopped means the order has started, is not Running, and is not Ended. PlannedStop, UnplannedStop, and Suspended on the remaining operations are still order Stopped. When all four flags are omitted, all are true. At least one flag must be true.

`get_production_order` returns that same progress on the Production order: NotInitiated, Stopped, Running, and Ended. `search_production_orders` can also include operations of the moment. Progress on those operations is Operation progress, not Production order progress. Operations of the moment are not the full routing. The full routing is `get_production_order`. Progress is not quantity. How produced quantity is read from that routing is defined in the quantity theme.

## Operation progress

An operation from `get_production_order` or `get_operation` uses Operation progress: NotInitiated, Production, Suspended, Ended, PlannedStop, UnplannedStop, and Setup. Operation progress Production is not Production order progress Running. Operation progress PlannedStop is not Production order progress Stopped, and it is not Resource status PlannedStop. Operation progress Suspended is not a ManufacturingAction type and not a Resource status. ProductionSuspension and OperationReopening are the action types that lead there, when nothing is open. `list_running_work` does not return it. Operation progress Ended follows a last action of ProductionEnding or OperationEnding, when nothing is open. The Production order is Ended only when every operation is Ended.

## ManufacturingAction type

A ManufacturingAction type is Production, Setup, PlannedStop, UnplannedStop, ProductionSuspension, ProductionEnding, OperationEnding, or OperationReopening.

`list_running_work` returns open ManufacturingActions in this Account now. It includes Production, Setup, PlannedStop, and UnplannedStop — not production-only. Use `search_work` for filtered or historical ManufacturingActions.

`search_work` `start` and `end` are a time window of intersection, not "started after start". Both bounds keep actions that overlap the interval. An open ManufacturingAction matches if it started on or before `end`. `start` alone keeps every currently open action plus closed ones that ended on or after `start`. Do not invent `end`. A supplied `end` is kept, including one later than the current Account wall time.

`stopReason` is a reason on that action. Use PlannedStop or UnplannedStop with the reason, not Production. Planned and unplanned stay distinct. Identity of a stop reason includes the Automatic stop reason, the one with the Automatic flag.

Completion is Open or Closed. Omit completion to include both.

A Resource group matches only when the Resource is in that group and the Operation is planned on that same Resource group — not membership-only like Work center or Work cell. Product, Production order, Operation, and Resource group require an Operation on the action. A standalone resource stop has none and will not match those. Filters on `search_work` combine. A Resource that does not satisfy the Resource group, Work center, or Work cell is an empty page, not an error and not a missing Resource.

Duration bounds are minutes of ManufacturingAction duration, from Start to End, or from Start to the Account's current date and time when the action is still open. They are not the clipped window overlap. Open rows still have no duration in the payload, because there is no End. Do not invent one.

`search_work` page size is 20, newest Start first. Nested HasNote flags are kept. HTML Note is omitted. `list_running_work` keeps nested HasNote flags and omits HTML Note. `get_production_order` omits HTML Note and keeps HasNote and HasAnnotation flags when they are present.

Use `get_stops` for stop-reason totals and `get_history` for production time buckets. Those results are defined in the statistics theme. How to send ISO-8601 arguments is defined in the time theme.
