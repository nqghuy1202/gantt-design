# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

This workspace customizes an Oracle APEX region plugin (`apex-plugin-dhtmlx-gantt`) into an
**MES Control Tower** — a manufacturing-execution Gantt that shows production-order (PO) progress
by stage → machine within a **single fixed 24-hour day**. It is NOT a generic project Gantt.

The original spec lives in `TÀI LIỆU ĐẶC TẢ TÍNH NĂNG SƠ ĐỒ GANT THEO CÔNG ĐOẠN SẢN XUẤT.docx`
(the target screen and field meanings). Design research, roadmap, and the data contract are in
`gantt-ux-research/` (numbered docs 01–10 + `mockups/`).

## Architecture: customize-in-place, dual-mode

The customization is a **fork-in-place** of the existing plugin — keep the internal name
`COM.GITHUB.OGOBRECHT.DHTMLXGANTT`, keep the dhtmlx vendor files, and add a **"Render Mode"**
branch so the plugin supports both:
- **Classic dhtmlx** — the original behavior, untouched.
- **MES Control Tower** — a custom renderer that *replaces* dhtmlx for this view.

What is reused vs replaced (see `gantt-ux-research/06-custom-tai-cho-tren-plugin.md`):
- **Reused:** the AJAX PL/SQL function (`dhtmlx_gantt_ajax`, returns a single CLOB), the region/attribute
  framework, `apexrefresh` binding, page-item submission.
- **Replaced:** the render layer — vendor `gantt.*` calls swapped for a self-written renderer.

**The branch is implemented** in `dhtmlx_gantt_render` (`sources/plugin-source.sql`): it reads
`v_render_mode := nvl(p_region.attribute_20,'classic')`. When `'mes'` it loads `mes-control-tower.css/js`,
emits `<div class="mesct gantt-only">`, calls `plugin_mesGantt.init({...})`, then **`RETURN`s before any
classic code** — the classic dhtmlx path is otherwise byte-for-byte unchanged. MES attribute slots:
**20**=Render Mode (`classic`/`mes`), **16**=Date Page Item, **21**=Duration Unit (`hour`/`minute`),
**22**=Default Date; 16/21/22 are `depending_on` attr 20 = `mes`.

### The renderer (`apex-plugin-dhtmlx-gantt/sources/`)

- `mes-control-tower.js` — module `window.plugin_mesGantt`. Entry point `plugin_mesGantt.init({...})`,
  called from the plugin's onload PL/SQL. It loads data via `apex.server.plugin(ajaxId, {pageItems}, …)`
  when `queryDefined`, re-renders on `apexrefresh`, and handles loading/empty/error states.
- `mes-control-tower.css` — all styles, **scoped under `.mesct`** so they never collide with the APEX theme.
- `_dev-harness.html` — runs the renderer **outside APEX** against embedded sample JSON (uses
  `sampleData` + `plugin_mesGantt.setDate()` instead of AJAX). This is the primary way to preview changes.
- `_header-optimized.html`, `_daytime-range-item-demo.html` — **standalone prototypes of the chrome only**
  (no renderer): the Header KPI strip and the **day + time-range context control** that feeds `dateItem`.
  Same convention as the harness chrome — scoped under `.mes-header` / `.dtr-*`, **not** `.mesct`, with all
  tokens declared on the wrapper. Preview in a browser before folding into native APEX components.
- `mock-data-query.sql` — a region SQL that builds the same sample JSON from `DUAL` (no real tables), for
  testing the MES region inside APEX. Anchored to `TRUNC(SYSDATE)` so it needs no date page item. Two
  contract traps it encodes: `material_short` must be a JSON **number** (1) not the string `"false"` (the
  renderer uses truthiness, so `"false"` reads as truthy); and `JSON_ARRAYAGG`/`JSON_OBJECT` use
  `RETURNING CLOB` to survive the 4000-char limit. Needs Oracle 12.2+ (`ABSENT ON NULL`).
- `real-data-query.sql` — the **production** region SQL: the same JSON contract built from the real MES
  tables (`wip_logs`/`wip_log_details`/`wip_job_*`/`inventory_items`/`order_headers`/`departments`).
  Locked mapping: **machine = `departments`** (`dep.dep_id = wjs.dep_id`) → stage(`wjs`)→machine(`dep`)→po(`batch_no`).
  The real data has **only `finish_date_expected` (no start time)**, so every PO spans the **whole day**
  (`start_date` = `TRUNC(day)` 00:00, `duration` = 23+59/60 h = 23:59) — do not expect truthful sub-day
  bars until a real start column exists. Same `RETURNING CLOB` / number-`material_short` traps as the mock.

**Mode:** the plugin runs `ganttOnly: true` — it renders **only the Gantt grid (`.g-scroll`)**. The Header
KPIs, Production Health sidebar, and the day picker are built separately as native APEX components. The
selected day comes from a page item (`dateItem`, e.g. `P1_NGAY`); a Dynamic Action refreshes the region
on change.

**Time-window context control.** Beyond a plain day, the screen lets the user pick **one day + a start/end
time** (e.g. `25-06-2026 07:00 - 10:00`) — APEX's native `a-date-picker` holds a single value and cannot do
this. Two approaches are prototyped: (1) a **self-written picker** (`_daytime-range-item-demo.html` /
`_header-optimized.html`) on a read-only Text Field, storing `DD-MM-YYYY HH24:MI - HH24:MI`; (2) the
**UC Range Slider** plugin in `UC_RangeSlider_v25.1 (1)/` (noUiSlider, two handles) driven in **Number mode**
over minute-of-day `0..1440` (Date mode only steps daily/weekly — no sub-day granularity). Either way the
region SQL splits the value and computes `TRUNC(day) + minutes/1440` for the bar start/end.

**Chosen real-APEX wiring (approach 1, native).** A read-only Text Field `P200416102_TIME` holds the composed
`DD-MM-YYYY HH24:MI - HH24:MI` string. The three editors live in a **Static Content region with template
"Inline Dialog", Static ID `inline-time`**: native `a-date-picker` `P200416102_DATE` (`format="DD-MM-YYYY"`,
so `getValue()` already returns that format — no ISO conversion) plus two Select Lists
`P200416102_HOUR_FROM`/`P200416102_HOUR_TO` whose LOV is a `connect by` query over `DUAL` emitting
`HH24:MI` display/value every 15 min. Page JS opens the dialog anchored under the item and syncs values:
```javascript
apex.jQuery("#inline-time").dialog("option", {
  modal: false, position: { my:"left top", at:"left bottom+6", of:"#P200416102_TIME" }});
apex.theme.openRegion('inline-time');   // pass dialog opts as 2nd arg, or set via .dialog("option") first
```
On open, split `P200416102_TIME` into the 3 items; on Apply, recombine and `setValue` back (validate
`TO > FROM`). Dialog buttons must use **Action = "Defined by Dynamic Action"** (not Close/Submit) or APEX
acts before the JS runs. Style the wrapper with `:has()` (jQuery UI generates `.ui-dialog`, not reachable by
the region's CSS Classes): `.ui-dialog:has(#inline-time){…}`, `.ui-dialog:has(#inline-time) .ui-dialog-titlebar{display:none}`.
The `_header-optimized.html` / `_daytime-range-item-demo.html` prototypes mirror this with a self-written
`.dtr-pop` + `dtrPicker(itemName,{dateItem,fromItem,toItem})` that falls back to DOM when `apex` is absent.

### The surrounding APEX chrome (prototyped inside the dev harness)

The chrome the plugin does **not** render — a left **Production Health sidebar** and a top **Header KPI
strip** — is prototyped as **static HTML/CSS inside `_dev-harness.html`** (its `<style>` block + body),
laid out beside/above the gantt to preview the whole screen. These are the staging ground for native APEX
components, so their conventions differ from the renderer:
- **Not scoped under `.mesct`.** They use plain BEM-ish classes: sidebar `.mes-sidebar`/`.health-item`/
  `.alert-card`; header `.mes-header`/`.kpi-strip`/`.kpi-perf`/`.perf`. Their CSS lives in the harness
  `<style>`, **not** in `mes-control-tower.css` (keep that file `.mesct`-only).
- **CSS variables are declared on the component wrapper** (`.mes-sidebar`, `.mes-header`) and map to ERP
  design tokens with fallbacks (`var(--primary-color,#15674C)`, `var(--red-color,#D81F25)`, …). The wrapper
  element MUST wrap the markup or every `var(--…)` collapses to transparent/inherit and the UI looks broken —
  the #1 integration gotcha when pasting a fragment into APEX.
- **Locked UX decision — exception-driven color.** Values stay neutral; semantic green/amber/red appears
  **only on threshold breach**, so usually ≤3 colored numbers at once. Production Health stage bars are
  colored by **load** (blue `<80%`, amber `80–90%`, red `>90%` = overload/bottleneck — high % is *bad* here,
  not completion). KPI `Performance` metrics are neutral unless `is-warn`/`is-crit`; status is a consistent
  dot indicator + `⚠` on the worst metric, with the target threshold in a hover `title`.

**Shipped native-APEX build of the chrome.** The two chrome pieces are now deployable (not just prototyped),
each as a trio in `sources/`: **query + PL/SQL Dynamic Content region + extracted CSS**.
- **Production Health sidebar:** `stage-health-query.sql` (raw columns), `stage-health-dynamic-content.sql`
  (region), `stage-health.css`. Completion % = `SUM(actual)/SUM(plan)` grouped at (stage, job-line) grain
  first so `wjl.quantity` is not fan-trap-multiplied. (Note: this build colors by **completion**, per an
  explicit user override of the locked load-coloring rule above.)
- **Header KPI strip:** `header-kpi-query.sql` (one row), `header-kpi-dynamic-content.sql` (region),
  `header-kpi.css` (also styles the native `a-date-picker` item `P200416102_TIME`). Locked KPI formulas:
  **OK Qty** = `SUM(wld.quantity WHERE finish_flag='Y')`, **Scrap** = `SUM(wld.quantity WHERE problem_flag='Y')`,
  **Target** = `SUM(DISTINCT wjl.quantity)` (fan-trap guard), **ORDERS** = `COUNT(DISTINCT order_number)`.
  `Yield%`=OK/(OK+Scrap); `OEE%`=`FCST%`=OK/Target (kept as two tiles despite being equal until a real
  rate×time standard exists). Threshold coloring: `v≥target` ok · `target-2≤v<target` warn · else crit
  (Yield 98 / OEE 85 / SA 95 / FCST 95). Three **data debts** encoded as placeholders: `SA%`=NULL→`—`
  (no real completion time), and `Running`/`Delayed` are **proxies** (Running = order with `0<OK<Target`;
  Delayed = order with `problem_flag='Y'`) — swap when real run status / planned-end time land.

**Two hard gotchas when shipping these regions to APEX** (both cost a debugging cycle here):
1. On this page a **PL/SQL Dynamic Content region is wrapped as a FUNCTION that must `RETURN`** — building
   HTML with `htp.p` throws `ORA-06503: Function returned without value`. Build a `CLOB` and `RETURN` it.
2. The class tokens (`--text`, `--danger`, …) are declared on the **`.mes-sidebar`/`.mes-header` wrapper**,
   whose CSS lives only in the prototypes — you must (a) copy the extracted `.css` into the page and
   (b) set the region's **CSS Classes = `mes-sidebar`/`mes-header`** so the tokens cascade, or every
   `var(--…)` resolves empty and the UI renders unstyled.

### Data contract (`gantt-ux-research/07-data-contract.md`)

Region SQL returns one CLOB of JSON `{ "data": [...], "links": [] }`. Rows are hierarchical via
`row_type` + `id`/`parent`:
- `stage` (công đoạn) → `machine` (tổ máy) → `po` (lệnh sản xuất).
- The **real SQL data contract is single-day**: rows describe one selected day, `end = start_date +
  duration × durationUnit` (`durationUnit` default `"hour"`, configurable). POs outside the current view
  window are clipped. (The renderer's zoom control can widen the *view* to week/month client-side — see
  the time-scale zoom section — but that currently runs on **mock multi-day data in the harness only**;
  the SQL contract has not been extended to return multiple days.)

### Key rendering rules (encoded in the renderer, reflect locked UX decisions)

- **Overlapping POs on one machine = legitimate parallel orders, not a conflict.** Stack into sub-rows
  (greedy first-fit, ≤3); collapse into a `+N` cluster at ≥4. No red/conflict styling.
- **Short bars degrade in-bar (no floating labels):** ≥150px shows code+%+product; 70–150px shows code+%;
  20–70px shows only the % (centered); <20px becomes a `marker` chip (status color only). All detail is in
  the hover popover. The timeline stays truthful to time — only markers get an enforced min-width.
- Status color lives on the machine-row dot; the stage-efficiency badge stays neutral.

### Time-scale zoom control (`.zoomctl` in `mes-control-tower.js`)

A neutral `[ − ][ level label ][ + ]` pill at the **top-right of the Gantt** zooms the time axis through a
**6-level ladder** (`ZOOM[]`, index 0 = most zoomed-in, `+` disabled; index 5 = most zoomed-out, `−`
disabled): **L0 "1 giờ"** → **L1 "2 giờ" (default, == the original view)** → **L2 "4 giờ"** → **L3 "12 giờ"**
→ **L4 "Tuần"** → **L5 "Tháng"**. `+` = zoom in (finer/index−−), `−` = zoom out (wider/index++); also bound
to keyboard `+`/`=` and `−`/`_` (ignored while typing in a field). Locked scope: finest is **1 hour** (never
sub-hour); `+` locks once at the fullest single-day detail, `−` locks at Tháng.

Architecture — `computeView(dayStr, z)` returns `{start, ms, ticks, grids, multi, gridPx}` and render uses a
generalized window `[dayStart, dayEnd]` (no longer a hardcoded 24h clip):
- **L0–L3** keep the single-day data; only tick *density* and timeline *width* change.
- **L4–L5** switch to a multi-day axis (Tuần = Mon–Sun containing the selected day; Tháng = 1st→last),
  one column per day, PO bars laid out by absolute datetime across days.
- **Bars scale with time (co-giãn):** each level carries `pxH` (px/hour) or `pxD` (px/day) → `gridPx` =
  timeline width, applied as `P.el.grid.style.minWidth`. Bars are `%`-positioned within the lane, so they
  stretch/compress with `gridPx` and the grid **scrolls horizontally** when wider than the viewport.
  `min-width` (not `width`) means levels narrower than the viewport fill it (no right-hand whitespace), so
  the stretch is most visible zooming **in** (1 giờ) and in Tuần/Tháng. Default L1 `gridPx`=880 (unchanged
  from the pre-zoom view). Multi-day labels are centered per column via `.tick.lab` (see the CSS override
  after the `:first-child`/`:last-child` axis rules).

Control stays **neutral** (exception-driven color rule — no semantic colors), matches `.daynav` tokens, and
lives in both shell branches (floating `.g-toolbar.zoombar` in `ganttOnly`, appended to the toolbar
otherwise). The harness seeds extra multi-day mock POs (P6–P11 on adjacent June-2025 days) purely to
exercise Tuần/Tháng.

### PO interaction events

- **Hover** any `.po` bar → detail popover (`showPop`/`bindPop` in `mes-control-tower.js`).
- **Click** a `.po` bar → the renderer emits `data-id` (rendered from `p.id`, so the region SQL **must
  return an `id` for `row_type='po'`**), sets `P.selectedId`, triggers the APEX event
  **`mesgantt_po_click`** on `#<regionId>` with detail `{id, code, order, data}`, and calls the optional
  `onPoClick(id, dataset, el)` config callback. Catch it in APEX via a Dynamic Action (Custom event
  `mesgantt_po_click`, Selection = the MES region) reading `this.data.id`. (Classic dhtmlx mode instead
  uses `onTaskDblClick` → event `dhtmlxgantt_task_double_click`.)

### PO detail drawer (the `onPoClick` consumer)

Clicking a PO opens a **right-slide drawer** to view PO status **and record production** (thành phẩm/phế
phẩm) several times per shift — primary user is a điều độ viên on desktop, keyboard-first. It is **not folded
into the renderer yet**; it is designed as a standalone mockup. Research + design contract live in
`gantt-ux-research/`: `09-drawer-ghi-nhan-user-story.md` (personas, friction F1–F9, exceptions E1–E12,
acceptance criteria AC1–AC10) and `10-drawer-ui-ux-nguyen-tac.md` (UI/UX principles + a11y/ship checklist).
Current build: `mockups/po-detail-drawer-v2.html` (`po-detail-drawer.html` is the original kept for diffing).

Locked drawer decisions: tabs **"Ghi nhận | Lịch sử"** with a shared at-a-glance header; **E3 over-plan =
warn + confirm, still save** (over-production is legitimate) while E1 negative / E2 sum-zero are hard-blocked;
E7 close-while-dirty → AlertDialog; after save, refocus the Thành phẩm field with realtime accumulation. All
scoped under one `.posd` wrapper with tokens on the wrapper (same APEX-safety convention as the chrome).
Next step (touches plugin code — ask first): wire `onPoClick` → open drawer, feed real data-contract rows.

## Commands

The renderer is plain JS/CSS — preview by opening `apex-plugin-dhtmlx-gantt/sources/_dev-harness.html`
in a browser (no build step needed for the new renderer).

The original plugin uses Grunt (lint the vendor helper, copy/minify to `server/`):

```bash
cd apex-plugin-dhtmlx-gantt
npm install
npx grunt          # default: jshint → copy → uglify
npx grunt watch    # rebuild on source changes
```

Note: Grunt's `jshint`/`copy`/`uglify` are wired to the *original* `sources/plugin-dhtmlxgantt-helper.js`,
not the new MES files. The MES renderer is shipped as-is (no minify pipeline yet).

## Packaging into APEX (current approach: manual paste)

The plugin install SQL (`plugin/apex-5.1.4-…-0.11.0-…sql`) embeds files via `wwv_flow_api.create_plugin_file`
and references them by `p_plugin.file_prefix`. The render/AJAX PL/SQL lives in `sources/plugin-source.sql`
**but the install SQL keeps its own copy** inside `p_plsql_code` as a `wwv_flow_string.join` array of
single-quoted lines (every `'` doubled). **These two copies must be kept in sync by hand** — when you change
the render PL/SQL, mirror it into that array. The MES attribute definitions also live in the install SQL
(`create_plugin_attribute` for sequences 20/16/21/22).

To deploy MES mode: import the install SQL, **upload `mes-control-tower.css`/`mes-control-tower.js` as Plugin
Files** (the `add_library`/`add_file` names must match the filenames exactly — they are NOT base64-embedded
yet, so a 404 here means `plugin_mesGantt is not defined`), set the region's Render Mode = MES, and point its
Source at a JSON-returning SQL query (e.g. `sources/mock-data-query.sql`).

Deploy gotchas seen in practice: if a page that *used to* host a Classic dhtmlx region still has page-level
JS referencing the dhtmlx global (`gantt.templates...`, `gantt.config...`), MES mode does NOT load dhtmlx, so
that leftover code throws `gantt is not defined` — strip it. `dữ liệu không phải JSON object` means the AJAX
response did not start with `{` (check the Network response: `no_query_defined`, an `ORA-…`, or no rows).

## Working norms for this repo

- Work and preview in `sources/` via the dev harness before touching the plugin install SQL / PL/SQL.
- UX changes are iterated as standalone HTML mockups in `gantt-ux-research/mockups/` and reviewed in a browser
  before being folded into the renderer. Preserve the locked UX decisions above unless explicitly changed.
- This is a Windows environment; the Bash tool maps `/tmp` outside the project — copy files into the project
  tree before reading generated artifacts with the Read tool.
- **UI/UX design uses an installed skill stack** (see numbered research docs for worked examples): `ui-ux-pro-max`
  (`.claude/skills/ui-ux-pro-max`, Python CLI — picks style/palette/typography via `search.py … --design-system`)
  to choose the direction; `design-taste-frontend` for anti-slop craft principles; `ui-skills` (`npx ui-skills
  get <slug>` — `baseline-ui`, `fixing-accessibility`) to audit a finished file. All three skew toward
  landing-page / Tailwind / React — take the **principles**, not literal classes (mockups are vanilla CSS scoped
  for APEX), and never let their color/pattern defaults override the locked exception-driven color rule.
