# Clean code

## Names

- A name says what a thing is or does in domain words: `pending_balance`,
  `cancelUnpaidOrders()`, `isAuctionClosed`. No abbreviations except
  universal ones (`id`, `url`), no `data`, `info`, `item`, `tmp`, `obj`,
  `handleStuff`, single letters outside tiny lambdas.
- Booleans read as questions: `is_verified`, `has_discount`, `canShip`.
- Functions are verbs, values are nouns, collections are plural.
- One concept, one word across the codebase (`store` everywhere, not
  `store`/`shop`/`vendor`).

## Functions

- One responsibility per function; if its description needs "and", split it.
- Keep functions short (about 30 lines); extract named steps.
- Return early instead of nesting; at most two levels of indentation in logic.
- At most three parameters; beyond that pass one object typed from the
  canonical type.
- No boolean flag parameters that switch behaviour; write two functions.
- Pure where possible: inputs in, output out, no hidden reads of globals.
- Side effects (DB writes, network, events) live at the edges, called from
  thin controllers/handlers.

## Values

- No magic numbers or strings: limits, fees, durations, status strings, route
  paths and storage keys are named constants with one home.
- No duplicated literals across files; the second copy moves to the shared
  constant.
- Money, time and ids use the project's dedicated types/helpers, never ad hoc
  math repeated in several places.

## Errors

- Fail loudly at the boundary: validate input once at the edge, trust it
  inside.
- Throw typed/domain errors with a stable `code`; don't return `null` to mean
  "error".
- Never swallow an exception silently; handle it, translate it, or let it
  propagate.

## Structure

- Files stay under about 300 lines; split by responsibility, not by line
  count.
- Modules expose a small public surface (`index.ts` barrels); internals stay
  private.
- No dead code: unused exports, unreachable branches, commented-out code,
  feature flags that are always on, files nothing imports.
- No speculative abstraction: build for the cases that exist; extract on the
  second real occurrence.
- Consistency beats preference: follow the pattern the codebase already uses
  for the same kind of thing.

## Comments

None. If code needs a comment, rename, extract or restructure until it
doesn't. Allowed survivors are tool directives only (see SKILL.md rule 3).
Delete commented-out code, section banners, TODOs without a ticket, JSDoc that
restates a signature, and change narratives.
