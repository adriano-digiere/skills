# Identity

Read this theme before passing an Id or a Name. A GUID is an Id. Any other value is a Name, compared exactly, except where this theme says Contains or search text.

## Id or exact Name

These tools take an Id (GUID) or an exact Name: Resource on `get_resource_shop_floor` and on `search_work`; Production order on `get_production_order` and on `search_work`; and, on `search_work`, Resource group, Work center, Work cell, Work shift, Product, Operator, and stop reason. The API treats a GUID as Id and any other value as Name. A GUID-shaped string is an Id even when a record is named with that GUID. It is not looked up as text. Operator identity is the stored Operator name and includes inactive Operators. It is not the linked User's name or email. Work shift identity includes inactive shifts. A picker that hides one is not proof it is missing.

`search_resources` is the exception for Resources. Its optional predicate is a Contains match on Resource name. Multiple matches are a list, not an identity error. A GUID-shaped Name can still match here.

`search_production_orders` search text is optional. Empty search text matches by Production order progress only. It is a Contains match on Production order name, Product name, sales order, and customer. Multiple matches are a list, not an identity error. It is not an exact Name lookup. Exact Name is `get_production_order`.

## Operation

`get_operation` requires an Operation Id as a GUID. Names are not accepted. Call `get_production_order` first and use an operation id from the routing. An empty GUID is not a valid operation id.

On `search_work`, Operation Name requires a Production order. Without an order, operation must be a GUID.

## Statistics ids

On `get_oee`, `get_stops`, and `get_history`, `entityId` is a non-empty GUID. Names are not accepted. On `get_oee`, an empty GUID is not a valid Resource id. `entityType` and `entityId` are both set or both omitted. Resource id or exact Name on `get_oee` is sugar for `entityType` Resource only. What each entity type means is defined in the statistics theme.

## Groups are not member lists

A Resource group Name is not a request for every Resource in the group. On `search_work`, a Resource group matches only when the Resource is in that group and the Operation is planned on that same Resource group. Work center and Work cell match membership. Stop reason identity includes the Automatic stop reason. That is the account stop reason with the Automatic flag, not a reason named Automatic. Exact Name and Id still include it.
