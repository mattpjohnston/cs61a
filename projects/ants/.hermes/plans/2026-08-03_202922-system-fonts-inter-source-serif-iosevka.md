# System Font Roles Implementation Plan

> **For Hermes:** Use subagent-driven-development skill to implement this plan task-by-task.

**Goal:** Make Inter Variable the desktop UI sans, Source Serif 4 the document serif, and Iosevka the monospace/code face across Matt's Fedora/SwayFX desktop.

**Architecture:** Use Fontconfig generic aliases as the system-wide source of truth, then update applications that currently hard-code IBM Plex or another code font. Install only Inter's two official variable-font files; keep the existing Fedora static Inter package and existing fallback fonts during the first pass so the rollout is reversible and coverage does not regress.

**Tech Stack:** Fedora 44, Fontconfig, Pango/GTK 3 and 4, GSettings, SwayFX, Waybar, Fuzzel, Mako, Ghostty, Zed, VS Code, Brave.

---

## Current state

- `sans-serif` resolves to IBM Plex Sans.
- `serif` resolves to IBM Plex Serif.
- `monospace` already resolves to Iosevka.
- GTK/GSettings currently use IBM Plex Sans Text, IBM Plex Serif, and Iosevka at 13.5.
- Source Serif 4 is already installed from `adobe-source-serif-pro-fonts` and resolves correctly.
- Iosevka is already installed under `~/.local/share/fonts/Iosevka/` with regular, italic, bold, and bold italic faces.
- Fedora's `rsms-inter-fonts` package contains static Inter 4.1 faces only. It does not provide the requested variable font, and `fc-match 'Inter Variable'` currently falls back to IBM Plex Sans.
- The current official upstream Inter release is 4.1 and provides `Inter-4.1.zip` under SIL OFL 1.1.
- Sway, Waybar, Fuzzel, and Mako explicitly name IBM Plex rather than following the generic sans alias.
- Ghostty already explicitly uses Iosevka.
- Zed explicitly uses `Iosevka Pragmata`; VS Code explicitly uses PragmataPro for its editor and integrated terminal.
- Brave has no custom family names stored in its active profile, so its web-content defaults are not yet guaranteed to match this system.

## Scope and policy

1. UI sans means desktop chrome and controls: GTK, Sway title bars, Waybar, Fuzzel, Mako, and applications that expose a UI-family setting.
2. Document serif means the Fontconfig `serif` generic and Brave's standard/serif web-content defaults. It does not override fonts explicitly supplied by websites or documents.
3. Monospace/code means the Fontconfig `monospace` generic, terminal, and installed editors. Preserve existing sizes, line heights, ligature choices, icons, and fallback chains.
4. Do not remove IBM Plex, PragmataPro, or the Fedora Inter package in this rollout. Removing packages is a separate cleanup decision after real use confirms the new setup.
5. Preserve Font Awesome at the front of icon-only Waybar stacks and preserve Noto/STIX/emoji/CJK fallbacks.
6. Make live desktop changes sequentially with one writer, a complete backup, and validation before reload.

## Task 1: Capture baseline and rollback material

**Objective:** Make every intended change reversible and detect concurrent drift before writing.

**Files:**
- Back up: `~/.config/fontconfig/conf.d/50-preferred-fonts.conf`
- Back up: `~/.config/gtk-3.0/settings.ini`
- Back up: `~/.config/gtk-4.0/settings.ini`
- Back up: `~/repos/dotfiles/sway/config`
- Back up: `~/repos/dotfiles/waybar/style.css`
- Back up: `~/repos/dotfiles/fuzzel/fuzzel.ini`
- Back up: `~/repos/dotfiles/mako/config`
- Back up: `~/repos/dotfiles/zed/settings.json`
- Back up: `~/.config/Code/User/settings.json`
- Record: current GSettings font values and current Fontconfig matches

**Steps:**

1. Create a timestamped rollback directory under `~/.local/state/hermes/backups/system-fonts-<timestamp>/`, outside live config and the dotfiles repo.
2. Record file type, symlink target, mode, owner, mtime, and SHA-256 for every scoped path.
3. Copy the scoped files while preserving metadata and prove the backup matches the manifest.
4. Record:
   - `fc-match sans-serif`
   - `fc-match serif`
   - `fc-match monospace`
   - `gsettings get org.gnome.desktop.interface font-name`
   - `gsettings get org.gnome.desktop.interface document-font-name`
   - `gsettings get org.gnome.desktop.interface monospace-font-name`
5. Immediately before the first write, compare live hashes with this approved baseline. Stop rather than overwriting any concurrent change.

## Task 2: Install official Inter Variable alongside Fedora Inter

**Objective:** Add the requested variable family without replacing package-managed files.

**Files:**
- Create directory: `~/.local/share/fonts/InterVariable/`
- Create: `~/.local/share/fonts/InterVariable/InterVariable.ttf`
- Create: `~/.local/share/fonts/InterVariable/InterVariable-Italic.ttf`
- Optional provenance record: `~/.local/share/fonts/InterVariable/SOURCE.txt`

**Steps:**

1. Download the official pinned asset `https://github.com/rsms/inter/releases/download/v4.1/Inter-4.1.zip` to a temporary directory.
2. Record its SHA-256 and source URL; do not download from a font mirror.
3. Inspect the archive before extraction and locate the upright and italic variable TTFs.
4. Use `fc-query` on the staged files before installation. Require:
   - expected Inter variable family naming;
   - upright and italic styles;
   - variable metadata present;
   - weight axis covering 100–900;
   - optical-size axis present if included by this upstream build.
5. Copy only the two variable TTFs into the user font directory. Do not copy the archive's duplicate static faces or webfonts.
6. Run `fc-cache -f ~/.local/share/fonts/InterVariable`.
7. Verify the exact internal family reported by `fc-query` and use that name in all following configuration. The expected name is `Inter Variable`, but the binary's metadata is authoritative.
8. Verify `fc-match 'Inter Variable'` resolves to one of the new files, while `fc-match Inter` may continue to resolve to Fedora's static package by design.

## Task 3: Change the three Fontconfig generic roles

**Objective:** Establish the system-wide generic-family defaults while preserving broad glyph coverage.

**Files:**
- Modify: `~/.config/fontconfig/conf.d/50-preferred-fonts.conf`

**Steps:**

1. In the `sans-serif` alias, replace only the first preference, IBM Plex Sans, with the exact installed Inter Variable family.
2. In the `serif` alias, replace only the first preference, IBM Plex Serif, with Source Serif 4.
3. Leave Iosevka first in the `monospace` alias.
4. Preserve the existing Noto, STIX, math, emoji, symbols, and CJK fallbacks in their current order.
5. Do not remove `60-ibm-plex-alt-g.conf` or `61-source-alt-g.conf` in this pass; they remain harmless family-specific rules and preserve behavior if those families are used explicitly.
6. Validate the complete Fontconfig configuration with `fc-conflist`/`fc-match` rather than treating XML well-formedness alone as sufficient.
7. Require these first matches:
   - `fc-match sans-serif` → new Inter variable file
   - `fc-match serif` → Source Serif 4
   - `fc-match monospace` → Iosevka
8. Probe representative Latin, punctuation, emoji, math, and CJK characters with `fc-match -s` to ensure fallback coverage remains available.

## Task 4: Update GTK and desktop UI consumers

**Objective:** Replace explicit IBM Plex UI settings while preserving all approved sizes and layout.

**Files:**
- Modify: `~/.config/gtk-3.0/settings.ini`
- Modify: `~/.config/gtk-4.0/settings.ini`
- Modify: `~/repos/dotfiles/sway/config:17`
- Modify: `~/repos/dotfiles/waybar/style.css:7,39,73,89,102`
- Modify: `~/repos/dotfiles/fuzzel/fuzzel.ini:3`
- Modify: `~/repos/dotfiles/mako/config:1`

**Steps:**

1. Change GTK 3 and GTK 4 `gtk-font-name` from IBM Plex Sans Text to the exact Inter variable family; keep size 13.5.
2. Set GSettings to:
   - UI: Inter variable family at 13.5
   - document: Source Serif 4 at 13.5
   - monospace: Iosevka at 13.5
3. Change Sway's Pango font to Inter Variable with Noto Sans fallback; keep size 13.5.
4. Replace IBM Plex Sans Text/IBM Plex Sans in Waybar text stacks with Inter Variable, followed by Noto Sans and `sans-serif`. Keep every size, weight, spacing, icon family, and CSS geometry unchanged.
5. Change Fuzzel's UI family to Inter Variable while keeping size 13.5 and every launcher setting unchanged.
6. Change Mako's UI family to Inter Variable while keeping size 13.5 and all notification geometry unchanged.
7. Validate before reload:
   - `sway -C -c ~/.config/sway/config`
   - `fuzzel --check-config`
   - a bounded Waybar parser/startup check appropriate for its JSONC config
   - `makoctl reload` only after its config has been checked
8. After all validators pass, reload consumers once. Follow any Sway reload with `kanshictl reload || true` and verify adaptive sync/output state remains correct.

## Task 5: Align code/editor consumers

**Objective:** Make explicit editor and terminal settings agree with the chosen Iosevka code role.

**Files:**
- Verify only: `~/repos/dotfiles/ghostty/config.ghostty:3`
- Modify: `~/repos/dotfiles/zed/settings.json:2`
- Modify: `~/.config/Code/User/settings.json:5,19`

**Steps:**

1. Leave Ghostty unchanged because it already uses Iosevka.
2. Change Zed's `buffer_font_family` from `Iosevka Pragmata` to `Iosevka`. Do not alter its size, line height, modes, theme, or agent settings.
3. Change VS Code's editor and integrated-terminal families from PragmataPro variants to `Iosevka, monospace`. Preserve font size 15, line height 1.2, ligatures, themes, and all unrelated settings.
4. Do not add an explicit Zed UI font family unless the installed Zed settings schema confirms it is supported. If supported and the user wants total app-level consistency, set only that UI family to Inter Variable; otherwise leave Zed's own chrome font alone rather than inventing a setting.
5. Validate both JSON/JSONC files with their actual parsers or application diagnostics; do not use strict JSON against a JSONC dialect.
6. Restart or reload each editor once and confirm the resolved family visually. Helix needs no config change because it renders through the Iosevka-configured terminal.

## Task 6: Set Brave document defaults through supported UI

**Objective:** Ensure ordinary browser documents use the chosen serif and code roles without directly editing Chromium's live Preferences database.

**Files:**
- Runtime profile setting: Brave active profile at `~/.config/BraveSoftware/Brave-Origin/Default/`

**Steps:**

1. Open Brave's supported font settings page (`brave://settings/fonts`) rather than editing `Preferences` while Brave is running.
2. Set:
   - Standard font: Source Serif 4
   - Serif font: Source Serif 4
   - Sans-serif font: Inter Variable
   - Fixed-width font: Iosevka
3. Preserve the current default font sizes (17 proportional, 14 fixed) unless Matt separately requests sizing changes.
4. Restart Brave and inspect only the relevant `webkit.webprefs` keys to verify the profile persisted the four families.
5. Confirm on a controlled local HTML page containing default body text, explicit `serif`, `sans-serif`, `monospace`, bold, and italic. Do not expect this to override a website's own webfonts.

## Task 7: End-to-end validation and rollback gate

**Objective:** Prove the requested roles are active without unrelated visual or configuration drift.

**Steps:**

1. Re-run exact `fc-match` and `fc-query` checks for all three generic roles and all three named families.
2. Confirm GTK/GSettings values exactly match the requested role map and preserve size 13.5.
3. Diff every changed file against the implementation-start backup. Classify every hunk; revert any geometry, color, keybinding, theme, size, or unrelated setting drift.
4. Verify one real surface per role:
   - UI sans: GTK app plus Sway/Waybar/Fuzzel/Mako
   - document serif: controlled Brave page
   - monospace: Ghostty plus Zed or VS Code
5. Verify regular, medium/semibold, bold, and italic rendering. This is especially important for the Inter variable axes and Source Serif's installed italic faces.
6. Confirm `fc-match -s` still supplies emoji, math, symbols, and CJK fallbacks.
7. Verify Sway/Kanshi output state, one persistent Waybar process, active Mako, and no parser errors.
8. If any primary role falls back or a UI looks materially wrong, restore only this font scope from the backup and rerun that scope's native validators. Do not restore unrelated theme/session state.
9. Keep IBM Plex and PragmataPro installed for a short burn-in. Consider package/font cleanup only after Matt approves the result at normal viewing distance.

## Files likely to change

- `~/.local/share/fonts/InterVariable/InterVariable.ttf` (new)
- `~/.local/share/fonts/InterVariable/InterVariable-Italic.ttf` (new)
- `~/.config/fontconfig/conf.d/50-preferred-fonts.conf`
- `~/.config/gtk-3.0/settings.ini`
- `~/.config/gtk-4.0/settings.ini`
- `~/repos/dotfiles/sway/config`
- `~/repos/dotfiles/waybar/style.css`
- `~/repos/dotfiles/fuzzel/fuzzel.ini`
- `~/repos/dotfiles/mako/config`
- `~/repos/dotfiles/zed/settings.json`
- `~/.config/Code/User/settings.json`
- Brave profile font preferences through `brave://settings/fonts`

## Risks and tradeoffs

- Fedora packages static Inter, so satisfying “Inter Variable” requires a small user-managed upstream install. Keeping the static package avoids breaking explicit `Inter` requests and makes rollback simple, at the cost of having both `Inter` and `Inter Variable` installed.
- Variable fonts are well supported by current Fontconfig/Pango, but each consumer must still be verified because family naming and variable-axis handling can differ.
- Fontconfig generics do not control applications or websites that hard-code another family; explicit desktop configs and Brave defaults therefore need separate alignment.
- Changing dormant VS Code/Zed settings makes the role map consistent, but those are app-level changes rather than strictly system defaults. They should remain scoped to family names only.
- Source Serif 4 is installed as static faces, which is sufficient because the request specifies the family, not a variable build.
- Font cleanup is intentionally deferred. First-pass removal would add risk without helping the requested role mapping.

## Acceptance criteria

- `fc-match sans-serif` resolves to the installed Inter variable file.
- `fc-match serif` resolves to Source Serif 4.
- `fc-match monospace` resolves to Iosevka.
- GTK, GSettings, Sway, Waybar, Fuzzel, and Mako use Inter Variable at their existing sizes.
- GSettings and Brave document defaults use Source Serif 4.
- Ghostty, Zed, and VS Code use Iosevka for code/terminal text.
- Existing emoji/math/symbol/CJK fallbacks remain intact.
- All edited configs pass native validation, Sway/Kanshi output state is preserved, and the final diff contains no unrelated changes.
- A tested rollback exists before any live reload.
