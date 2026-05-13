# FiduciaryPlan — Personal Financial Planning App

A browser-based fiduciary financial planner that builds an interactive model from your answers. Nothing is stored anywhere but your browser.

## Quick Start

**Option 1 — Open directly:**
```
open index.html
```
(Works in Chrome, Safari, Firefox, Edge — latest versions only. Does not work with `file://` due to ES module CORS restrictions in some browsers.)

**Option 2 — Local HTTP server (recommended):**
```bash
python3 -m http.server 8080
# then open http://localhost:8080
```

**Option 3 — Any static host:** Drop `index.html` on Netlify Drop, GitHub Pages, or any static host. No build step required.

---

## Getting an Anthropic API Key

The AI Advisor tab requires a Claude API key:

1. Go to [console.anthropic.com](https://console.anthropic.com)
2. Create an account and add billing
3. Navigate to API Keys → Create Key
4. Copy the key (starts with `sk-ant-`)
5. In FiduciaryPlan, open **Settings** (top right) and paste your key

Your key is stored in `localStorage` only. It is never sent to any server other than `api.anthropic.com`. You can clear it at any time in Settings.

**Estimated cost:** A typical advisor session (~5-10 exchanges) costs well under $0.10 at current Claude Sonnet pricing.

---

## Privacy Model

| What | Where it lives | Who can see it |
|------|---------------|----------------|
| All financial data | `localStorage` in your browser | Only you, on this device |
| Anthropic API key | `localStorage` | Only you; transmitted only to api.anthropic.com |
| AI conversations | `localStorage` (session) | Only you |
| Exported JSON | Your file system | You control completely |

**No telemetry. No analytics. No tracking pixels. No external requests except to `api.anthropic.com` when you use the advisor.**

When you export a plan, the JSON contains all your financial data. Treat it like a tax document — store securely and do not share.

---

## Features

- **Conversational intake** — one question at a time, adaptive branching, progress saved after every answer
- **7-tab dashboard:**
  - **Snapshot** — net worth waterfall, cash flow, fiduciary observations
  - **Portfolio Projection** — 3-scenario area chart with sliders, Monte Carlo bands
  - **Tax Optimization** — marginal/effective rates, contribution headroom, Roth conversion modeler
  - **Retirement Readiness** — success probability, bridge period, withdrawal sequencing, Monte Carlo
  - **Life Events** — goal timeline with funding requirements
  - **Estate** — document checklist, insurance gap, estate tax exposure
  - **AI Advisor** — conversational Claude with full plan context, streaming responses
- **Export/Import** — full plan portability via JSON
- **Editable assumptions** — every model input visible and overridable in Settings

---

## Known Prototype Limitations

1. **State tax calculations are simplified.** Flat-rate states are accurate; graduated states use a two-bracket approximation. Do not use for tax filing.
2. **Social Security estimate uses a simplified PIA formula.** For accuracy, use [ssa.gov/myaccount](https://ssa.gov/myaccount).
3. **No actual financial account connections.** Plaid, Fidelity, etc. integrations are stubs — all data is entered manually.
4. **401(k) contribution tracking** only covers the employee pre-tax amount; catch-up, after-tax, and mega backdoor Roth are flagged but not fully modeled.
5. **Single device only** — localStorage does not sync across devices. Use Export/Import to move between devices.
6. **No AMT modeling** — Alternative Minimum Tax can affect high-income households significantly; not included in prototype.
7. **Monte Carlo uses constant standard deviation (15%)** — does not model volatility clustering or fat tails.
8. **2026 tax brackets are estimates** — based on inflation adjustment from IRS Rev Proc 2024-40; verify against IRS.gov for official 2026 figures.
9. **No RMD modeling** — Required Minimum Distributions from traditional accounts are not computed.
10. **AI model is `claude-sonnet-4-5`** (configurable in Settings) — responses are educational, not licensed financial advice.

---

## Phase 2 Migration Plan — Next.js Production App

### What changes

| Concern | Phase 1 (Prototype) | Phase 2 (Production) |
|---------|--------------------|--------------------|
| Runtime | Browser, no build | Next.js 14 App Router, TypeScript |
| API calls | Browser → Anthropic directly | Server Route → Anthropic (key never in browser) |
| Storage | localStorage | Supabase (Postgres) with optional localStorage fallback |
| Auth | None | Supabase Auth (email magic link or OAuth) |
| State | React useState | Zustand or Jotai + SWR for server state |
| Styling | Tailwind CDN | Tailwind + shadcn/ui |
| Charts | Recharts CDN | Recharts (same API) |
| Schema validation | None | Zod (matches schema.json) |
| Deployment | Any static host | Vercel |

### Migration steps

1. **`npx create-next-app@latest fiduciaryplan --typescript --tailwind --app`**
2. **Move all calculation engine functions** from `index.html` script to `lib/calculations/` — they are pure functions with no browser dependencies, zero migration work.
3. **Move intake question definitions** to `lib/intake/questions.ts` — same logic, typed with Zod.
4. **Create `/app/api/advisor/route.ts`** — server-side Anthropic call; remove direct browser fetch. (Marked `// PROD: move to server route` in source.)
5. **Replace localStorage calls** with Supabase upsert. Add optional localStorage as offline cache. Wrap in a `usePlan()` hook.
6. **Port components** from htm template literals to TSX — mechanical find/replace.
7. **Add Zod validation** using `schema.json` as source of truth: `z.object({...})` matching each section.
8. **Shareable read-only links** — store plan ID in URL hash; Supabase RLS policy allows public read of `is_shared=true` rows.
9. **Add shadcn/ui** for improved component library: `Button`, `Dialog`, `Tabs`, `Slider`, `Card`.

### What stays identical

- All calculation engine logic (tax, projection, Monte Carlo, net worth)
- All intake question definitions and branching rules
- The data schema (schema.json)
- The advisor system prompt
- All UX flows and tab structure

### Scaffold target directory structure

```
fiduciaryplan/
├── app/
│   ├── layout.tsx
│   ├── page.tsx               # Landing → redirect to /intake or /dashboard
│   ├── intake/page.tsx
│   ├── dashboard/page.tsx
│   └── api/
│       └── advisor/route.ts   # PROD: server-side Anthropic call
├── lib/
│   ├── calculations/
│   │   ├── tax.ts
│   │   ├── retirement.ts
│   │   ├── netWorth.ts
│   │   └── monteCarlo.ts
│   ├── intake/
│   │   └── questions.ts
│   └── schema.ts              # Zod schema from schema.json
├── components/
│   ├── intake/
│   └── dashboard/
│       ├── Tab1Snapshot.tsx
│       ├── Tab2Portfolio.tsx
│       └── ...
├── hooks/
│   └── usePlan.ts             # localStorage or Supabase
└── supabase/
    └── migrations/
        └── 001_plans.sql
```
