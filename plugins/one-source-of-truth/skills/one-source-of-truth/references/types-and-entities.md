# Types and entities

## Where the canonical definition lives

| Stack | Canonical source | Everything else |
|---|---|---|
| Backend with an ORM (TypeORM, Prisma, Drizzle) | The entity / schema model | DTOs derive with `PickType`, `OmitType`, `PartialType`, `IntersectionType` (`@nestjs/mapped-types` or `@nestjs/swagger`), Prisma/Drizzle inferred types |
| Frontend with a generated client (OpenAPI) | The generated model | Aliases and `Pick`/`Omit` of it; never a hand-written copy |
| Frontend without a generated client | One `types.ts` per entity in the owning feature, written to match the API contract field for field | Form values = `z.infer` of the schema whose keys are those same fields |
| Validation | One schema per input (Zod, class-validator DTO) | Types inferred from it (`z.infer`), not declared next to it |
| Enums and status values | One `as const` object or string-literal union, exported once | `typeof STATUS[keyof typeof STATUS]`, never a second union with the same strings |

## Deriving variants

```ts
type Product = components["schemas"]["Product"];
type ProductCard = Pick<Product, "id" | "title" | "price" | "cover_url">;
type ProductCreate = Omit<Product, "id" | "created_at" | "updated_at">;
type ProductUpdate = Partial<ProductCreate>;
type ProductStatus = Product["status"];
```

```ts
export class CreateProductDto extends PickType(Product, ["title", "price", "category_id"] as const) {}
export class UpdateProductDto extends PartialType(CreateProductDto) {}
```

## Smells that mean a second source exists

- Two `interface`/`type`/`class` declarations whose field sets overlap by
  more than half, or that share a name across files (`Product` in `api.ts`
  and `Product` in `types.ts`).
- A union of string literals that repeats an enum declared elsewhere.
- A constant (page size, max length, fee, regex, URL, route path) typed as a
  literal in more than one file.
- Validation limits (max length, min price) declared in the backend DTO and
  again by hand in the frontend schema. Keep the number once (shared
  package, generated from OpenAPI, or a single frontend constants module that
  cites the contract) and import it.
- A frontend mock/fixture type that is not the real type.

## Across repositories

The API contract (OpenAPI file, generated client, or the contract document
the project names) is the bridge. Backend owns it; frontend and admin consume
it. When both sides hand-write types, they must be identical in names and
shapes; a difference is a bug on one side, fixed at the side that is wrong.
