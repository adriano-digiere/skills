---
name: aloee-mcp
description: >-
  Applies the Aloee shop-floor MCP contract for identity, status vocabularies,
  produced quantity, statistics, and Account wall time. Use whenever Aloee MCP
  tools are available and the user asks about production, the shop floor, OEE,
  pace, stops, downtime, production orders, operations, quantity, resources,
  machines, shifts, operators, apontamento, paradas, ordem de produção,
  quantidade produzida, saldo, operação, recurso, máquina, turno, or operador —
  including get_oee, get_stops, get_history,
  search_work, list_running_work, get_resource_shop_floor, search_resources,
  search_production_orders, get_production_order, get_operation,
  get_account_date_time, and get_account_time_zone, even when they do not name
  a tool. Read it before passing an id, a name, a status, or a date, and before
  explaining a returned clock, percentage, or quantity.
compatibility: Requires the Aloee MCP tools in the agent session.
---

# Aloee MCP

Answer shop-floor questions for one Account by calling the Aloee MCP. The tool schemas repeat some rules and still describe offset-less instants as UTC. The themes in `references/` are the contract. When a schema disagrees with a theme, follow the theme. The API compares the clock digits you send with Account wall time. Converting those digits to UTC, or adding `Z` or a numeric offset, moves the window.

Reply in the user's language. Keep enum strings, tool names, and identifiers exactly as the themes and the payload specify. If the Aloee tools are not available in this session, say so. Do not invent statuses, OEE, paces, quantities, or times.

## Read the themes that apply

Read each matching file before you choose arguments. One question often needs more than one theme. The sections below only route the call. They do not replace the theme.

- Passing an Id or a Name, or deciding exact match versus Contains: [identity](references/identity.md)
- Naming a status, a stop, a progress, or an open ManufacturingAction: [shop floor](references/shop-floor.md)
- Answering how much a Production order or an operation produced, or reading a quantity field: [quantity](references/quantity.md)
- Answering with OEE, pace, stop-reason totals, or production history: [statistics](references/statistics.md)
- Reading a clock digit or sending a date: [time](references/time.md)

## Pick the tool from the question

| The user wants | Call |
| --- | --- |
| The current Account clock | `get_account_date_time` |
| The Account timezone id, so a returned clock can be read | `get_account_time_zone` |
| Resources whose name contains a fragment | `search_resources` |
| One Resource's status and its open ManufacturingActions | `get_resource_shop_floor` |
| Production orders by text and progress | `search_production_orders` |
| One Production order and its full routing | `get_production_order` |
| One operation | `get_operation` with an operation GUID from that routing |
| Open ManufacturingActions in the Account now | `list_running_work` |
| Filtered or historical ManufacturingActions | `search_work` |
| OEE, or Resource production pace | `get_oee` |
| Stop-reason totals in a range | `get_stops` |
| Time buckets and production-order operation history | `get_history` |

`entityType` on the statistics tools is case-sensitive: `ResourceGroup`, `WorkCenter`, `WorkCell`, `Resource`, `AnalysisGroup`.

`search_resources` and `search_production_orders` return lists. Several hits are not an identity failure. Use exact Name or Id on a get tool only after the user, or the payload, identifies one record.

On `get_stops` and `get_history`, and for any statistics scope that is not the Resource sugar on `get_oee`, a Name is not an id. If you only have a Resource group, Work center, Work cell, or Analysis group name, you do not yet have an `entityId`. Do not pass that name, and do not invent a GUID.

## Say what the payload actually is

Resource status, Production order progress, Operation progress, and ManufacturingAction type are four vocabularies. A shared word is not a shared fact. Explain a value in the user's language only after you name which vocabulary it came from.

The order object has `expectedProductionQuantity` and `productionQuantityBalance`. It has no produced-quantity field. Produced quantity of the order is the sum of `goodQuantity` at the highest `operationLevel` in the routing. `producedQuantity` on an operation is that operation's `goodQuantity`. The quantity theme defines which fields stay out of that sum.

Returned date-times have no zone designator. Repeat those digits. Call `get_account_time_zone` when the user needs to know which clock they belong to.

`search_work`, `search_resources`, `search_production_orders`, `list_running_work`, and the operation-history page of `get_history` return 20 rows. Say when the answer is one page. `search_work` is newest Start first. Operations included on `search_production_orders` are operations of the moment, not the routing. The routing is `get_production_order`.

HTML Note is omitted. `HasNote` or `HasAnnotation` means a note exists and was left out of the tool result. Do not treat the flag as an empty note.

An empty page is an empty match. It is not a missing master-data record and not a failed call. A tool error string is a failed call. Report that string.

When pace is omitted, keep the OEE figures. Repeat `paceOmittedReason` when it is present. Do not calculate a pace.

## Leave arguments out on purpose

Omit `end` when the theme says the API defaults it. On `search_work`, a supplied `end` is kept, including one later than now. Do not add an `end` the user did not ask for. `end` is never valid without `start`.

Do not send `Z`, a numeric offset, or a calendar date such as `09/11/2026`. A date-only `yyyy-MM-dd` is midnight at the start of that Account day.

Do not put a duration on an open ManufacturingAction. The payload has no End.

Do not treat a Resource group name as the list of member Resources.

## Examples

**"A prensa 3 está parada?"**

Call `get_resource_shop_floor` with the exact Name. Answer with the Resource status enum you received. `PlannedStop` on that enum is not Production order progress `Stopped`. Do not replace the enum with livre, produzindo, or parada as if those were the stored values. You may explain the enum after you name it.

**"Qual o OEE de hoje da prensa 3?"**

Call `get_oee` with the Resource exact Name and no `start`. That is Current production OEE: the Account-local calendar day from midnight through now. It is not the current shift and not a live snapshot. Pace is present only while that Resource has an open ManufacturingAction of type Production.

**"Quanto a OP 12345 já produziu?"**

Call `get_production_order`. Sum `goodQuantity` of the operations at the highest `operationLevel` only. `producedQuantity` on an operation is that same `goodQuantity`. Leave rework and scrap out. `productionQuantityBalance` is the good quantity still left of `expectedProductionQuantity`, and it stays 0 after the target is met, so expected minus balance is the produced quantity only while the order is still short of the target.

**"Paradas não planejadas do centro Usinagem nesta semana"**

`get_stops` needs `start` as offset-less Account wall time, and `entityType` `WorkCenter` with `entityId` as a GUID. The center's name is not that id. Omit `end`. Keep planned and unplanned separate. `totalTimeInMilliseconds` is clipped overlap inside the window, not the ManufacturingAction duration. Stop-reason totals are not production history. History is `get_history`.
