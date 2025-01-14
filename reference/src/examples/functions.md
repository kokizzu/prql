This mostly works, apart from window clauses, and the version & db args on the
first line.

```prql_no_test
prql version:0.1 db:snowflake                         # Version number & database name.

func lag_day x ->  (
  window x
  by sec_id
  sort date
  lag 1
)
func ret x ->  x / (x | lag_day) - 1 + dividend_return
func excess x ->  (x - interest_rate) / 252
func if_valid x ->  is_valid_price ? x : null

from prices
derive [
  return_total =      prices_adj   | ret | if_valid  # `|` can be used rather than newlines.
  return_usd =        prices_usd   | ret | if_valid
  return_excess =     return_total | excess
  return_usd_excess = return_usd   | excess
]
select [
  date,
  sec_id,
  return_total,
  return_usd,
  return_excess,
  return_usd_excess,
]
```

## Notes

- The SQL can be a bit better, by pulling the window clause out.
