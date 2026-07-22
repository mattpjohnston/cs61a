# Consistent Neovim Font-Style Policy Implementation Plan

> **For Hermes:** Use the `subagent-driven-development` skill to implement this plan task-by-task.

**Goal:** Apply one restrained, legibility-first bold/italic policy to all ten Neovim theme families while retaining each theme's colours and useful UI emphasis.

**Architecture:** Keep the official theme plugins and existing family/mode selector unchanged. Extend the post-colourscheme normalisation in `~/.config/nvim/lua/config/theme.lua` so typography is governed centrally after each official theme loads. Normalise source/prose styles, but leave non-code UI bold choices to the theme so active and urgent interface states remain easy to find.

**Tech Stack:** Neovim 0.12.4, Lua, `vim.pack`, Tree-sitter highlight groups, LSP semantic-token groups, `vim.api.nvim_get_hl()`, `vim.api.nvim_set_hl()`.

---

## Recommendation

Adopt this policy:

### Italics

1. **Italicise only authored prose emphasis**:
   - `@markup.italic*`
   - `@markup.emphasis*`
   - legacy `@text.emphasis*`
   - corresponding Markdown/HTML italic groups
2. **Keep theme-native italic return statements** when the selected theme supplies them by default.
   - Current audit: only Modus does this (`@keyword.return`).
3. Keep comments, doc comments, variables, parameters, properties, strings, built-ins, diagnostics, hints, paths and plugin metadata upright.
4. Allow bold-italic only when prose is explicitly both strong and emphasised.

### Bold

1. **Routine source code remains regular weight**:
   - variables and parameters
   - functions and calls
   - methods
   - classes/types
   - constants and built-ins
   - strings and comments
   - ordinary keywords, including `return`
   - LSP declaration/definition modifiers
2. **Bold authored prose structure**:
   - `@markup.strong*` / legacy strong groups
   - Markdown/HTML bold groups
   - markup headings/title groups
3. **Bold explicit action markers**, but not the whole surrounding comment:
   - TODO, FIXME, WARN, NOTE and ERROR captures/labels
4. **Leave non-code UI bold styling theme-owned**:
   - selected/focused rows
   - active tab/statusline/mode indicators
   - urgent signs and compact alert labels
   - picker/launcher navigation cues
5. Do not introduce selective bold declarations in this first pass. Tree-sitter and LSP coverage differs by language, so “bold declarations only” would be inconsistent across Python, Rust, TypeScript and Go. Reconsider it only after using the restrained baseline.

This gives each style one job:

- **Colour:** token category and theme identity
- **Italic:** deliberate prose emphasis; plus return only where a theme already treats it specially
- **Bold:** authored strong/heading structure and compact actionable UI—not routine code

---

## Research basis

1. **Syntax highlighting is useful, but the evidence supports categorisation rather than widespread font styling.** Sarkar's small eye-tracking study (10 participants) found highlighted Python tasks completed faster by a median 8.4 seconds and with fewer context switches. It did not compare bold or italics. It also cites earlier work finding that added visual information is effective when it relates to the task. This supports retaining colour categories while reserving weight/style for meaningful emphasis.
   - https://ppig.org/files/2015-PPIG-26th-Sarkar1.pdf

2. **The broader code-legibility evidence is immature.** A 2023 systematic review examined 15 human-subject papers and found null, contradictory and outdated evidence across several formatting choices. Therefore, do not present one typography policy as scientifically optimal; use research-informed restraint plus Matt's observed comfort with Berkeley Mono.
   - https://doi.org/10.1016/j.jss.2023.111728

3. **Heavier text is not automatically easier to read.** A 2024 reading study with 179 participants found lighter weights/grades were read slightly faster than bolder ones, although the reported overall weight effect was marginal (`p = .063`) and involved prose rather than code. This is supporting evidence for sparse—not zero—bold.
   - https://doi.org/10.1167/jov.24.10.1032

4. **Accessibility guidance favours small, meaningful amounts of bold and warns against large italic regions.** University of Leeds guidance recommends only small amounts of bold and notes that large bold/italic areas may be difficult for readers with dyslexia. This is prose guidance, not a direct code study, but aligns with Matt's firsthand report that Berkeley Mono's obliques are distracting.
   - https://digitalaccessibility.leeds.ac.uk/guidance/styling-text/

5. **Critical state should not rely on hue alone.** WCAG 1.4.1 says colour should not be the only visual means of conveying important information. While an editor is not a web page conformance target, the principle supports retaining weight/background/sign cues for urgent or selected UI states rather than flattening every highlight globally.
   - https://www.w3.org/WAI/WCAG21/Understanding/use-of-color.html

6. **Semantic highlighting is layered and can spread broad style rules unexpectedly.** VS Code's official documentation explains that semantic tokens are applied over grammar-based highlighting and can classify parameters, variables, properties, declarations and other modifiers. Neovim similarly combines classic, Tree-sitter and LSP groups. A central effective-highlight normaliser is therefore more reliable than configuring only one layer.
   - https://code.visualstudio.com/api/language-extensions/syntax-highlight-guide
   - https://code.visualstudio.com/api/language-extensions/semantic-highlight-guide
   - https://neovim.io/doc/user/treesitter.html#treesitter-highlight-groups

7. **The installed font supports a real bold but only an oblique italic.** Fontconfig resolves Berkeley Mono Regular and Bold as distinct faces, while italic resolves to `Berkeley Mono Oblique`. This makes sparse bold technically sound and explains why broad italics are less comfortable.

---

## Current audit

Themes tested in both light and dark modes:

- Ayu
- Catppuccin
- Flexoki
- GitHub
- Gruvbox
- Modus
- Monokai Pro
- Oxocarbon
- Rosé Pine
- Token

Light and dark variants currently expose the same bold/italic group sets within each family.

### Routine-code bold currently present

- **Ayu:** none
- **Catppuccin:** none beyond prose/TODO
- **Flexoki:** none beyond prose/TODO
- **GitHub:** none beyond prose
- **Gruvbox:** functions, methods and calls
- **Modus:** booleans
- **Monokai Pro:** none beyond prose/TODO
- **Oxocarbon:** functions and methods
- **Rosé Pine:** built-in attributes/constants/functions/modules/types/variables plus `Statement`
- **Token:** LSP declaration/definition/modification groups

Raw total bold-group counts range from 20 to 254, but those totals are misleading because official themes predefine integrations for many plugins that are not necessarily active. The implementation should distinguish source/prose groups from UI integration groups rather than using a total-count threshold.

### Italics currently present after the existing policy

- Only prose-emphasis compatibility groups remain in most themes.
- Modus additionally retains `@keyword.return` because that theme emphasises returns by default.
- This matches the requested italic policy and should be preserved while refactoring.

---

## Proposed implementation

### Task 1: Refactor style classification into named predicates

**Objective:** Make the policy auditable instead of expanding ad-hoc pattern checks.

**Files:**
- Modify: `/home/matt/.config/nvim/lua/config/theme.lua:44-80`

**Steps:**

1. Keep one predicate for prose italics, covering modern and legacy groups.
2. Add `is_prose_bold(group)` covering:
   - `@markup.strong*`
   - `@text.strong*`
   - `@markup.heading*`
   - `@text.title*`
   - Markdown/HTML bold and heading compatibility groups
3. Add `is_action_marker(group)` for exact TODO/FIXME/WARN/NOTE/ERROR capture or label groups.
4. Add `is_source_group(group)`:
   - all Tree-sitter/LSP groups beginning with `@`, excluding accepted prose/action groups
   - classic Vim syntax groups such as `Keyword`, `Statement`, `Function`, `Type`, `Constant`, `String`, `Comment`, `Identifier`, `Boolean`, `Operator`, etc.
5. Avoid matching arbitrary plugin UI names merely because they contain words such as `Function` or `Keyword`; UI styling stays theme-owned.

**Expected result:** The file clearly states which semantic categories may use each font style.

### Task 2: Consolidate italic and bold handling

**Objective:** Apply the policy after every colourscheme without per-theme duplication.

**Files:**
- Modify: `/home/matt/.config/nvim/lua/config/theme.lua:53-111`

**Steps:**

1. Replace `normalise_italics()` with `normalise_font_styles()`.
2. Before stripping styles, capture resolved `@keyword.return*` highlights that are italic in the freshly loaded theme.
3. For every effective highlight:
   - remove italic unless it is prose emphasis
   - remove bold only when it is a source group and is neither prose structure nor an action marker
4. Restore captured theme-native italic return highlights after generic keyword styles are normalised, preventing a parent `Keyword` change from removing the child return style.
5. Explicitly ensure canonical prose groups preserve their original colours while receiving their semantic style:
   - `@markup.italic`: italic
   - `@markup.strong`: bold
   - `@markup.heading*`: bold when those groups exist
6. Preserve combined bold+italic for actual prose that carries both semantics.
7. Remove the Rosé-Pine-specific setup block if the generic policy fully reproduces its behaviour. This keeps one source of truth and prevents a plugin-specific exception from drifting.
8. Call `normalise_font_styles()` immediately after successful colourscheme load or Token fallback.

**Expected result:** All ten official themes retain their colours while routine source code has an even Berkeley Mono Regular texture.

### Task 3: Add a reusable headless policy audit

**Objective:** Turn the policy into testable invariants rather than visual guesswork.

**Files:**
- Prefer no permanent new file initially; use a documented headless Lua check.
- If the inline command becomes unmaintainable, create: `/home/matt/.config/nvim/scripts/check-theme-font-styles.lua`

**Steps:**

1. Use a temporary `XDG_STATE_HOME` so tests never change the active desktop family/mode.
2. Iterate all ten families and both modes.
3. For each of the 20 combinations, start Neovim headlessly and inspect resolved highlight attributes.
4. Fail when:
   - a non-prose, non-return group remains italic
   - a routine source group remains bold outside prose/action-marker exceptions
   - `@markup.italic` is not italic
   - `@markup.strong` is not bold
5. Assert `@keyword.return` remains italic for Modus light/dark and does not become newly italic in themes where it was upright by default.
6. Print a compact family/mode summary suitable for future theme upgrades.

**Expected result:** 20/20 family-mode combinations pass with no active theme-state changes.

### Task 4: Perform representative language smoke tests

**Objective:** Verify that group-level invariants translate into a comfortable real editor view.

**Files:**
- Temporary fixtures only; do not add sample files to the CS61A lab.

**Steps:**

1. Open short fixtures for Python, Rust, TypeScript, Go and Markdown.
2. Verify source code uses regular weight for:
   - variables and parameters
   - function declarations and calls
   - types and built-ins
   - strings/comments
   - ordinary keywords
3. Verify Markdown:
   - `*emphasis*` is oblique
   - `**strong**` is bold
   - `***strong emphasis***` is bold-oblique
   - headings are bold
4. Verify TODO/FIXME marker labels are bold without making the complete comment bold.
5. Verify Modus retains italic `return`; other themes do not gain it.
6. Check one light and one dark representative visually first, then spot-check the remaining families for regressions in selection, completion, statusline and diagnostics.

**Expected result:** Typography is stable across source code, while prose and actionable UI retain meaningful emphasis.

### Task 5: Verify reload, fallback and rollback behaviour

**Objective:** Ensure the normaliser works during normal theme switching and failure fallback.

**Files:**
- Modify only if a bug is found: `/home/matt/.config/nvim/lua/config/theme.lua`

**Steps:**

1. Run `nvim --headless '+ThemeStatus' '+qa!'`; expect clean startup and the current family/mode.
2. In a live session, run `:ThemeReload`; expect the same scheme and immediate policy application.
3. Test a temporary invalid family in isolated `XDG_STATE_HOME`; expect Token fallback plus the same font-style policy.
4. Confirm `theme-family current` and `theme-switch current` are unchanged after testing.
5. If `~/.config/nvim` is version-controlled, review the diff and commit only the targeted theme module change. Otherwise retain a timestamped backup before implementation.

---

## Files likely to change

- `/home/matt/.config/nvim/lua/config/theme.lua`
- Optional only if the inline audit is too unwieldy: `/home/matt/.config/nvim/scripts/check-theme-font-styles.lua`

No theme plugin source, generated desktop family file, terminal config, Zed config or VS Code config should change.

---

## Risks and mitigations

1. **Resolved highlights can materialise links.** `nvim_get_hl(..., link = false)` followed by `nvim_set_hl()` turns an inherited style into a concrete group. Limit writes to groups whose style actually needs changing, then reapply on every `:ThemeReload`.
2. **Plugins may define highlights after the colourscheme.** First test the current load order. Add a single targeted reapplication event only if a real late-loading plugin violates the policy; do not add timers or broad recurring autocmds pre-emptively.
3. **Theme upgrades may add new groups.** Classification by semantic prefixes plus the headless audit should catch these without maintaining hundreds of theme-specific names.
4. **UI flattening.** Do not strip bold from arbitrary plugin/UI groups in the first pass. If UI bold remains distracting after normal code is regularised, conduct a separate UI-only audit rather than broadening this change.
5. **Language inconsistency.** Avoid declaration-only bold for now because parser/LSP coverage differs across Python, Rust, TypeScript and Go.

---

## Acceptance criteria

- All ten families pass in light and dark modes.
- Routine source code contains no theme-forced bold or italics.
- Explicit prose italic, strong, combined strong-emphasis and headings render semantically.
- TODO/FIXME/WARN/NOTE/ERROR markers may remain bold; surrounding comments remain regular.
- Only themes that already italicised return statements retain that behaviour; currently Modus only.
- Theme colours, terminal colours and non-code UI emphasis remain theme-specific.
- `:ThemeReload`, startup and Token fallback work without errors.
- Active desktop family/mode are unchanged after tests.
