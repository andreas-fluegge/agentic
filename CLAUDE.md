# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

| Command | Description |
|---|---|
| `npm start` | Start the Angular dev server at `http://localhost:4200` |
| `npm run build` | Production build |
| `npm test` | Run unit tests (Karma) |
| `npx playwright test` | Run all E2E tests (requires dev server running) |
| `npx playwright test --project=chromium --reporter line` | Run E2E tests in Chrome with compact output |
| `npx playwright test tests/book-list.spec.ts` | Run a single spec file |
| `npx bookmonkey-api` | Start the backend API at `http://localhost:4730` |
| `npm run format.write` | Format source files with Prettier |
| `npm run build.angular-mcp` | Build the custom Angular MCP server tool |

## Architecture

This is an **Angular 21** (next channel) book management application using standalone components, signals, and TailwindCSS v4.

### API
The app talks to `http://localhost:4730/books` (BookMonkey API). The `BookApiClient` service (`src/app/books/core/book-api-client.service.ts`) wraps all HTTP calls. The API is **not part of this repo** — run `npx bookmonkey-api` separately.

### Feature structure

```
src/app/
  books/
    core/           # Domain model (Book, BookStatus) and API client
    display/        # Presentational components (BookItem, BookCover)
    highlights/     # Carousel/highlight feature with BookHighlightService
    kanban/         # Kanban board view (KanbanBoard, KanbanColumn, KanbanService)
    management/     # CRUD views: BookList, BookDetail, BookCreate, BookEdit
    shared/         # Reusable UI: ErrorMessage, LoadingIndication
  shared/           # App-wide shared: ToastService
```

Routes are defined in `app.routes.ts`: `/` → BookList, `/book/create`, `/book/:id`, `/book/:id/edit`, `/kanban`.

### State management
Local component state uses Angular signals (`signal()`, `computed()`). There is no NgRx store in the current codebase. `@ngrx/signals` and `@tanstack/angular-query-experimental` are available as dependencies but not yet wired up.

### MCP tools
`.claude/mcp.json` registers the Angular CLI as an MCP server (`@angular/cli mcp`). A custom local MCP tool lives in `tools/angular-mcp/` and exposes `generate_component` and `migrate-self-closing-tags` — build it with `npm run build.angular-mcp`.

## Angular coding conventions

These rules are always applied (`.claude/rules/cursor.mdc`):

- **Standalone components only** — never add `standalone: true` (it is the default in Angular 19+)
- **`ChangeDetectionStrategy.OnPush`** on every `@Component`
- **`input()` / `output()`** functions instead of `@Input` / `@Output` decorators
- **`inject()`** instead of constructor injection
- **Native control flow** (`@if`, `@for`, `@switch`) — never `*ngIf`, `*ngFor`, `*ngSwitch`
- **`[class]` bindings** — never `ngClass` or `ngStyle`
- **`host` object** in `@Component`/`@Directive` — never `@HostBinding` / `@HostListener`
- `computed()` for derived state; no `mutate()` on signals — use `update()` or `set()`
- `providedIn: 'root'` for singleton services
- Prefer `toSignal()` (from `@angular/core/rxjs-interop`) to convert Observables to signals; avoid `async` pipe

## Signal forms (experimental)

When working with forms, the project uses the experimental `@angular/forms/signals` API (not the stable `ReactiveFormsModule`). Key points:

- Import from `@angular/forms/signals`: `form`, `Control`, `required`, `minLength`, `maxLength`, `min`, `pattern`
- Create with `form(writableSignal)` — form state is read via `formInstance().value()`
- Replace `formControlName` / `ngModel` with `[control]="bookForm.<field>"`
- Remove `formGroup` / `ngForm` directives entirely
- Validation is configured as the second argument to `form()`, not via `Validators`
- All model properties bound to `[control]` must be **non-optional** (no `?`) — this is a current limitation

## E2E tests (Playwright)

- Tests live in `tests/**/*.spec.ts`
- Use Gherkin-style test names: `Given ... When ... Then ...`
- Prefer `data-testid` selectors; add them to templates when missing
- Use `getByRole` / `getByLabelText` as fallbacks — never CSS class selectors
- The dev server auto-starts via `webServer` in `playwright.config.ts`; use `reuseExistingServer` locally

## Refactoring guidelines (`.claude/rules/refactoring.mdc`)

- One module per file (one `class`, `interface`, `type`, or `function` per file)
- Prefer `[routerLink]` + `<a>` over `(click)` + `<button>` for navigation
- Use `@let` in templates to cache signals read more than once
- Use `@if(value(); as v)` instead of the safe-navigation operator (`?.`)
- Global HTTP error handling is in place — do not add inline `catchError` for generic errors
