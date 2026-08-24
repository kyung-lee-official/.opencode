---
name: frontend-convention
description: Use when editing React/Next.js components, forms, or API clients — user-visible dates as `YYYY-MM-DD HH:mm:ss`, Tailwind built-in utility classes instead of arbitrary `[...]` values, `react-hook-form` + `zodResolver` for forms, and `useQuery` / `useMutation` (TanStack Query) instead of `useState` per field or `useEffect` + `fetch`.
---

# Frontend conventions

## Date and time display

Use **`YYYY-MM-DD HH:mm:ss`** for user-visible dates and times (tables,
labels, exports, API examples in docs, human-oriented log lines) unless the
task or product copy **explicitly** requires another format (locale-relative
time, ISO-8601 with `T`, date-only fields).

- **dayjs:** `dayjs(d).format("YYYY-MM-DD HH:mm:ss")` or a shared helper using
  this pattern.
- **Avoid** ad hoc formats (`YYYY-MM-DD HH:mm`, `MM/DD/YYYY`,
  `toLocaleString()`) unless required.
- **Exceptions** (state in the request or API contract): HTML `datetime-local`,
  file names, third-party APIs, RFC3339, calendar-date-only `YYYY-MM-DD` —
  add a short comment at the call site.

Before finishing UI or string output with a timestamp, confirm the format
matches **`YYYY-MM-DD HH:mm:ss`** or a documented exception.

---

## Forms and data fetching

Applies when editing **frontend UI** (React, Next.js, or similar). Mandatory
for new or refactored surfaces.

### Forms

Use **react-hook-form** for field values, submit, and validation. Do **not**
use `useState` per input or hand-rolled validation for standard forms.

Use **Zod** (`import { z } from "zod"`) with **`zodResolver`** from
`@hookform/resolvers/zod`.

```typescript
// ✅ GOOD
const schema = z.object({ name: z.string().trim().min(1) });
type FormValues = z.infer<typeof schema>;
const form = useForm<FormValues>({
  resolver: zodResolver(schema),
  defaultValues: { name: "" },
});

// ❌ BAD — ad-hoc field state / validation
const [name, setName] = useState("");
const nameError = name.trim() ? null : "Required";
```

- Surface errors via `formState.errors` (or RHF-controlled fields); disable
  submit while `formState.isSubmitting` or mutation pending.
- `useState` only for **non-field** UI (dialogs, tabs, ephemeral drafts not
  modeled as fields).

### Server state and HTTP

Use **TanStack Query** (`useQuery`, `useMutation`, `useQueries`,
`useQueryClient`) for caching, loading/error, and invalidation. Do **not**
call `fetch` directly in components or UI hooks.

Route HTTP through the **project's API client wrapper** (e.g. a shared
`fetch` helper), in **dedicated API modules**. Components pass those
functions to `queryFn` / `mutationFn`.

```typescript
// ✅ API module + TanStack Query
export async function updateItem(id: string, body: UpdateItemBody) {
  return apiClient.put<Item>(`/items/${id}`, body);
}
const mutation = useMutation({
  mutationFn: (values: FormValues) => updateItem(id, values),
  onSuccess: () =>
    queryClient.invalidateQueries({ queryKey: [ItemQueryKey.List] }),
});

// ❌ BAD — useEffect + fetch in component; axios ad hoc
```

- Stable **query keys** (enums or factories) next to the API module.
- Type errors with the project's **HTTP error type** where applicable.
- No alternate HTTP clients for app API calls unless the project already
  standardizes on one.

### Legacy

When editing files still on `useState` forms or inline `fetch`, migrate to
this pattern in the same change when practical; do not expand legacy patterns
for new fields or endpoints.

---

## Tailwind CSS

Use Tailwind's **built-in utility classes** instead of arbitrary values
(`[...]`) whenever a standard class exists for the design intent.

| Don't            | Do                                              |
| ---------------- | ----------------------------------------------- |
| `max-w-[1920px]` | `max-w-480` (Tailwind v4) or `max-w-screen-2xl` |
| `text-[14px]`    | `text-sm`                                       |
| `p-[16px]`       | `p-4`                                           |
| `gap-[8px]`      | `gap-2`                                         |
| `rounded-[4px]`  | `rounded`                                       |
| `w-[50%]`        | `w-1/2`                                         |

Use arbitrary values **only** when no built-in class matches the design
(brand colors, precise pixel offsets from a spec, one-off animations). When
you do use one, prefer the Tailwind v4 bare-number syntax (`max-w-480`) over
the bracket syntax (`max-w-[1920px]`) when the value maps to the default
spacing/sizing scale.