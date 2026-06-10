# React Agent Instructions

These instructions define a reusable structure for Vite React TypeScript applications that use a feature-first frontend architecture.

This file captures LegioSoft frontend architecture patterns for AI-assisted development. It should guide agents toward maintainable project structure without referencing private projects or local machine paths. For more about LegioSoft, visit [legiosoft.net](https://legiosoft.net/).

The purpose of this file is to help AI agents place new code in the correct folder, preserve ownership boundaries, and avoid turning the project into a flat collection of unrelated components and helpers.

## Core Architecture

Use a feature-first structure.

Feature-first means business functionality is grouped by domain, not by technical file type. A billing feature, products feature, users feature, or settings feature should keep most of its own components, queries, schemas, types, constants, hooks, modals, and helpers inside its feature folder.

Global folders still exist, but they are reserved for code that is genuinely reused across multiple features.

This structure keeps related code close together:

- A feature can be changed without searching through the whole app.
- Shared code stays intentional instead of becoming a dumping ground.
- Route files stay thin and delegate real UI/workflow logic to features.
- API/query logic remains close to the UI and types that consume it.

## Expected Source Layout

Use this top-level `src` shape when it matches the project:

```text
src/
  api/
  assets/
  components/
  constants/
  features/
  hooks/
  lib/
  routes/
  theme/
  types/
  utils/
  App.tsx
  main.tsx
  router.ts
```

Each folder has a specific job.

## `src/api`

Use `src/api` for shared HTTP plumbing and cross-feature API helpers.

Put these here:

- API client creation.
- Axios/fetch wrappers.
- Request and response interceptors.
- Shared API result types or unwrapping helpers.
- Auth token attachment and common error normalization.
- API helpers used by more than one feature.

Do not put every feature endpoint here by default. If an endpoint only belongs to one feature, prefer a query or service file inside that feature folder.

Good pattern:

```text
src/api/
  api.ts
  interceptor.ts
  apiResponseError.ts
  index.ts
```

## `src/assets`

Use `src/assets` for static files that are bundled with the frontend.

Common subfolders:

```text
src/assets/
  icons/
  images/
  locales/
```

Use `icons` for SVGs and icon sprites. Use `images` for raster images such as PNG, JPG, or WebP. Use `locales` for translation JSON files.

For localization, prefer this shape:

```text
src/assets/locales/
  en/translation.json
  fr/translation.json
  de/translation.json
```

When adding user-facing text, add translation keys instead of hardcoding copy in components unless the project explicitly does not use localization.

## `src/components`

Use `src/components` for reusable app-level UI components.

These components should not belong to one business feature. They should be useful across the application.

Good candidates:

- Layout shells.
- Inputs.
- Selects.
- Tables.
- Pagination.
- Modal wrappers.
- Alert dialogs.
- App headers.
- Sidebars.
- Shared empty/loading/error states.

Use subfolders by component family:

```text
src/components/
  Form/
  Input/
  Layout/
  Pagination/
  Select/
  Table/
```

Do not move feature-specific UI here just because another feature might use it later. Promote code to `src/components` only after reuse is real or clearly required.

## `src/constants`

Use `src/constants` for constants shared across features.

Examples:

- Global route labels.
- Locale lists.
- Currency lists.
- Shared query keys.
- App-wide option sets.

If a constant is only meaningful inside one feature, keep it in `src/features/{feature}/constants`.

## `src/features`

Use `src/features` for business-domain modules.

Each feature should own the code required to implement that domain. A feature folder may contain:

```text
src/features/{feature}/
  components/
  constants/
  hooks/
  modals/
  queries/
  types/
  utils/
  validation/
  index.ts
```

Not every feature needs every subfolder. Create a subfolder only when that kind of code exists.

### `components`

Feature-specific UI components and page sections.

Use this for components that express feature behavior, such as billing tables, product cards, setup flows, account panels, or domain-specific empty states.

### `queries`

TanStack Query hooks and server mutations owned by the feature.

Use this for:

- Fetch hooks.
- Create/update/delete mutation hooks.
- Query option factories.
- Feature-specific cache invalidation logic.

Queries should call the shared API client, but the feature should own the hook names, query keys, payload types, and result types.

### `validation`

Zod schemas for forms, route search parameters, and feature input validation.

Export inferred TypeScript types from schemas when other files need the validated shape.

Example:

```ts
export const searchSchema = z.object({
  page: z.number().catch(1),
  pageSize: z.number().catch(10),
});

export type SearchParams = z.infer<typeof searchSchema>;
```

### `types`

Feature-owned TypeScript types.

Use this for API DTOs, UI model types, enum-like types, and feature-specific props that are reused across multiple files.

Do not place feature-only types in global `src/types`.

### `constants`

Feature-owned constants.

Use this for query keys, tab ids, modal action names, table column metadata, feature option lists, and feature-specific labels.

### `utils`

Pure helpers owned by the feature.

Use this for formatting, mapping, status interpretation, payload creation, and small business rules used by the feature.

Keep helpers pure when possible. If a helper needs React hooks or browser state, it probably belongs in `hooks` instead.

### `hooks`

Feature-owned React hooks.

Use this for state composition, derived feature behavior, and browser/runtime interactions that are not global enough for `src/hooks`.

### `modals`

Feature-owned dialogs, alerts, and modal workflows.

Use this when a modal is part of a feature workflow, such as editing a billing profile, creating a product instance, changing a payment method, or confirming a destructive action.

### `index.ts`

Use `index.ts` as a curated public export surface for the feature.

Export only what outside folders should consume. Avoid exporting every internal file automatically.

Good exports:

- Route-level feature components.
- Public schemas needed by routes.
- Public types used by other features.

Avoid exporting:

- Internal table rows.
- Modal internals.
- Private helper functions.
- Component parts used only inside the feature.

## `src/hooks`

Use `src/hooks` for app-wide hooks that are not owned by one feature.

Examples:

- Debounce hooks.
- Current profile hooks.
- Browser capability hooks.
- Shared permission hooks.

If a hook knows about a specific feature domain, keep it in that feature.

## `src/lib`

Use `src/lib` for third-party integration setup and runtime configuration.

Examples:

- Auth provider configuration.
- Stripe initialization.
- Analytics setup.
- Runtime environment normalization.
- SDK adapters.

This folder should isolate integration details so feature code does not need to know how a provider is initialized.

Do not read or print environment files while inspecting or editing a project. Reference environment variable names only when needed.

## `src/routes`

Use `src/routes` for file-based route definitions.

Route files should stay thin. They should wire routing concerns and delegate business UI to feature modules.

Route files may contain:

- Route declaration.
- Search validation.
- Loaders and guards.
- Layout selection.
- Route-level metadata.
- Delegation to feature components.

Route files should not grow into full pages with deeply nested feature logic.

Preferred pattern:

```tsx
export const Route = createFileRoute('/_auth/billing/')({
  component: RouteComponent,
  validateSearch: billingSearchSchema,
});

function RouteComponent() {
  return (
    <PageWrapper>
      <BillingPage />
    </PageWrapper>
  );
}
```

## `src/theme`

Use `src/theme` for design-system setup.

This can include:

- Chakra UI theme configuration.
- Semantic tokens.
- Component recipes.
- Color mode settings.
- Shared design primitives required by the provider.

Use theme tokens in components instead of hardcoded colors when the project has a theme.

## `src/types`

Use `src/types` for shared application types.

Good candidates:

- Generic API result types.
- Global enum-like types.
- Shared pagination types.
- Cross-feature DTOs.
- Common utility types.

Do not use this folder for types that belong to only one feature.

## `src/utils`

Use `src/utils` for shared pure helper functions.

Good candidates:

- Date formatting.
- Money formatting.
- Input normalization.
- Shared validation message helpers.
- Generic option builders.

If a helper contains business language specific to one feature, keep it inside that feature.

## Routing Rules

Use TanStack Router conventions when the project uses TanStack Router.

- Put route files under `src/routes`.
- Use route groups such as `_auth` for authenticated sections when the project uses them.
- Use `validateSearch` for typed URL search parameters.
- Read and update route search state through route APIs instead of manual URL parsing.
- Treat generated route tree files as generated output.

Generated router files should not be hand-edited unless the project explicitly requires it.

## Data Fetching Rules

Use TanStack Query for server state when the project uses it.

- Put feature-specific query hooks under `src/features/{feature}/queries`.
- Keep query keys in feature constants unless shared globally.
- Use the shared API client from `src/api`.
- Return unwrapped successful data from query functions when the project has API result wrappers.
- Keep cache invalidation close to mutations.
- Do not fetch server state directly inside deeply nested presentational components if a feature-level component can own the query.

## Form And Validation Rules

Use Zod schemas when the project uses Zod.

- Put feature form schemas in `src/features/{feature}/validation`.
- Put route search schemas near the feature if the route delegates to that feature.
- Export inferred types from schemas when forms, routes, or payload mappers need them.
- Keep API payload mapping in feature utilities or modal-specific utilities.
- Reuse shared form field components before creating one-off form controls.

## UI Rules

Use the existing component system before creating new primitives.

- Prefer shared components from `src/components`.
- Prefer feature components from the owning feature.
- Use the existing design system and theme tokens.
- Keep layout shells separate from feature content.
- Keep reusable tables, inputs, selects, pagination, modals, and alert patterns consistent.

When adding a new component, first decide ownership:

- Shared across app: `src/components/{ComponentName}`.
- Specific to one feature: `src/features/{feature}/components`.
- Specific to one modal workflow: keep it inside that modal folder.
- Specific to one route only: keep it near the route only if it is small; otherwise move it into the owning feature.

## Import Rules

Prefer the configured source alias, commonly `@/*`, for imports from `src`.

Use relative imports inside a small local folder when it improves readability, such as importing sibling modal components or helper files.

Avoid long fragile relative imports that climb many folder levels.

## Localization Rules

When the project uses i18next or locale JSON files:

- Put user-facing copy in locale files.
- Add new keys to all supported locale files unless the task explicitly limits scope.
- Use existing translation helper conventions.
- Keep validation messages translation-aware.

Avoid hardcoded user-facing strings in feature components unless existing code clearly does so.

## Asset Rules

- Put SVG icons in `src/assets/icons`.
- Put raster images in `src/assets/images`.
- Use existing icon wrapper components when available.
- Avoid importing raw assets directly from many feature files if the project has a shared icon system.

## Environment And Secrets

- Do not read, print, or modify `.env*` files unless the user explicitly asks.
- Do not copy environment values into documentation or code comments.
- Refer to environment variable names only when necessary.
- Keep runtime config access centralized in `src/lib` or the existing config module.

## Verification

Run the smallest relevant check after changes.

Common checks:

```text
npm run lint
npm run build
npm run build-ts
tsc --noEmit
```

Use the commands already defined in `package.json`. Do not invent verification commands when the project already provides scripts.

## Placement Checklist

Before adding a file, decide:

- Is this code shared across multiple features? Use a global folder.
- Is this code owned by one business domain? Use `src/features/{feature}`.
- Is this code only route wiring? Use `src/routes`.
- Is this code provider setup or runtime config? Use `src/lib`.
- Is this code static media or translation data? Use `src/assets`.
- Is this generated? Do not hand-edit it.

This checklist matters more than making imports shorter.
