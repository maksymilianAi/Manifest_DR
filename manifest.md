# Design Review Manifest V3.3

## About this file
This file contains personal preferences and rules for design review sessions.
Always follow these rules when asked to do a design review.

---

## Viewport
The browser viewport is always set to 1440px width manually before the session.
Do not attempt to resize it.

---

## How I work
- I will provide one screenshot: Figma design only
- You will take one screenshot of the current browser page for the frontend — timing is defined in Step 0
- If you need to verify a value from Figma, ask me: "Please provide Figma link for: [specific component]"
- Do not fetch Figma on your own — always ask me for a specific node link

---

## Step 0 — Before starting analysis
After receiving the Figma screenshot:
1. Ask me: "Is there any context I should know before starting the review? (e.g. known issues, things to skip, specific focus areas)"
2. Wait for my answer. If I say "no" or "nothing" or similar — proceed immediately without any follow-up questions.
3. Then take one screenshot of the current browser page and begin the analysis.

---

## Rules
- Do NOT show a plan, outline, or list of steps before starting the analysis — begin immediately
- Do NOT ask for approval before proceeding — just start
- Take MAXIMUM 1 screenshot of the current browser page for frontend comparison
- Do not take additional screenshots unless absolutely necessary
- Only check what looks visually different in the provided screenshots
- Use DevTools only to confirm values where a visual difference is already confirmed from screenshot comparison — never use DevTools proactively on elements that look correct
- When DevTools verification is needed, batch ALL queries into a single JavaScript call that collects every suspicious value at once — never make separate JS calls per element

---

## Ignore completely
- Top navigation header with logo and menu (already tested, skip entirely)
- Sidebar navigation (already tested, skip entirely)
- Sticky header presence on scrolled screenshots
- Transparent backgrounds where the parent background produces the same visual result
- Pagination page count (number of pages is dynamic and depends on real data — do not flag as bug)
- Any Claude plugin UI elements visible on the page (e.g. "Claude is active in this tab group" banner) — these are browser extension artifacts, not part of the frontend

**Must check:**
- Page header with breadcrumbs — must be checked and must match the design

---

## Text content rules
Some text is static and must match the design exactly. Some text is dynamic — do not flag it as a bug.

**Static text — must match exactly (flag as bug if different):**
- Button labels
- Page titles and headings
- Table column headers and legend text
- Description / helper text
- Status labels in tables
- Placeholders

**Dynamic text — values change with real data (do NOT flag as bug):**
- Table cell values (numbers, dates, names, amounts)
- Pagination page count
- User-generated content
- Data fetched from backend

If unsure whether a text difference is a real bug or just dynamic data — ask me:
"Is the text '[value]' in [component] supposed to be static or dynamic?"

---

## Pagination rules
- Pagination component itself (arrows, page buttons, styling) must match the design exactly
- Number of pages shown is dynamic — do NOT flag as a bug
- Everything else in pagination (button size, color, font, spacing) must match

---

## What to check
- Button HEIGHT (even 1px difference = bug)
- Icon type and size
- Text size and text color
- Background and UI colors
- Static text content must match exactly (see Text content rules above)
  - "❌ Text mismatch" if static text differs between Figma and frontend
  - "⚠️ Typo in frontend" if spelling error on frontend
  - "⚠️ Typo in design" if spelling error in Figma
- Spacing between icon and adjacent text
- Line height of text blocks (verify via DevTools only if a visual difference is confirmed)
- Margin between text lines/blocks (verify via DevTools only if a visual difference is confirmed)

---

## Acceptable tolerance
- Button WIDTH: up to ±10px difference is acceptable

---

## Color checking rule
Always evaluate the VISUAL result, not just the raw CSS value.
If an element is transparent but visually matches the design due to its parent background — this is NOT a bug.

**Color format — always use hex.**
When reporting any color value (expected or actual), always convert it to a hex code.
Never use rgb(), rgba(), hsl(), or any other format in the bug table or report.
If DevTools returns rgb(23, 106, 246) — convert it to #176AF6 before reporting.
If the design shows a color name or variable — resolve it to its hex value.

---

## Design audit (Figma itself)
While reviewing, also check the Figma screenshot for design-side issues:
- Font "Popping" anywhere in Figma → "⚠️ Design bug — wrong font in Figma"
- Any typo in Figma text → "⚠️ Typo in design — must fix in Figma"

---

## Bug numbering
Number every bug found, starting from 1. Example: "#1", "#2", "#3"
Do NOT assign priority — I will decide which bugs are critical during the review.

---

## Chat formatting rules
- Do NOT use bold text anywhere in chat responses during the review
- Write all commentary and questions in plain text only
- Only the table itself uses formatting

## Output table format
When showing the bug table in chat, the table must look exactly like this:

| # | Component | Property | Expected (Figma) | Actual (Frontend) |
|---|-----------|----------|------------------|-------------------|
| 1 | Button    | Height   | 40px             | 42px              |

CRITICAL RULES:
- Exactly 5 columns, no more, no less
- NO empty columns anywhere
- NO extra column between # and Component
- Every row must have all 5 cells filled
- Do NOT add a Priority column

---

## After showing all bugs
Wait for me to review the list.
I will tell you which bugs to dismiss and which are critical, for example:
"dismiss 3, 5, 8 — bug 2 is critical"

**Dismiss silently.** Remove dismissed bugs from the report without asking why. No follow-up questions per bug.

Exception: if I volunteer a reason (e.g. "dismiss 3 — sidebar is always excluded"), treat it as a potential manifest rule and say "Got it, I'll note that for the manifest."

After dismissals are resolved:
1. Auto-detect the feature name from the current page title — do not ask me
2. Ask me once: "Would you like to add a Figma link to the report? (optional)"
   - If I provide a link — include it in the report header as a clickable "🔗 Link to Actual Designs"
   - If I say no or skip — proceed without it

Then generate the HTML report immediately.

---

## Visual evidence — screenshot cropping rules

For each bug, crop both screenshots to visually highlight the problem area.

### Core principles

**Focus on the problem place.**
Each crop must be centered on the specific element that differs — not the whole page, not the whole component. The viewer should immediately see what is wrong without scanning.

**Same height for both sides.**
The frontend crop and design crop in each bug card must be scaled to the same pixel height. Never let one side be taller or shorter than the other.

**Minimum height: 100px.**
No screenshot may render shorter than 100px. Thin strips (e.g. breadcrumb bars) must be scaled up to meet this minimum.

**Exclude the sidebar.**
Never include the sidebar navigation in any crop. Use DevTools `getBoundingClientRect()` or pixel scanning to find where sidebar ends and content begins before cropping.

**Zoom level: 2× default, 3× for small/thin elements.**
Scale crops up by 2× for normal elements. Use 3× for thin rows (table headers, breadcrumbs, single-line labels) to ensure readability.

### How to crop

1. Identify the exact bounding box of the problem element using DevTools:
   `const r = document.querySelector('SELECTOR').getBoundingClientRect(); console.log(r.top, r.left, r.width, r.height)`
2. Add padding: 16px horizontal, 10px vertical around the element
3. Find the matching area in the Figma screenshot
4. Scale both crops to the same height (use the taller of the two, then apply zoom multiplier)
5. Ensure height ≥ 100px after scaling

### Centering rule
The element being compared must appear centered (or near-centered) in both the frontend and design crop. Do not let the subject sit at the far left or right edge of the frame.

Example: if the bug is about the "Elevate" label in a breadcrumb, crop so "Elevate" sits in the middle of the screenshot — not at the edge with the sidebar occupying half the frame.

### What to do when text is at the edge of the sidebar
If the problem element starts right after the sidebar:
- Find the exact pixel where the sidebar ends (scan for dark navy pixels)
- Start the crop from that x position + a small gap
- The content must be centered in the remaining crop window

---

## Report format

Generate a single HTML report file with only the active (non-dismissed) bugs.

### How to generate the report
- Generate exactly ONE HTML file — nothing else
- Do NOT generate PDF files
- Do NOT open new browser tabs
- Do NOT download any other files
- If something fails — report the error instead of trying alternatives

**File name:** `design-review-[feature-name]-[YYYY-MM-DD].html`
Example: `design-review-expense-details-2026-03-10.html`

Before generating the report, get today's real date by running this in the browser console:
```
new Date().toLocaleDateString('en-US', { year: 'numeric', month: 'long', day: '2-digit' })
```
Use the returned value as the display date and in the filename. Never hardcode or guess the date.

---

### HTML report structure

**Header:**
- Title: "Design Review Report"
- Feature name
- Date (real date from browser console)
- Figma link if provided — shown as clickable "🔗 Link to Actual Designs" on the same line

**Summary bar:**
Three equal cards side by side: Total Bugs / Critical / Minor

**Bug cards — one card per bug:**

Each bug is a separate card with this structure:

```
┌─────────────────────────────────────────────────────────┐
│ COMPONENT NAME (uppercase)          Bug #N  [CRIT/Minor] │  ← grey header row
├─────────────────────────────────────────────────────────┤
│  [Design screenshot]     →   [Frontend screenshot]      │  ← visual comparison
│  "Design — expected"         "Frontend — current"       │    (fafafa background)
├─────────────────────────────────────────────────────────┤
│  Property      │  Expected (Figma)  │  Actual (Frontend) │  ← info row
└─────────────────────────────────────────────────────────┘
```

**CRITICAL — screenshot order:**
- Left screenshot: Design (expected) — labeled "Design — expected"
- Right screenshot: Frontend (current) — labeled "Frontend — current"
- The "Expected (Figma)" info column must sit underneath the Design screenshot
- The "Actual (Frontend)" info column must sit underneath the Frontend screenshot

**Card header row (grey background #f1f3f7):**
- Left: component name in small uppercase
- Right: "Bug #N" pill + priority badge side by side

**Visual comparison section:**
- Two screenshots side by side, equal height, separated by a "→" arrow
- Left screenshot: labeled "Design — expected" (light pill: #f8fafc background, dark text, grey border)
- Right screenshot: labeled "Frontend — current" (dark pill: #1e293b background, white text)
- Each screenshot sits in a container with min-height 100px, white background, light border, rounded corners
- Section background: #fafafa

**Info row (three equal columns):**
- Property — dark text
- Expected (Figma) — green text (#16a34a) — placed under the Design screenshot
- Actual (Frontend) — red text (#dc2626) — placed under the Frontend screenshot
- Columns separated by light vertical dividers

**Priority badges:**
- CRIT: red subtle (background #fef2f2, text #b91c1c, border #fecaca)
- Minor: grey subtle (background #f8fafc, text #475569, border #cbd5e1)

**Footer:**
"Generated by Claude Design Review · [date]"

---

### HTML styling
- Clean, minimal design, white background (#fff cards on #f4f5f7 page)
- System font stack: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif
- Max-width 1040px, centered
- Cards with 1px border (#e2e4e9), 12px border-radius
- Summary cards: same border style, 10px border-radius
- No bold text used outside of component names and labels

---

## Zero bugs case
If no visual differences are found after the full review, respond with:
"No bugs found. The frontend matches the design."
Then ask once: "Would you like to add a Figma link to the report?"
Generate the HTML report with an empty bug list and a "No bugs found" summary.

---

## Start
The Figma screenshot is provided above. Begin with Step 0.
