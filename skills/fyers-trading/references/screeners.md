# Screeners (market data)

Ready-made scans over the cash, F&O and commodity markets. Four GET endpoints under the **REST base
`https://api-t1.fyers.in/api/v3`** (not `/data`): `/screeners/config`, `/screeners/standard`,
`/screeners/candlestick`, `/screeners/technical`. Auth header `app_id:access_token`, as for every API.
Python: `fyers-apiv3>=3.2.1` has `screeners_config/standard/candlestick/technical` (3.2.0 had a broken
`screeners_standard`). `scripts/fyers_client.py` has stdlib wrappers: `screeners_config()`,
`screeners_standard(...)`, `screeners_candlestick(id)`, `screeners_technical(id)` (rows as dicts).

## Workflow
1. `screeners_config()` once. It lists 111 standard, 23 candlestick and 6 technical screeners with the
   parameters each accepts (`supported_params`), 118 columns grouped as `column_groups.equity|fno|commodity`,
   `universes` (cash ids like `iw001` Nifty 50, default `iw010`), futures/options universes (`fw001`, `ow001`...),
   16 timeframes (`t0_1min` ... `alltime`, `c1_week`...), `option_types` (CE/PE) and `expiries` (epoch values).
2. Run a screener with only parameters its config entry lists.

## Standard screeners
`GET /screeners/standard?screener_id=top_gainers&fields=ltp,chp&universe=iw001&order_by=chp&order=desc`
- `screener_id` required. Optional: `fields` (max 10, comma separated; `symbolticker` is always returned and
  not counted; **omitted `fields` returns only symbolticker**), `universe`, `timeframe`, `order_by`,
  `order` (`asc|desc`), `expiry` (epoch from config, only for futures/options universes), `option_type`.
- Max **500 rows**, no paging: `limit`, `offset`, `page` return 400. Parameter names are case-sensitive,
  values are not. Unsupported parameter, field, universe, timeframe or expiry -> 400 with the allowed list.
- Response: `data.header` (`unique_key`, `description`, `data_type`), `data.body` (list of rows, same
  order as the header), plus `order`, `order_by`, `timeframe`, `universe`, `expiry`, `option_type`,
  `fetch_time`, `last_update_time`. An empty `body` is valid (nothing matched).
- Data is end-of-day for cash/F&O (`last_update_time` 15:30 / 15:40 IST); commodity screeners update with the
  MCX session.

## Candlestick and technical screeners
`GET /screeners/candlestick?screener_id=<id>` and `GET /screeners/technical?screener_id=<id>`: fixed
screeners, **only `screener_id` is accepted** (anything else is 400). Ids come from `screeners_config()`.
Technical screeners scan the Nifty 500 universe.

## Errors and limits
401 code -16 (token), 405 `method not allowed; use GET`, 400 code -50 for bad/missing parameters, 404 for a
wrong path. Limits: Standard plan 50 requests/day; Prime 10/s and 500/min. Do not poll screeners in a loop.
