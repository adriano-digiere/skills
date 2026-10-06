# Quantity

Read this theme before answering how much a Production order or an operation produced, or before adding quantity fields together.

## Produced quantity of a Production order

`get_production_order` and `search_production_orders` return `expectedProductionQuantity` and `productionQuantityBalance` on the order. They do not return `goodQuantity`, `producedQuantity`, `scrapQuantity`, or `reworkQuantity` there.

`expectedProductionQuantity` is the order's target. `productionQuantityBalance` is the good quantity still left before that target. The balance never goes below zero: once good quantity meets or passes the target, the balance is 0.

Produced quantity of the order is the sum of `goodQuantity` on every operation whose `operationLevel` equals the highest `operationLevel` in the routing from `get_production_order`. That level is the final routing step. Each earlier step posts quantity for work on the same flow. Adding every operation counts that flow once per step.

`operationLevel` is routing depth. Level 0 is every operation with no previous operation. It is not a progress status. A routing with one operation has highest level 0, and that operation's `goodQuantity` is the order's produced quantity. Several operations can share the highest level, including after an operation split. Sum all of them. An operation with no next operation can still sit below the highest level; that operation is an earlier step.

On an operation, `producedQuantity` and `goodQuantity` are the same number. Rework and scrap are tracked beside it. `reworkQuantity` and `scrapQuantity` are not produced quantity. An operation's `expectedProductionQuantity` is that step's target and can differ from the order's target. `unitQuantity` scales a variable-quantity step's plan. `transferBatchQuantityNextOperation` is the good quantity that step must reach before each next operation may start. None of those is the order's produced quantity.

`expectedProductionQuantity - productionQuantityBalance` equals the order's produced quantity only while that quantity is still below the target. After the target is met, the balance stays 0 and the subtraction stops at the target. Read the last-level `goodQuantity` values.

`search_production_orders` includes operations of the moment with id, name, and Operation progress only. They carry no quantity. One `get_operation` result is that operation's quantity. The order total needs the full routing.

`product.unit` on the order and `unit` on each operation are labels for the numbers already stored. The order total is the numeric sum of last-level `goodQuantity`. Report the `unit` values on those operations with the sum.

An empty `operations` list means produced quantity 0.

## Postings and history

A `productionQuantityPostings` entry on `search_work` is one posting on one ManufacturingAction: `goodQuantity`, `scrapQuantity`, and `reworkQuantity`. Summing the postings of one operation reconstructs that operation's quantities. A posting on a Setup action is already inside the operation's `goodQuantity`, `reworkQuantity`, and `scrapQuantity`. A page of ManufacturingActions is not the routing. Summing postings across operations does not produce the order total.

`get_history` returns operations that overlapped the window, 20 per page. `goodQuantity`, `scrapQuantity`, and `reworkQuantity` on a row are that operation's totals. `goodQuantityInRange`, `scrapQuantityInRange`, and `reworkQuantityInRange` are postings whose posting time falls inside the window. Summing a page is not the Production order's produced quantity.

## Example

Routing from `get_production_order`, order `expectedProductionQuantity` 100:

| Operation | operationLevel | goodQuantity | reworkQuantity | scrapQuantity |
| --- | --- | --- | --- | --- |
| Corte | 0 | 100 | 0 | 2 |
| Usinagem | 1 | 80 | 5 | 1 |
| Embalagem A | 2 | 40 | 0 | 0 |
| Embalagem B | 2 | 30 | 1 | 0 |

Produced quantity of the order is 70. Corte's 100 is the first step. Rework on Embalagem B stays out. `productionQuantityBalance` is 30.

When Embalagem A and Embalagem B later show good 60 and 50, produced quantity is 110 and `productionQuantityBalance` is 0. Expected minus balance is 100.
