# Time

Read this theme before interpreting a clock digit or sending a date argument.

## Returned values are Account wall time

Tools that return date-times return Account wall time. Those values have no zone designator. Pass those digits back unchanged; do not convert them to UTC.

`timeZoneId` from `get_account_time_zone` is the Account timezone. Use it to interpret Account wall time returned by other tools. Do not treat those clock digits as UTC.

`get_account_date_time` is the current Account wall time. When the Account timezone id is needed, call `get_account_time_zone`.

## Arguments are Account wall time

`search_work`, `get_oee`, `get_stops`, and `get_history` accept ISO-8601 with no offset. Those digits are Account wall time. A date-only `yyyy-MM-dd` is midnight at the start of that Account calendar day. That day runs until midnight at the start of the next day. A start later than the end is not a window. Equal bounds still are. An instant with `Z` or a numeric offset is rejected. Calendar dates such as 09/11/2026 are rejected.

Do not invent end. `end` requires `start`.

- `get_stops` and `get_history`: `start` is required. Omitted `end` stays omitted so the API defaults it to the current Account wall time. An `end` later than the current Account wall time is cut back to that time.
- `get_oee`: omitted `start` means Current production OEE, defined in the statistics theme. When `start` is set, omit `end` to let the API default it. An `end` later than the current Account wall time is cut back to that time.
- `search_work`: omitted `end` stays omitted. A supplied `end` is kept, including one later than the current Account wall time. Which ManufacturingActions overlap that window is defined in the shop-floor theme.

`get_account_time_zone` and `get_account_date_time` take no time arguments.
