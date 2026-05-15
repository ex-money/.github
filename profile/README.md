# elixir-money

A small family of Elixir libraries for working with money — the data type, the database, and the form input — built on top of [CLDR](https://cldr.unicode.org/) via the [`localize`](https://hex.pm/packages/localize) package so that every layer is currency- and locale-aware out of the box.

## Why a dedicated money library?

Floats are wrong for money. Strings are wrong for arithmetic. Two-tuple `{amount, currency}` shapes break the moment you cross a system boundary. A library that gets this right has to do four things well:

* Represent money as **arbitrary-precision decimal** paired with a **valid** currency code (ISO 4217 fiat or ISO 24165 digital token).
* Enforce that **arithmetic only happens between the same currency** — at compile time where possible, at runtime where not.
* Round according to the **currency's actual rules**, not a generic "two decimal places" — JPY has zero fractional digits, BHD has three, CHF rounds cash to the nearest five rappen.
* Format and parse against **CLDR locale data** — `"1.234,56 €"` is correct German output, not a quirk to be normalised away.

Three sibling packages divide the work along clean seams. Each is independently usable; together they cover the round-trip from a user typing in a form field to a row in Postgres and back.

## The three libraries

### [`ex_money`](https://hex.pm/packages/ex_money) — the core type

The `Money.t/0` struct, its arithmetic (`add`, `sub`, `mult`, `div`, `cmp`, `split`, `round`), parsing (`Money.parse/2` — handles symbols, ISO codes, accounting parens, locale-aware separators), and formatting (`Money.to_string/2`). Currency metadata covers all current and historical ISO 4217 codes plus ISO 24165 digital tokens. FX support via configurable rate providers.

Use it standalone for any calculation, conversion, or display work — no Phoenix, no database, no JS.

### [`ex_money_sql`](https://hex.pm/packages/ex_money_sql) — Ecto persistence

`Money.Ecto.Composite.Type` for Postgres (a real composite type that round-trips amount **and** currency in one column), `Money.Ecto.Map.Type` for MySQL (JSON with the decimal as a string to preserve precision), and the migration helpers that create them. Queries can filter and sort on the composite type directly; aggregates compose cleanly.

Use it whenever a money value needs to land in a relational database without losing the currency.

### [`ex_money_input`](https://hex.pm/packages/ex_money_input) — Phoenix LiveView form input

The `<.money_input>` and `<.currency_picker>` HEEx components, an AutoNumeric-backed JS hook for live formatting and cursor preservation, an Ecto changeset bridge (`cast_money`, `validate_money`), and a Plug-based visualizer for previewing every component × locale × currency combination during development.

Submits a `%{"amount", "currency"}` map that `Money.Ecto.Composite.Type` casts directly — no custom param-flattening, no canonical-vs-locale ambiguity on the wire. Use it whenever a user types money into a form.

For plain locale-aware *number* input (no currency), see the sibling [`localize_inputs`](https://hex.pm/packages/localize_inputs) package — same architecture, no currency layer.

## License

All three packages are Apache-2.0.
