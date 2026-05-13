# FiduciaryPlan — Manual Test Plan

Run these checks before each release. Mark ✅ pass / ❌ fail / ⚠ partial.

---

## 1. Environment Setup

- [ ] Open `index.html` via `python3 -m http.server 8080` → `http://localhost:8080`
- [ ] Open browser DevTools → Console — no errors on load
- [ ] Confirm no external requests in Network tab on page load (only CDN scripts)

---

## 2. Landing Screen

- [ ] Landing screen displays correctly with title, subtitle, and both buttons
- [ ] "Start fresh" button navigates to intake
- [ ] "Import saved plan (JSON)" opens file picker
- [ ] Footer disclaimer visible

---

## 3. Fresh User Flow — Standard Case

Persona: 35-year-old W-2 employee, married filing jointly, 2 kids, owns a home.

### Intake
- [ ] First question: name (optional, can be skipped)
- [ ] Age field accepts 35, rejects letters
- [ ] State dropdown shows all 50 states + DC
- [ ] Filing status shows 4 options; selecting "Married Filing Jointly" reveals spouse questions
- [ ] Dependents = 2: "College funding" appears in Life Events options
- [ ] Progress bar advances with each answer
- [ ] "← Back" button returns to previous question with prior answer pre-filled
- [ ] "Skip" appears on optional questions; required questions show error if submitted empty
- [ ] `ownOrRent = own` → real estate section (home value, mortgage) appears
- [ ] Life Events multi-select: can select multiple, can deselect
- [ ] Sliders: drag works; keyboard arrow keys work; value label updates in real time
- [ ] After final question, auto-navigates to Dashboard Tab 1

### LocalStorage persistence
- [ ] Refresh browser mid-intake → returns to last answered question with answers intact
- [ ] `localStorage.getItem('financePlan_v1')` in console returns valid JSON

---

## 4. Fresh User Flow — Edge Cases

### 4a. No income / no assets user
- [ ] All currency fields left at 0 or skipped
- [ ] Dashboard loads without errors
- [ ] Net worth shows $0
- [ ] Observations include emergency fund warning
- [ ] Retirement projection chart shows flat line (no contributions)
- [ ] Tax tab shows $0 tax, no division-by-zero errors

### 4b. Already retired user (age 68, no income)
- [ ] Age = 68; target retirement age slider starts ≤ 68
- [ ] Retirement tab: bridge period = 0 (already past SS age)
- [ ] Projection chart shows only drawdown phase
- [ ] No "years to retirement" shown as negative number

### 4c. Very young user (age 22, single, renting)
- [ ] Single filing status: spouse questions do not appear
- [ ] ownOrRent = rent: real estate section skipped / shown as optional
- [ ] Retirement projection extends 68 years (age 22 → 90)
- [ ] Chart renders correctly at that scale

### 4d. High-income user (income > $400k, married)
- [ ] Backdoor Roth eligibility shows as eligible
- [ ] Marginal rate shows 32% or 35%
- [ ] Roth conversion space calculation is non-negative
- [ ] State tax estimate shown (not $0 for CA/NY/etc.)

---

## 5. Dashboard — Tab 1: Snapshot

- [ ] Net worth card shows correct value: (k401 + IRA + brokerage + home) - (mortgage + debts)
- [ ] Savings rate = (k401 contrib + brokerage contrib) / gross income
- [ ] Emergency fund months = (checking + savings) / (gross / 12)
- [ ] Asset breakdown bar chart renders with correct categories
- [ ] Cash flow bar chart shows gross income and itemized deductions
- [ ] At least 1 observation generated when intake has real data
- [ ] Observation type colors: warning=amber, success=green, opportunity=indigo

**Spot-check calculation:**
> Input: $100k income, 6% 401k, checking $10k, savings $20k, gross $100k
> Expected: savings rate ~6%, emergency fund = 30k / (100k/12) = 3.6 months, federal tax ~$12k-15k

---

## 6. Dashboard — Tab 2: Portfolio Projection

- [ ] Chart renders with 3 area lines (conservative, base, optimistic)
- [ ] Red dashed "Retire" reference line at correct age
- [ ] Equity return slider (4–14%): adjusting updates chart immediately
- [ ] Inflation slider (1–8%): adjusting updates chart immediately
- [ ] Retirement age slider updates reference line
- [ ] Allocation donut chart shows correct stock/bond/cash split
- [ ] Assumptions table shows source footnotes

**Spot-check:** Age 30, retire 65, $50k in accounts, $100k income, 6% contrib, 9% return → ~$2M–4M at retirement (order of magnitude check)

---

## 7. Dashboard — Tab 3: Tax Optimization

- [ ] Gross income, federal tax, FICA, state tax all display
- [ ] Effective rate = total tax / gross income
- [ ] Marginal rate = correct bracket for taxable income after deductions
- [ ] 401k headroom = $23,500 - current contributions (shows $0 remaining if maxed)
- [ ] IRA and HSA headroom display correctly
- [ ] Roth conversion space = next bracket threshold - current taxable income
- [ ] Backdoor Roth: single >$161k or MFJ >$236k → shows "eligible"
- [ ] State tax note shows "no state income tax" for TX, FL, NV, etc.

---

## 8. Dashboard — Tab 4: Retirement Readiness

- [ ] Portfolio at retirement uses base projection at target retirement age
- [ ] Portfolio income = balance × 0.04
- [ ] SS estimate = simplified PIA calculation (non-zero for non-zero income)
- [ ] Success rate: if portfolio income + SS ≥ desired spend → ≥ 85%
- [ ] Bridge period shown only when retire age < SS age (67)
- [ ] Projection chart shows area with SS and retire reference lines
- [ ] Monte Carlo: 5 percentile boxes display (p10, p25, p50, p75, p90)
- [ ] Spending adjustment slider updates success rate in real time
- [ ] Withdrawal sequence (taxable → deferred → Roth) displays

---

## 9. Dashboard — Tab 5: Life Events

- [ ] If no events selected: shows empty state message
- [ ] Events sorted by target year ascending
- [ ] Monthly savings needed = cost / yearsAway / 12
- [ ] Total cost and retirement impact displayed
- [ ] Priority badges render (must-have=red, want=amber, nice-to-have=green)

---

## 10. Dashboard — Tab 6: Estate

- [ ] Will status: "yes" → ✅, "outdated" → ⚠, "no" → ❌
- [ ] Life insurance gap = max(0, income × 10 - coverage)
- [ ] Net worth < $13.99M → "No federal estate tax"
- [ ] Net worth > $13.99M → shows taxable amount
- [ ] TCJA sunset note displayed

---

## 11. Dashboard — Tab 7: AI Advisor

### Without API key
- [ ] Shows "API Key Required" state with instructions

### With valid API key
- [ ] Quick prompt buttons appear
- [ ] Clicking a quick prompt sends message and shows loading spinner
- [ ] Response streams/appears correctly
- [ ] Response is in correct chat bubble (right=user, left=assistant)
- [ ] Error state shows if API key is wrong (clear error message)
- [ ] Second message in conversation works (history maintained)
- [ ] Plan JSON context is included (verify by asking "what is my income?")

### API key storage
- [ ] API key entered in Settings → persists after page refresh
- [ ] "Clear" button in Settings removes key from localStorage
- [ ] Key shown masked (last 4 chars visible) after save

---

## 12. Settings Panel

- [ ] Opens as right-side drawer
- [ ] API key save/clear works (see Tab 7 checks above)
- [ ] Assumptions editor: changing equity return saves and updates projections
- [ ] Export: produces valid JSON file with warning dialog first
- [ ] Import: loading exported JSON returns to correct screen
- [ ] Reset: requires confirmation; clears all data; returns to landing

---

## 13. Export / Import Round-Trip

- [ ] Complete intake fully
- [ ] Export plan → download `fiduciary-plan-YYYY-MM-DD.json`
- [ ] Open file in text editor: valid JSON, all fields present
- [ ] Reset app → landing screen
- [ ] Import the exported file → returns to dashboard with all data intact
- [ ] All tabs show same data as before export

---

## 14. Accessibility

- [ ] Tab through entire intake using keyboard only
- [ ] Sliders respond to arrow keys
- [ ] Select dropdowns work with keyboard
- [ ] All buttons have visible focus ring
- [ ] Dashboard tabs navigable by keyboard
- [ ] Charts have accessible Tooltip on hover

---

## 15. Mobile (320px – 768px width)

- [ ] Landing screen single-column layout
- [ ] Intake questions readable and inputs touch-friendly
- [ ] Progress bar visible
- [ ] Dashboard tab nav scrolls horizontally
- [ ] Stat cards stack in 2-column grid
- [ ] Charts are responsive (ResponsiveContainer)
- [ ] Settings panel full-width on mobile

---

## 16. Privacy / Security Checks

- [ ] Network tab: no requests on page load except CDN scripts
- [ ] Network tab: no requests when navigating between tabs
- [ ] Network tab: only `api.anthropic.com` request when AI advisor used
- [ ] No `console.log` output containing financial data (in production mode)
- [ ] `__DEV__` flag: `window.__PLAN__` only accessible on localhost
- [ ] Export dialog shows security warning before download

---

## 17. Performance

- [ ] Intake moves between questions in <100ms
- [ ] Dashboard tab switch renders in <300ms
- [ ] Retirement projection chart renders in <500ms
- [ ] Monte Carlo (500 runs) completes in <2s

---

## Regression Checklist (run after any change)

- [ ] Fresh intake still completes end-to-end
- [ ] localStorage save/load still works
- [ ] Net worth calculation correct (manual spot-check)
- [ ] Federal tax calculation correct for a known case
- [ ] Charts render without JS errors
- [ ] AI advisor still sends and receives messages
