# One name end to end

## The chain

```
DB column  →  entity property  →  API JSON key  →  frontend type  →  form field name  →  request body / query param  →  DTO property  →  DB column
phone_number   phone_number        phone_number      phone_number       name="phone_number"   phone_number                     phone_number      phone_number
```

Every arrow is the identity. If a name changes anywhere in the chain, the
chain is broken.

## Not allowed

- A global interceptor/serializer that converts key case
  (`camelCase` ↔ `snake_case`) in either direction. Pick one case for data
  fields and use it in the database, the code that touches the data, the wire
  and the UI.
- Mappers that copy fields under new names
  (`{ name: p.title }`, `phone: dto.mobile`, `fullName: u.display_name`).
- Frontend adapters `fromApi()` / `toApi()` / `toPayload()` that rename keys
  between the response, the component state, the form and the request.
- Form field names invented for the screen (`f1`, `value`, `input2`,
  `mobile` for a `phone_number` column).
- Two names for one thing in different screens or endpoints
  (`store_id` here, `shop_id` there).
- Translation keys or labels that describe a different thing than the field
  name.

## Allowed

- **Omitting** fields that must not leave the server: password hashes,
  refresh tokens, OTP codes, provider secrets, internal moderation notes.
  Omit by selecting fields or with a serializer exclude — never by renaming.
- **Adding** computed fields under an honest new name (`can_cancel`,
  `average_rating`, `server_time`). They are new facts, not renamed ones.
- **Nesting** a related entity under its relation name (`store`, `category`)
  with its own unchanged field names.
- **Formatting for display** at render time (dates, money, digits) without
  storing the formatted value under the field's name.
- Code identifiers that are not data (functions, local variables, React
  components) follow the language's normal case.

## Choosing the case

Use the case the database already uses for columns (for Postgres usually
`snake_case`) for every data field in every layer. In TypeScript that means
entity properties and DTO properties are `snake_case`
(`created_at: Date`); this is intentional and lint rules that forbid it are
configured to allow data properties.

## Renaming safely

Renaming a column is a data migration, not an edit:

1. Rename at the source with `ALTER TABLE … RENAME COLUMN` (a migration), or
   keep the column and only rename the code property when the column already
   has the target name (`@Column({ name: 'display_name' }) displayName` →
   `@Column() display_name`).
2. With `synchronize: true` environments, prove the schema diff is empty
   before shipping: build the app, point it at a copy of the real schema and
   log the pending schema changes (`dataSource.driver.createSchemaBuilder().log()`
   in TypeORM). Any `DROP COLUMN` / `ADD COLUMN` pair is data loss — stop.
3. Rename the property with a type-aware tool (IDE rename, `ts-morph`), then
   grep for the old name in strings: query-builder aliases
   (`'u.displayName'`), `@Index(['displayName'])`, `order: { … }`, raw SQL,
   JSON stored in `jsonb` columns, socket event payloads, tests, fixtures.
4. Update the frontend type, form names and request bodies in the same
   change, and run both sides end to end.
