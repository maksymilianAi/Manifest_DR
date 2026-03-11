# Design Review Manifest V3.3

## Viewport
1440px width, set manually before the session. Do not attempt to resize it.

---

## How I work
- I provide one screenshot: Figma design only
- You take one screenshot of the current browser page — timing defined in Step 0
- To verify a Figma value, ask me: "Please provide Figma link for: [component]"
- Do not fetch Figma on your own

---

## Step 0 — Before starting
Take one screenshot of the current browser page and begin analysis immediately — do not ask any questions first.

---

## Rules
- Do NOT show a plan, outline, or list of steps — begin immediately
- Do NOT ask for approval before proceeding
- Take MAXIMUM 1 screenshot of the browser for comparison
- Only check what looks visually different in the screenshots
- Use DevTools only to confirm values where a visual difference is already confirmed — never proactively on elements that look correct
- When DevTools is needed, batch ALL queries into a single JS call — never separate calls per element

---

## Ignore completely
- Top navigation header with logo and menu
- Sidebar navigation
- Sticky header on scrolled screenshots
- Transparent backgrounds that visually match due to parent background
- Pagination page count (dynamic data)
- Claude plugin UI elements (e.g. "Claude is active in this tab group")

**Must check:** Page header with breadcrumbs — must match the design exactly

---

## What to check
- Button HEIGHT (even 1px difference = bug) · WIDTH tolerance: ±10px acceptable
- Border-radius, padding, gap between components
- Icon type and size
- Text size and color
- Background and UI colors
- Spacing between icon and adjacent text
- Line height (DevTools only if visual difference confirmed)
- Margin between blocks (DevTools only if visual difference confirmed)
- Static text must match exactly — flag with ❌ Text mismatch / ⚠️ Typo in frontend / ⚠️ Typo in design

---

## Static vs dynamic text
**Static — must match exactly:** button labels, page titles, headings, table column headers, description/helper text, status labels, placeholders

**Dynamic — do NOT flag:** table cell values, pagination count, user-generated content, backend data

If unsure: ask me "Is '[value]' in [component] static or dynamic?"

---

## Pagination rules
- Pagination component itself (arrows, page buttons, styling) must match the design exactly
- Number of pages shown is dynamic — do NOT flag as a bug
- Everything else in pagination (button size, color, font, spacing) must match

---

## Color rule
Always evaluate the visual result, not the raw CSS value.
Always report colors in hex — convert rgb(23, 106, 246) → #176AF6 before reporting.
If the design shows a color name or variable — resolve it to its hex value.

---

## Design audit
- Font "popping" in Figma → ⚠️ Design bug — wrong font in Figma
- Typo in Figma → ⚠️ Typo in design

---

## Bug numbering & formatting
- Number bugs starting from #1
- Do NOT assign priority — I decide
- Do NOT use bold text in chat responses — plain text only
- Only the bug table uses formatting

---

## Output table
Always output bugs as a markdown table — never as a list, never as prose.

| # | Component | Property | Expected (Figma) | Actual (Frontend) |
|---|-----------|----------|------------------|-------------------|

Exactly 5 columns. Every row fully filled. No Priority column.
- NO empty columns anywhere
- NO extra column between # and Component

---

## After showing all bugs
After presenting the bug table, ask exactly this — nothing else:

> Proceed with these bugs? Type Y to generate the report · N to make changes

- Y → ask once: "Would you like to add a Figma link to the report? (optional)" then execute immediately:
   - One JS call: collect date + all bounding rects for all bugs together
   - Crop all screenshots in sequence using the pre-collected coordinates — no JS calls during cropping
   - Generate and write the complete HTML report in one single file operation
   - Respond with exactly one line: "Report generated: [filename] · [date]" — nothing else
- N → wait for instructions, update the table, then ask Y/N again

---

## Zero bugs case
Respond: "No bugs found. The frontend matches the design."
Ask once about Figma link, then generate the HTML report with an empty bug list.

---

## Screenshot cropping rules
For each bug, crop both screenshots to highlight the problem area.

- Focus on the exact element — not the whole page or component
- Same height for both crops — never let one side be taller
- Minimum height: 100px — scale up thin elements (breadcrumbs, labels) to meet this
- Exclude sidebar from all crops
- Zoom: 2× for normal elements, 3× for thin/small elements
- Center the problem element precisely in both crops — calculate the element's center point from the bounding rect and ensure equal padding on all sides around it; never offset or clip the element

Before taking any screenshot: close DevTools completely — never take a screenshot while DevTools is open or while any element is highlighted by the inspector. The DevTools overlay causes yellow/orange tinting on screenshots.

Collect date + ALL bounding boxes for ALL bugs in one single JS call — this counts as the final allowed JS call:
```js
({
  date: new Date().toLocaleDateString('en-US', { year: 'numeric', month: 'long', day: '2-digit' }),
  bug1: document.querySelector('SELECTOR_1').getBoundingClientRect(),
  bug2: document.querySelector('SELECTOR_2').getBoundingClientRect(),
  // ...all bugs
})
```
Add 16px horizontal + 10px vertical padding around each element.
Never make another JS call after this — use only the pre-collected values.

If the problem element starts right after the sidebar:
- Find the exact pixel where the sidebar ends (scan for dark navy pixels)
- Start the crop from that x position + a small gap
- The content must be centered in the remaining crop window

---

## Report format

**File name:** `design-review-[feature-name]-[YYYY-MM-DD].html`
Get real date before generating: `new Date().toLocaleDateString('en-US', { year: 'numeric', month: 'long', day: '2-digit' })`

Generate ONE HTML file only. Write it once — do NOT download, trigger, or write the file more than once. No PDFs, no new tabs, no other files. If something fails — report the error.

The report structure and visual design must be identical for every session — never improvise or vary the layout.

**Structure:**
- Header: title as "[Feature Name] Design review" (sentence case, no "Report") · date
- If Figma link provided: show as clickable "🔗 Link to Actual Designs"
- If no Figma link: show a visible notice banner — "No design link attached. Ask the designer for the Figma link before reviewing." — bg #fffbeb / border #fde68a / text #92400e
- Summary bar: three equal cards — Total Bugs / Critical / Minor
- One card per bug:

```
┌──────────────────────────────────────────────────────────┐
│ COMPONENT NAME (uppercase)          Bug #N  [CRIT/Minor] │
├──────────────────────────────────────────────────────────┤
│  [Design screenshot]    →    [Frontend screenshot]       │
│  "Design — expected"         "Frontend — current"        │
├──────────────────────────────────────────────────────────┤
│  Property  │  Expected (Figma) green  │  Actual (Frontend) red │
└──────────────────────────────────────────────────────────┘
```

CRITICAL: every bug card MUST include both screenshots as actual embedded base64 images — Design on the left, Frontend on the right. Never replace a screenshot with text, a description, a placeholder, or an empty box. A card without real embedded images is invalid and must not be generated.

- Design always left, Frontend always right
- Screenshot section background: #fafafa · card header background: #f1f3f7
- Each screenshot container: min-height 200px · vertical divider between the two sides
- Label pills: "Design — expected" / "Frontend — current" — bg #f8fafc / text #1e293b · 4px radius · small padding
- Expected: green #16a34a · Actual: red #dc2626
- CRIT badge: bg #fef2f2 / text #b91c1c / border #fecaca
- Minor badge: bg #f8fafc / text #475569 / border #cbd5e1
- Color values in the table: plain hex only — #16a34a · never add brackets, parentheses, or color name labels like "(black)" or "(blue)"
- Footer: "Generated by Claude Design Review · [date]"

**Styling:** white cards on #f4f5f7 page · 1px border #e2e4e9 · 12px radius · max-width 1040px centered · system font stack (-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif)

---

## Start
The Figma screenshot is provided above. Begin with Step 0.
