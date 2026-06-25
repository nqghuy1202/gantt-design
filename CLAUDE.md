# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

This workspace customizes an Oracle APEX region plugin (`apex-plugin-dhtmlx-gantt`) into an
**MES Control Tower** — a manufacturing-execution Gantt that shows production-order (PO) progress
by stage → machine within a **single fixed 24-hour day**. It is NOT a generic project Gantt.

The original spec lives in `TÀI LIỆU ĐẶC TẢ TÍNH NĂNG SƠ ĐỒ GANT THEO CÔNG ĐOẠN SẢN XUẤT.docx`
(the target screen and field meanings). Design research, roadmap, and the data contract are in
`gantt-ux-research/` (numbered docs 01–08 + `mockups/`).

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

### Data contract (`gantt-ux-research/07-data-contract.md`)

Region SQL returns one CLOB of JSON `{ "data": [...], "links": [] }`. Rows are hierarchical via
`row_type` + `id`/`parent`:
- `stage` (công đoạn) → `machine` (tổ máy) → `po` (lệnh sản xuất).
- Time axis is fixed 00:00–24:00 of the selected day; `end = start_date + duration × durationUnit`
  (`durationUnit` default `"hour"`, configurable). POs outside the day are clipped.

### Key rendering rules (encoded in the renderer, reflect locked UX decisions)

- **Overlapping POs on one machine = legitimate parallel orders, not a conflict.** Stack into sub-rows
  (greedy first-fit, ≤3); collapse into a `+N` cluster at ≥4. No red/conflict styling.
- **Short bars degrade in-bar (no floating labels):** ≥150px shows code+%+product; 70–150px shows code+%;
  20–70px shows only the % (centered); <20px becomes a `marker` chip (status color only). All detail is in
  the hover popover. The timeline stays truthful to time — only markers get an enforced min-width.
- Status color lives on the machine-row dot; the stage-efficiency badge stays neutral.

### PO interaction events

- **Hover** any `.po` bar → detail popover (`showPop`/`bindPop` in `mes-control-tower.js`).
- **Click** a `.po` bar → the renderer emits `data-id` (rendered from `p.id`, so the region SQL **must
  return an `id` for `row_type='po'`**), sets `P.selectedId`, triggers the APEX event
  **`mesgantt_po_click`** on `#<regionId>` with detail `{id, code, order, data}`, and calls the optional
  `onPoClick(id, dataset, el)` config callback. Catch it in APEX via a Dynamic Action (Custom event
  `mesgantt_po_click`, Selection = the MES region) reading `this.data.id`. (Classic dhtmlx mode instead
  uses `onTaskDblClick` → event `dhtmlxgantt_task_double_click`.)

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
