# Example: Domain Glossary

This is what a finished domain glossary looks like. Yours should use your project's actual terms — this uses a trading system as a placeholder.

---

# Domain Glossary

## Order
A request to buy or sell a specific quantity of an Instrument at a given price.
An Order always belongs to exactly one Account. An Order is immutable once filled.

## Instrument
A tradeable asset identified by a unique symbol (uppercase, 3-8 alphanumeric chars).
Examples: BTC, ETH, AAPL. An Instrument exists independently of any Order.

## Account
A container for Orders and Positions belonging to a single user.
Accounts have a balance (denominated in a single settlement currency)
and zero or more open Positions.

## Position
The net exposure to an Instrument within an Account.
Created when an Order fills. Closed when exposure reaches zero.
A Position can be long (positive quantity) or short (negative quantity).
