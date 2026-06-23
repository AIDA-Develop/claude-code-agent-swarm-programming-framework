# TypeScript Best Practices for Claude Code Agent Swarm

> One-sentence rule: **Let the type system enforce your contracts; validate every external boundary.**

---

## Philosophy

- **Strict types** — `strict: true` is non-negotiable. Every value has a known shape.
- **Explicit contracts** — interfaces and types are documentation that the compiler enforces.
- **Functional preference** — prefer pure functions and immutable data. Use classes only when they model a real entity or encapsulate state.

---

## Project Setup

### Initialize a New Project

```bash
npm init -y
npm install typescript tsx
npm install -D vitest eslint prettier
npx tsc --init
```

### Required `tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "noImplicitReturns": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "**/*.test.ts"]
}
```

### Required `package.json` Scripts

```json
{
  "scripts": {
    "build": "tsc",
    "test": "vitest run",
    "test:watch": "vitest",
    "lint": "eslint src/",
    "format": "prettier --write src/",
    "format:check": "prettier --check src/",
    "typecheck": "tsc --noEmit"
  }
}
```

### Pre-Commit Checklist

```bash
npm run format:check && npm run lint && npm run typecheck && npm test
```

---

## Required Commands

| Command | Purpose |
|---------|---------|
| `npm init -y` | Initialize a new Node.js project |
| `npm install typescript tsx` | Install TypeScript runtime and executor |
| `npm install -D vitest eslint prettier` | Install dev dependencies for testing, linting, formatting |
| `npx tsc --init` | Generate a default `tsconfig.json` |
| `npm test` | Run all tests via Vitest |
| `npm run build` | Compile TypeScript to JavaScript |
| `npm run lint` | Run ESLint on source code |
| `npm run format` | Auto-format with Prettier |

---

## Code Style Rules

### Strict TypeScript

- `strict: true` and `noImplicitAny: true` are mandatory. No exceptions.
- Prefer `interface` for object shapes; use `type` for unions, tuples, and mapped types.
- Export types for all public function signatures.

### Interfaces and Types

```typescript
// Good — interface for object shape
interface User {
  readonly id: string;
  name: string;
  email: string;
  role: "admin" | "user" | "guest";
}

// Good — type for unions
type Result<T> = { ok: true; value: T } | { ok: false; error: string };

// Bad — using `any`
function process(data: any): any { ... }
```

### Runtime Validation for External Input

- **Never trust external data** — HTTP bodies, file contents, environment variables, CLI args.
- **Prefer `zod`** for schema validation:

```typescript
import { z } from "zod";

const UserSchema = z.object({
  id: z.string().uuid(),
  name: z.string().min(1),
  email: z.string().email(),
  role: z.enum(["admin", "user", "guest"]),
});

export type User = z.infer<typeof UserSchema>;

function parseUser(input: unknown): User {
  return UserSchema.parse(input); // throws on failure
}

function safeParseUser(input: unknown): User | null {
  const result = UserSchema.safeParse(input);
  return result.success ? result.data : null;
}
```

### Functional vs. Class-Based Code

- **Prefer functional modules** — top-level exported functions, pure where possible.
- **Avoid unnecessary classes.** Use a class only when:
  - It encapsulates meaningful state with invariants.
  - It implements an interface or abstraction.
  - It is required by a framework (e.g., NestJS controller).

```typescript
// Good — functional
export function calculateTotal(items: LineItem[]): number {
  return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
}

// Acceptable — class with real state
export class RateLimiter {
  private tokens: number;
  private lastRefill: number;

  constructor(private maxTokens: number, private refillRateMs: number) {
    this.tokens = maxTokens;
    this.lastRefill = Date.now();
  }

  tryAcquire(): boolean {
    this.refill();
    if (this.tokens > 0) {
      this.tokens--;
      return true;
    }
    return false;
  }

  private refill(): void { ... }
}
```

### Async/Await

- **Always `await` or `.catch()` promises.** Never fire-and-forget.
- **Use `Promise.all()` for independent async operations.**
- **Handle errors at the boundary:**

```typescript
// Good — handled
async function fetchUser(id: string): Promise<User | null> {
  try {
    const response = await fetch(`/api/users/${id}`);
    if (!response.ok) return null;
    const data = await response.json();
    return safeParseUser(data);
  } catch {
    return null;
  }
}

// Bad — fire and forget
fetchUser(id); // promise may reject unhandled
```

### Error Handling

- Use the `Result<T, E>` pattern or throw only for truly exceptional cases.
- Never swallow errors silently.
- Log errors with context before propagating.

### Package Structure

Separate concerns clearly:

```
src/
├── index.ts           # public API exports
├── types.ts           # shared domain types
├── utils.ts           # pure helper functions
├── validation.ts      # zod schemas
├── services/          # business logic
│   ├── userService.ts
│   └── emailService.ts
├── adapters/          # external integrations (DB, HTTP, filesystem)
│   ├── db.ts
│   └── httpClient.ts
└── config.ts          # environment config with validation
```

For full-stack projects:

```
packages/
├── shared/            # types, validation schemas, utilities
├── frontend/          # React/Vue/whatever
└── backend/           # API server
```

### Dependency Management

- **Avoid dependency bloat** — prefer standard library and small, focused packages.
- Audit regularly:

```bash
npm audit
npm outdated
```

- Pin versions in `package-lock.json`. Review before adding any new dependency.

---

## Testing with Vitest

### Configuration

```typescript
// vitest.config.ts
import { defineConfig } from "vitest/config";

export default defineConfig({
  test: {
    globals: true,
    environment: "node",
    coverage: {
      reporter: ["text", "html"],
      exclude: ["node_modules/", "dist/"],
    },
  },
});
```

### Test Patterns

```typescript
import { describe, it, expect } from "vitest";
import { calculateTotal, type LineItem } from "./pricing.js";

describe("calculateTotal", () => {
  it("sums line items correctly", () => {
    const items: LineItem[] = [
      { price: 10, quantity: 2 },
      { price: 5, quantity: 1 },
    ];
    expect(calculateTotal(items)).toBe(25);
  });

  it("returns 0 for empty cart", () => {
    expect(calculateTotal([])).toBe(0);
  });
});
```

---

## ESLint and Prettier Configuration

### `.eslintrc.json`

```json
{
  "env": { "es2022": true, "node": true },
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended",
    "plugin:@typescript-eslint/recommended-requiring-type-checking"
  ],
  "parser": "@typescript-eslint/parser",
  "parserOptions": {
    "project": "./tsconfig.json"
  },
  "plugins": ["@typescript-eslint"],
  "rules": {
    "@typescript-eslint/no-unused-vars": ["error", { "argsIgnorePattern": "^_" }],
    "@typescript-eslint/no-explicit-any": "error",
    "@typescript-eslint/explicit-function-return-type": "warn",
    "@typescript-eslint/no-floating-promises": "error"
  }
}
```

### `.prettierrc`

```json
{
  "semi": true,
  "trailingComma": "all",
  "singleQuote": false,
  "printWidth": 100,
  "tabWidth": 2
}
```

---

## Security Rules

1. **Validate all external input** with `zod` or equivalent.
2. **Never trust `as` casts** — they bypass the type system. Use type guards instead.
3. **No secrets in source code** — use environment variables, validated at startup.
4. **Sanitize data before DOM insertion** — prevent XSS even in typed code.
5. **Run `npm audit`** in CI to catch known vulnerabilities.
6. **Use least-privilege dependencies** — prefer smaller, audited packages.

---

## Project Structure Template

```
my-ts-project/
├── package.json
├── tsconfig.json
├── vitest.config.ts
├── .eslintrc.json
├── .prettierrc
├── .gitignore
├── src/
│   ├── index.ts
│   ├── types.ts
│   ├── validation.ts
│   ├── config.ts
│   ├── utils.ts
│   ├── services/
│   │   └── userService.ts
│   └── adapters/
│       └── db.ts
├── tests/
│   └── userService.test.ts
└── dist/              # compiled output (gitignored)
```

---

## Common Mistakes to Avoid

| Mistake | Why It's Wrong | Fix |
|---------|---------------|-----|
| `any` type | Defeats the purpose of TypeScript | Use `unknown` + validation or proper types |
| `as` casts to silence errors | Hides real bugs | Fix the underlying type issue |
| Fire-and-forget promises | Unhandled rejections crash the process | Always `await` or `.catch()` |
| No runtime validation | Type safety ends at the runtime boundary | Use `zod` for all external data |
| Classes for everything | Unnecessary complexity | Use functions and plain objects |
| Mutable shared state | Hard to reason about and test | Use immutable data and pure functions |
| Deeply nested callbacks | Callback hell | Use async/await |
| Ignoring `noImplicitAny` | Silent loss of type safety | Enable `strict: true` |
| Mixing frontend/backend in one package | Unclear boundaries | Separate packages or directories |
| Skipping tests for "simple" code | Bugs accumulate | Test all logic branches |

---

## Example: Minimal Correct TypeScript Project

```
greet/
├── package.json
├── tsconfig.json
├── vitest.config.ts
├── .eslintrc.json
├── .prettierrc
└── src/
    ├── index.ts
    ├── greet.ts
    └── greet.test.ts
```

**`package.json`**

```json
{
  "name": "greet",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "build": "tsc",
    "test": "vitest run",
    "lint": "eslint src/",
    "format": "prettier --write src/",
    "typecheck": "tsc --noEmit"
  },
  "dependencies": {
    "typescript": "^5.4.0"
  },
  "devDependencies": {
    "vitest": "^1.0.0",
    "eslint": "^8.0.0",
    "prettier": "^3.0.0",
    "@typescript-eslint/eslint-plugin": "^7.0.0",
    "@typescript-eslint/parser": "^7.0.0"
  }
}
```

**`tsconfig.json`**

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "noImplicitAny": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "declaration": true
  },
  "include": ["src/**/*"]
}
```

**`src/greet.ts`**

```typescript
import { z } from "zod";

const GreetInputSchema = z.object({
  name: z.string().min(1),
  greeting: z.string().optional(),
});

export type GreetInput = z.infer<typeof GreetInputSchema>;

/**
 * Generates a greeting message.
 */
export function greet(input: unknown): string {
  const parsed = GreetInputSchema.parse(input);
  const greeting = parsed.greeting ?? "Hello";
  return `${greeting}, ${parsed.name}!`;
}
```

**`src/greet.test.ts`**

```typescript
import { describe, it, expect } from "vitest";
import { greet } from "./greet.js";

describe("greet", () => {
  it("returns default greeting", () => {
    expect(greet({ name: "Ada" })).toBe("Hello, Ada!");
  });

  it("returns custom greeting", () => {
    expect(greet({ name: "Ada", greeting: "Hi" })).toBe("Hi, Ada!");
  });

  it("throws on empty name", () => {
    expect(() => greet({ name: "" })).toThrow();
  });
});
```

**`src/index.ts`**

```typescript
export { greet, type GreetInput } from "./greet.js";
```

**Run the example:**

```bash
npm install
npm run typecheck
npm run lint
npm test
```

---

## Quick Reference

```bash
# Pre-commit check
npm run format:check && npm run lint && npm run typecheck && npm test

# Run tests in watch mode
npm run test:watch

# Check types without emitting
npx tsc --noEmit

# Run a single test file
npx vitest run src/greet.test.ts

# Audit dependencies
npm audit
```
