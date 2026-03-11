# I Don't Just Ship Features. I Ship Systems That Think.
### A data-driven breakdown of how I designed, built, and deployed a production-grade full-stack AI app — every architectural decision explained, every tradeoff measured.

---

> **Stack:** React · TypeScript · Node.js · Express · TypeORM · PostgreSQL · Google Gemini 2.5 Flash · PWA · Render
>
> **Live app:** [budget-pro-nmmr.onrender.com](https://budget-pro-nmmr.onrender.com)

---

## Step 01 — Define the Problem Precisely

Before writing a line of code, I wrote a problem statement.

Most budgeting apps are glorified spreadsheet viewers. They give you data but zero insight. They tell you *what* happened — never *why*, and never *what to do about it*.

**What existing apps do wrong:**
- Log an expense. See it in a pie chart. Close the app.
- No context for whether spending is "good" or "bad"
- No ability to query your own financial data in natural language
- Expenses tracked by date range — breaks on overlapping periods

**What I set out to build:**
- AI with full access to live budget and expense data
- Stateful multi-turn conversation — follow-up questions just work
- Category-level granularity with real-time limit tracking
- Expenses stamped to budget by foreign key — not fragile date matching
- PWA: native install, offline-capable, zero app store friction

> **Key insight:** The problem wasn't "no budgeting app exists." The problem was "no budgeting app makes data *queryable* in natural language." That single distinction drove every architecture choice downstream.

---

## Step 02 — Design the Solution Layer by Layer

Every technology choice was made to serve the problem — not the resume.

| Layer | Technology | Why |
|---|---|---|
| Frontend | React + TypeScript + Vite + TanStack React Query v5 | React Query replaced the entire useEffect/useState fetch pattern. Caching, background refetch, mutation invalidation — all declarative. |
| Styling | TailwindCSS + Lucide React | Utility-first CSS eliminates style drift. Consistent spacing tokens. No bundle bloat. |
| Backend | Node.js + Express + TypeScript + TypeORM + Joi | TypeORM with `synchronize:true` for rapid schema iteration. Joi validation with a global error handler — no more silent 400s. |
| Database | PostgreSQL via Neon (Serverless) | Serverless Postgres removes infrastructure overhead. Relational model enforces `budgetId` FK integrity. |
| AI Layer | Google Gemini 2.5 Flash — `startChat()` | Each request includes full conversation history + current budget/expense snapshot. True multi-turn financial Q&A. |
| Delivery | PWA via vite-plugin-pwa + Render | Custom `usePWAInstall` hook captures `beforeinstallprompt`. Gracefully degrades on iOS Safari and already-installed state. |

---

## Step 03 — Architecture Decisions With the Tradeoffs

Good engineers don't just pick tools. They explain why — and what they gave up.

### 1. React Query over useEffect fetch loops

Four separate pages all called `/budget/list`. With raw fetch, that's 4 network requests on every mount. React Query deduplicates to 1, caches with a 3-minute stale time, and auto-invalidates when a mutation fires.

```js
// Before: 4x fetch, no cache, no sync
// After: 1x fetch, shared cache, mutation invalidation cascades to all
queryClient.invalidateQueries(['budgets'])
```

### 2. `budgetId` FK on expenses — not date range matching

Original design matched expenses to budgets by date range overlap. When periods overlapped, expenses bled across budgets incorrectly.

```sql
-- Bad: WHERE date BETWEEN budget.start AND budget.end
-- Edge case: overlapping periods = wrong assignment

-- Good: WHERE expense.budgetId = budget.id
-- Ownership is explicit and unambiguous
```

### 3. Personality-first system instruction for AI

My first prompt: *"Output ONLY markdown. Do not answer anything outside the data."*

Result: Fin refused to greet users or give financial advice. The fix was defining Fin's *capabilities and personality* first, then adding constraints. Same model — completely different product.

```
// Lesson: Restrictions without identity = a broken assistant, not a safe one.
// Define who it IS before what it can't do.
```

### 4. PORT env var collision on cloud deploy

Render sets `process.env.PORT=10000` for the web server. My TypeORM config was reading the same variable for the database port. Postgres tried to connect on port 10000.

```bash
# Before: DB_HOST, PORT (collision — DB used web server port)
# After:  DB_HOST, DB_PORT, PORT
# Rule:   namespace all env vars by service
```

### 5. Custom `usePWAInstall` hook

The native `beforeinstallprompt` event is ephemeral — you get one shot to capture it. Built a hook that caches the event in a ref, exposes an `install()` function, and tracks state so the install button gracefully hides when unavailable (iOS Safari, already-installed).

### 6. Global Joi error handler with structured logging

Validation failures were completely silent. Added a 4-argument Express error handler that logs the full Joi error detail and returns a structured 400 response.

```js
app.use((err, req, res, next) => {
  if (err.isJoi) console.error(err.details)
  res.status(400).json({ errors: err.details })
})
```

---

## Step 04 — Proof of Work

| Feature | Technical Challenge Solved | Outcome |
|---|---|---|
| Budget CRUD | TypeORM entities, FK constraints, JWT-scoped access control | Zero data leakage between users |
| Category Limits | Aggregated expense sum per category per budget period | Sub-50ms query on indexed FK |
| Period Cloning | Atomic transaction: clone budget → clone categories → shift dates | One-tap new period, no stale state |
| AI Assistant (Fin) | Full expense+budget context injected per request, history replayed | True follow-up awareness across turns |
| PWA Install | Custom hook captures `beforeinstallprompt`, stores in ref | Install button only shows when available |
| Data Caching | React Query staleTime=3min, invalidation on all mutations | Instant page transitions, zero redundant fetches |

---

## Step 05 — The Pitch

**I think in systems, not in features.**

When I built BudgetPro, every choice — React Query, FK relationships, context injection in AI, PWA hooks — came from a clear mental model of how data flows end-to-end. I don't just write code that works today. I write code that doesn't break in 3 months.

**I debug from first principles.**

PORT collision, silent Joi errors, stateless AI — I didn't Google "how to fix Render deployment." I traced the data flow, identified the assumption that broke, and fixed the root cause. Not the symptom.

**I ship to production, not just to GitHub.**

Live at: [budget-pro-nmmr.onrender.com](https://budget-pro-nmmr.onrender.com). Backend on Render. Frontend on Render static. DB on Neon Serverless. Auth, validation, error handling, caching — all production-grade.

**I own the full stack — and the product decisions.**

I didn't have a PM writing specs or a designer handing me Figma files. I defined the problem. I designed the solution. I built it. I deployed it. That's not a portfolio project. **That's a product.**

---

**Skills demonstrated in this single project:**

`React` `TypeScript` `React Query` `Vite` `TailwindCSS` `PWA` `Node.js` `Express` `TypeORM` `PostgreSQL` `JWT` `Joi` `Google Gemini API` `AI Prompt Engineering` `System Design` `Cloud Deployment` `Debugging Under Ambiguity`

---

*Open to full-stack, backend, or AI-integrated roles. If you're building systems that need engineers who think end-to-end — let's talk.*
