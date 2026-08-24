---
name: nestjs-convention
description: Use when editing NestJS controllers, DTOs, or `@nestjs/swagger` OpenAPI metadata — Zod DTOs with `z.infer` plus `ZodValidationPipe` (no `@ApiProperty` classes), and all `*Options` decorators for `@nestjs/swagger` must import named exports from colocated `swagger/*.swagger.ts` (never inline in `*.controller.ts`).
---

# NestJS conventions

Applies when editing **NestJS HTTP API** code (controllers, DTOs, OpenAPI
metadata).

## DTOs and OpenAPI

Use **Zod + `z.infer`** instead of `@nestjs/swagger` class DTOs with
`@ApiProperty`.

### Zod DTOs (`*.dto.ts`)

1. Export `const fooSchema = z.object({ ... })` and
   `export type Foo = z.infer<typeof fooSchema>`.
2. **Inputs** — `new ZodValidationPipe(fooSchema)` on `@Body()`, `@Param()`,
   or `@Query()`. Use `z.object({ paramName: ... })` when the pipe receives
   the full params object.
3. **Responses (optional)** — `responseSchema.parse(await this.service...)`
   when server output must be checked in development.

```typescript
// ✅
export const widgetSchema = z.object({ id: z.string().uuid() });
export type Widget = z.infer<typeof widgetSchema>;

// ❌ Swagger class DTOs
export class WidgetDto {
  @ApiProperty() id!: string;
}
```

### OpenAPI options — **only** in `*.swagger.ts`

**Every option object** for `@nestjs/swagger` route decorators MUST be a
**named export** in colocated `**/swagger/*.swagger.ts`. Controllers import
and pass as the **sole** decorator argument.

**Applies to:** `@ApiOperation`, `@ApiBody`, `@ApiOkResponse`,
`@ApiCreatedResponse`, `@ApiAcceptedResponse`, `@ApiNoContentResponse`,
`@ApiResponse`, `@ApiParam`, `@ApiQuery`, `@ApiConsumes`, `@ApiProduces`,
`@ApiExtraModels`, and similar `*Options` decorators.

```typescript
// ❌ Forbidden in *.controller.ts
@ApiOperation({ summary: "Create widget" })
@ApiBody({ schema: { type: "object", properties: { ... } } })

// ✅
import { createWidgetApiOperationOptions, createWidgetBodyOptions } from "./swagger/create-widget.swagger";

@ApiOperation(createWidgetApiOperationOptions)
@ApiBody(createWidgetBodyOptions)
```

#### `*.swagger.ts` rules

| Rule        | Detail                                                                                          |
| ----------- | ----------------------------------------------------------------------------------------------- |
| **Location**  | `swagger/<feature>.swagger.ts` next to the controller module                                    |
| **Types**     | Import `ApiOperationOptions`, `ApiBodyOptions`, `ApiResponseOptions`, etc. from `@nestjs/swagger` |
| **Naming**    | `{verb}{Resource}ApiOperationOptions`, `{verb}{Resource}BodyOptions`, …                          |
| **Schemas**   | Manual OpenAPI `schema` objects; keep aligned with Zod DTOs                                     |
| **Sync**      | Zod or route contract change → update matching `*.swagger.ts` in the same change               |

```typescript
export const createWidgetApiOperationOptions: ApiOperationOptions = {
  summary: "Create a widget",
};
export const createWidgetBodyOptions: ApiBodyOptions = {
  schema: { type: "object", required: ["name"], properties: { name: { type: "string" } } },
};
```

#### `@ApiTags`

Export tag in `swagger/<module>.swagger.ts` and use `@ApiTags(widgetsApiTags)`
on the **controller class**. No new inline tag strings on routes you add or
refactor.

#### Legacy controllers

If you touch a route with **inline** `@Api*` options, **move** them into
`*.swagger.ts` in the same change.