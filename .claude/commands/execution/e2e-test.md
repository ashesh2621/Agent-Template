---
description: Run comprehensive E2E browser testing with agent-browser CLI, DB validation, and responsive checks
argument-hint: (none)
---

# E2E Test: Comprehensive End-to-End Application Testing

Automate full end-to-end validation of this web application using the Vercel Agent Browser CLI. Tests every user journey, validates database state, and checks responsive layouts.

---

## Pre-flight Checks

### 1. Platform Validation

```bash
uname -s
```

**Supported platforms:** Linux, WSL, macOS only.

If the result is `MINGW`, `CYGWIN`, or `Windows_NT` — halt immediately with:
> "E2E testing is not supported on native Windows. Use WSL or a Linux/macOS environment."

### 2. Frontend Detection

Verify this project has a browser-accessible UI:

```bash
# Check for package.json with dev/start scripts
cat package.json | grep -E '"dev"|"start"'

# Check for frontend framework indicators
ls src/ || ls pages/ || ls app/ 2>/dev/null
```

If no frontend is detected — halt with:
> "No browser-accessible frontend detected. E2E testing requires a web UI."

### 3. agent-browser Setup

```bash
# Check if agent-browser is installed
agent-browser --version

# If not found, install it
npm install -g agent-browser

# Install browser dependencies (Chromium, etc.)
agent-browser install --with-deps
```

Confirm installation succeeded before proceeding.

---

## Phase 1: Parallel Research

Launch three sub-agents simultaneously to gather full context before testing.

### Sub-agent 1 — Application Structure

Research this codebase thoroughly. Return a structured summary covering:

1. How to start the application (exact dev command, port)
2. All authentication flows (login, signup, OAuth, logout)
3. Every user-facing route and what it renders
4. All complete user journeys from entry to completion
5. Every interactive UI element (forms, buttons, modals, dropdowns, toggles)

Output format:
```
START_COMMAND: npm run dev
PORT: 3000
AUTH_FLOWS: [list]
ROUTES: [list of path → purpose]
USER_JOURNEYS: [list of named journeys with steps]
INTERACTIVE_ELEMENTS: [list per page]
```

### Sub-agent 2 — Database Schema

Research the data layer:

1. Database type and connection env var (from `.env.example`)
2. Full schema: all tables, columns, types, relationships, foreign keys
3. For each user-facing action, identify which table(s) change and how
4. Write validation queries for each action (SELECT statements to confirm records exist)

Output format:
```
DB_TYPE: postgres | sqlite | supabase
CONNECTION_VAR: DATABASE_URL | SUPABASE_URL | etc.
SCHEMA: [table → columns]
VALIDATION_QUERIES: [action → SQL query]
```

### Sub-agent 3 — Bug Hunting

Analyze the codebase for issues before testing:

1. Logic errors (off-by-one, wrong conditionals, missing edge cases)
2. UI/UX issues (broken layouts, inaccessible elements, missing states)
3. Data integrity risks (missing validation, no error handling, race conditions)
4. Security concerns (unvalidated input, exposed tokens, IDOR risks)

Output format:
```
[severity: CRITICAL|HIGH|MEDIUM|LOW]
file: path/to/file.ts:line
issue: one-line description
detail: explanation
```

**Wait for all three sub-agents to complete before proceeding to Phase 2.**

---

## Phase 2: Start Application

Using the `START_COMMAND` and `PORT` from Sub-agent 1:

```bash
# Install dependencies if needed
npm install  # or pip install -r requirements.txt for Python

# Start dev server in background
npm run dev &
DEV_PID=$!

# Wait for server to be ready (max 30s)
sleep 5
```

Verify server is reachable with agent-browser:

```bash
agent-browser open http://localhost:3000
agent-browser screenshot screenshots/00_initial_load.png
```

If the server fails to start or the page doesn't load — document the error and halt.

---

## Phase 3: Task Creation

Using the `USER_JOURNEYS` list from Sub-agent 1 and bug findings from Sub-agent 3, create a task for each journey:

- `journey_[name]` — Test the complete [name] flow end-to-end
- `responsive_check` — Verify all key pages at mobile, tablet, and desktop
- `bug_verification_[N]` — Verify and fix each bug found by Sub-agent 3

Work through tasks in order. Mark each complete before starting the next.

---

## Phase 4: User Journey Testing

For each journey, follow this pattern:

### Snapshot → Act → Verify → Screenshot

```bash
# Get all interactive element references on current page
agent-browser snapshot -i

# Interact with elements by reference
agent-browser click @e1                    # Click a button or link
agent-browser fill @e2 "test@example.com"  # Fill a text input
agent-browser fill @e3 "SecurePass123!"    # Fill a password field
agent-browser select @e4 "Option Label"    # Choose a dropdown option

# Capture what happened
agent-browser screenshot screenshots/[journey]/[step].png
```

**Critical rules:**
- Re-snapshot after every navigation or DOM change — element refs become invalid
- Screenshot every meaningful state: before action, after action, success/error states
- Check browser console for errors after each action:
  ```bash
  agent-browser snapshot  # Check for console errors in output
  ```

### Database Validation

After any action that creates, updates, or deletes data, validate via the queries from Sub-agent 2:

```bash
# PostgreSQL / Supabase
psql $DATABASE_URL -c "SELECT * FROM users WHERE email = 'test@example.com' LIMIT 1;"

# SQLite
sqlite3 ./database.db "SELECT * FROM users WHERE email = 'test@example.com';"
```

Confirm records exist, values match what was entered, and no orphaned records were created.

### Issue Resolution

If a bug is found during testing:

1. Screenshot the broken state: `agent-browser screenshot screenshots/bugs/[issue-name]-broken.png`
2. Document: file path, line number, description, root cause
3. Fix the code
4. Restart the dev server if needed
5. Re-test the failing journey from the beginning
6. Screenshot the fixed state: `agent-browser screenshot screenshots/bugs/[issue-name]-fixed.png`

### Responsive Testing

Test each key route at three viewports:

```bash
# Mobile (iPhone 14)
agent-browser set viewport 375 812
agent-browser screenshot screenshots/responsive/[route]-mobile.png

# Tablet (iPad)
agent-browser set viewport 768 1024
agent-browser screenshot screenshots/responsive/[route]-tablet.png

# Desktop
agent-browser set viewport 1440 900
agent-browser screenshot screenshots/responsive/[route]-desktop.png
```

Check for: horizontal scroll, overflowing text, broken layouts, touch target size (min 44×44px on mobile).

**Coverage goal: Go through EVERY interaction, EVERY form field, EVERY button. By the time this completes, every visible part of the UI must have been exercised and screenshotted.**

---

## Phase 5: Cleanup

```bash
# Close the browser session
agent-browser close

# Stop the dev server
kill $DEV_PID 2>/dev/null || pkill -f "npm run dev" || true
```

---

## Phase 6: Reporting

### Text Summary (always output)

```
## E2E Test Summary

Journeys Tested:    [N]
Screenshots Taken:  [N]
Bugs Found:         [N]
Bugs Fixed:         [N]
Bugs Remaining:     [N]

### Issues Fixed
- [description] — [file:line]

### Issues Remaining
- [severity] [description] — [file:line]
  Recommendation: [how to fix]

### Database Validation
- [action]: PASS | FAIL — [details]

### Responsive Coverage
- Mobile (375×812):  PASS | FAIL
- Tablet (768×1024): PASS | FAIL
- Desktop (1440×900): PASS | FAIL

### Overall: PASS / FAIL
```

### Detailed Markdown Report (on request)

If the user asks for a full report, save to `e2e-test-report.md`:

```markdown
# E2E Test Report — [Date]

## Summary
[stats block from above]

## Per-Journey Results

### Journey: [Name]
- Steps: [N]
- Screenshots: [list of paths]
- Database Validated: YES / NO
- Status: PASS | FAIL
- Issues: [list or "none"]

## Bug Hunt Findings
[Sub-agent 3 full output]

## Database Validation Queries & Results
[For each action: query + result]

## Responsive Testing
[Per-route, per-viewport screenshots and notes]

## Remediation Recommendations
[Prioritized list of remaining fixes]
```

---

## Notes

- **Element refs expire** on every navigation or DOM mutation — always re-run `agent-browser snapshot -i` after page changes
- **Screenshots directory:** organize as `screenshots/[journey-name]/[step-N]-[description].png`
- **Parallel TTS:** do NOT parallelize database queries during testing — validate sequentially after each action
- **Supabase auth:** if the app uses Supabase Auth, use a dedicated test account, never production credentials
- **Quota awareness:** if the app has API rate limits (e.g., YouTube Data API), mock external calls where possible during testing
