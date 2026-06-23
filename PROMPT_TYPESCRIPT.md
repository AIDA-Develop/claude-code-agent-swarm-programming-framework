# TypeScript-Specific Claude Code Prompt

Append this prompt after `PROMPT_UNIVERSAL.md` for all TypeScript projects. These rules supplement -- not replace -- the universal workflow.

---

## TypeScript Build Commands

Use these commands throughout the workflow:

| Command | Purpose | When to run |
|---------|---------|-------------|
| `npm init -y` | Initialize project | Project initialization |
| `npm install --save-dev typescript tsx @types/node` | Core TypeScript tooling | Project setup |
| `npm install` | Install dependencies | After package.json changes |
| `npm test` | Run test suite | After implementation, must pass |
| `npm run build` | Compile TypeScript to JavaScript | Before deployment |
| `npm run lint` | Run ESLint | Before every commit |
| `npm run format` | Run Prettier | Before every commit |
| `npm run typecheck` | Type check without emit | Quick feedback loop |
| `npm run dev` / `npm start` | Run in development mode | Manual verification |
| `npx tsc --init` | Generate tsconfig.json | Project initialization |

---

## TypeScript Configuration (tsconfig.json)

Use this strict configuration. No exceptions without justification:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "lib": ["ES2022"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictBindCallApply": true,
    "strictPropertyInitialization": true,
    "noImplicitThis": true,
    "alwaysStrict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "resolveJsonModule": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "**/*.test.ts", "**/*.spec.ts"]
}
```

### Configuration Rules
- **`strict: true`** is non-negotiable. Every TypeScript project uses full strict mode.
- **`noImplicitAny: true`** -- no implicit `any` types anywhere. Explicit `any` must be justified.
- **`noUnusedLocals` / `noUnusedParameters`** -- clean code, no dead variables.
- Target `ES2022` for Node.js 18+. Adjust only if targeting older environments.
- Use `NodeNext` for module resolution in modern Node.js projects.

---

## TypeScript Code Rules

### Type Safety
- **Strict TypeScript only.** No `// @ts-ignore` without justification. No `// @ts-nocheck`.
- Explicit `any` is allowed only for (1) third-party untyped modules, (2) gradual migration boundaries. Document each usage.
- Use `unknown` over `any` for values of uncertain type. Narrow with type guards.
- Enable `no-explicit-any` in ESLint. If you must disable it for a line, explain why in a comment.

### Types and Interfaces
- Use `interface` for object shapes that may be extended. Use `type` for unions, tuples, mapped types, and aliases.
- Export types for all public API inputs and outputs.
- Use branded types (`type UserId = string & { readonly __brand: unique symbol }`) for type-safe IDs.
- Use `readonly` arrays and properties where mutation is not intended.
- Use `satisfies` operator (TypeScript 4.9+) for inline validation of object shapes.

### Runtime Validation
- **All external input must be validated at runtime.** TypeScript types disappear at runtime.
- Use `zod` for schema validation of API payloads, user input, file contents, environment variables.
- Define schemas alongside types and infer TypeScript types from schemas:
  ```typescript
  import { z } from 'zod';
  const UserSchema = z.object({ id: z.string(), name: z.string().min(1) });
  type User = z.infer<typeof UserSchema>;
  ```
- Parse, do not cast: `UserSchema.parse(data)` not `data as User`.
- Use `zod` for environment variable validation with `z.coerce` types.

### Functions and Architecture
- Prefer functional programming patterns: pure functions, immutable data, explicit dependencies.
- Avoid unnecessary classes. Use functions and closures. Use classes only for true OOP patterns (polymorphism, encapsulation of complex state).
- Keep functions small and single-purpose. Under 30 lines where practical.
- Use named exports over default exports. Default exports make refactoring harder.
- Use early returns to reduce nesting. Flatten promise chains with async/await.

### Async and Error Handling
- Use `async/await` for all asynchronous code. No raw `.then()` chains except for `Promise.all()`.
- Always `await` promises. No floating promises (enable `@typescript-eslint/no-floating-promises`).
- Wrap async operations in try/catch at appropriate boundaries.
- Use `Result<T, E>` patterns (via `neverthrow` or similar) for error-heavy code paths, or throw with typed errors.
- Rejections must be handled. Unhandled promise rejections crash the process.

### Code Organization
- Separate concerns: business logic, I/O, presentation, and infrastructure in different files/directories.
- For full-stack: `src/shared/`, `src/server/`, `src/client/` -- no cross-imports between client and server.
- For libraries: flat structure near the root, `src/index.ts` as the public API.
- Co-locate tests with source (`*.test.ts` alongside `*.ts`) or in a dedicated `tests/` directory. Be consistent.

### Dependencies
- Every dependency must be justified in the Architecture Plan.
- Prefer standard library and built-in Node.js APIs over packages.
- Evaluate package size, download count, maintenance status, and security history before adding.
- Pin versions in `package.json`. Use `package-lock.json`. No `*` or `latest` ranges.
- Audit with `npm audit` before final delivery.

---

## TypeScript Testing

### Test Framework
- Use **Vitest** for new projects (fast, native TypeScript, modern API).
- Use **Jest** with `ts-jest` only if the project already uses it.
- Configure in `vitest.config.ts`:
  ```typescript
  import { defineConfig } from 'vitest/config';
  export default defineConfig({
    test: { globals: true, environment: 'node' }
  });
  ```

### Test Practices
- Test file naming: `*.test.ts` or `*.spec.ts`. Be consistent.
- Unit test every exported function. Integration test every API endpoint or workflow.
- Use descriptive test names: `it('rejects negative deposit amounts', ...)` not `it('test 3', ...)`.
- Use `describe` blocks to group related tests.
- Mock external I/O (HTTP calls, file system, database) in unit tests. Use `vitest` mocks or `msw` for HTTP.
- Test error cases explicitly. Every error path must be exercised.
- Use explicit test data. No shared mutable fixtures between tests.

### Test Commands
```bash
npm test              # Run all tests
npm test -- --run    # Run once (no watch)
npm test -- --coverage  # With coverage report
```

---

## ESLint and Prettier Configuration

### ESLint (.eslintrc.cjs or eslint.config.mjs)

```javascript
// eslint.config.mjs
import tseslint from 'typescript-eslint';

export default tseslint.config(
  tseslint.configs.strictTypeChecked,
  tseslint.configs.stylisticTypeChecked,
  {
    languageOptions: {
      parserOptions: { project: true, tsconfigDirName: import.meta.dirname }
    },
    rules: {
      '@typescript-eslint/no-explicit-any': 'error',
      '@typescript-eslint/no-floating-promises': 'error',
      '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
      '@typescript-eslint/prefer-readonly': 'error',
    }
  }
);
```

### Prettier (.prettierrc)

```json
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 100,
  "arrowParens": "always"
}
```

### Quality Commands

```bash
npm run lint          # ESLint check
npm run lint -- --fix  # Auto-fix ESLint issues
npm run format        # Prettier check
npm run format -- --write  # Auto-format
npm run typecheck     # tsc --noEmit
```

All must pass before delivery.

---

## TypeScript Security

- **No secrets in source code.** Use environment variables validated with `zod`.
- Validate all external input with `zod` schemas before processing.
- Sanitize data before DOM insertion (if frontend). Use textContent over innerHTML.
- No `eval()`, `new Function()`, or dynamic code execution.
- Use parameterized queries for database access (no string concatenation).
- Escape user data in logging. No log injection.
- Run `npm audit` before final delivery. Address or document all findings.
- Use `helmet` for Express servers. Enable CORS only with explicit allowlists.
- Set secure HTTP headers. Use HTTPS in production.
- Validate and sanitize file uploads if applicable.

---

## TypeScript Project Structure Template

Use this structure unless the project demands otherwise:

```
project_name/
  package.json
  tsconfig.json
  tsconfig.build.json     -- Separate config for builds (excludes tests)
  vitest.config.ts        -- or jest.config.js
  eslint.config.mjs       -- ESLint flat config
  .prettierrc
  .gitignore
  src/
    index.ts              -- Public API exports
    types.ts              -- Shared domain types and Zod schemas
    config.ts             -- Environment configuration (validated with zod)
    errors.ts             -- Custom error classes
    utils/                -- Pure utility functions
      validation.ts
      formatting.ts
    core/                 -- Business logic (pure functions)
      domain_logic.ts
    services/             -- I/O and external integration
      api_client.ts
      database.ts
    server/               -- HTTP server (if applicable)
      app.ts
      routes.ts
      middleware.ts
  tests/                  -- or co-located *.test.ts
    unit/
    integration/
  examples/
    basic_usage.ts
  README.md
  LICENSE
```

---

## TypeScript Review Checklist

Before marking a TypeScript project complete, verify every item:

### Type Safety
- [ ] `strict: true` in tsconfig, no exceptions
- [ ] `npm run typecheck` passes (tsc --noEmit)
- [ ] No implicit `any` types
- [ ] No `@ts-ignore` or justified with comments
- [ ] External input validated with `zod` at every boundary
- [ ] No type assertions (`as`) without justification

### Code Quality
- [ ] `npm run lint` passes cleanly
- [ ] `npm run format` produces no changes
- [ ] No unused variables or imports
- [ ] Functions are small and focused
- [ ] No unnecessary classes
- [ ] Named exports preferred over default exports
- [ ] `readonly` used where appropriate

### Testing
- [ ] `npm test` passes (all unit and integration tests)
- [ ] Every exported function has at least one test
- [ ] Error paths tested
- [ ] Edge cases tested (empty input, null, undefined, boundary values)
- [ ] No floating promises
- [ ] Async errors handled

### Security
- [ ] `npm audit` passes or findings documented
- [ ] No secrets in source code
- [ ] Environment variables validated
- [ ] External input sanitized
- [ ] No eval() or dynamic execution

### Documentation
- [ ] README includes: what it does, how to build, how to test, how to run
- [ ] Examples compile and run (`npx tsx examples/basic_usage.ts`)
- [ ] Public API types exported and documented
- [ ] Environment variables documented

---

## TypeScript-Specific Decision Guide

| Decision | Prefer | Avoid |
|----------|--------|-------|
| Validation | `zod` | Manual validation, `as` casting |
| Testing | `vitest` | Unmaintained test frameworks |
| HTTP server | Fastify or native `http` | Heavy frameworks for simple APIs |
| HTTP client | native `fetch` | Heavy client libraries |
| Logging | `pino` (structured) | `console.log` in production |
| Config | `zod` schemas for env vars | Raw `process.env` access |
| CLI args | `cac` or `commander` | Manual argv parsing |
| Result types | `neverthrow` | Untyped throws everywhere |
| Path mapping | `tsconfig` paths | Deep relative imports (`../../../`) |
| Build | `tsc` or `tsx` | Heavy bundlers for backend code |
