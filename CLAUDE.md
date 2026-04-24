# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

An RT (Request Tracker) 6.x extension that provides a classic CRT "green screen" terminal theme. It ships a single stylesheet named `terminal` with both light (pale-green) and dark (phosphor-green) modes, controlled by RT6's standard `WebDefaultThemeMode` mechanism.

## Build & install

```bash
perl Makefile.PL   # generates Makefile
make
make install       # may need root; installs into RT's directory tree
```

After install, enable in `/opt/rt6/etc/RT_SiteConfig.pm`:
```perl
Plugin('RT::Extension::TerminalTheme');
Set($WebDefaultStylesheet, 'terminal');
# Optional — defaults to 'auto' (browser preference):
Set($WebDefaultThemeMode, 'dark');  # or 'light'
```

Then clear the Mason cache and restart the webserver:
```bash
rm -rf /opt/rt6/var/mason_data/obj
sudo systemctl restart apache2   # or nginx/starman depending on setup
```

There are no automated tests in this extension.

## Architecture

RT6 extensions hook into RT's Mason-based rendering pipeline. This extension uses three mechanisms:

**Callback** (`html/Callbacks/RT-Extension-TerminalTheme/Elements/Header/Head`): Runs during `<head>` rendering. If the active stylesheet is `terminal`, injects the custom SVG logo into `LogoURL` and `SmallLogoURL` args (which are passed through to `/Elements/Logo`).

**CSS delivery via Mason** (`html/NoAuth/css/terminal/`): RT serves CSS through Mason templates. Three callback files:
- `InHeader` — injects `<meta viewport>` and (on dashboard render) the mail layout stylesheet
- `BeforeNav` — Bootstrap 5 overflow-menu JavaScript for `#main-navigation` and `#page-navigation`, including HTMX `registerLoadListener` for partial-page reloads
- `AfterMenus` — sticky page-menu initialization

**Static CSS** (`static/css/terminal/`):
- `main.css` — entry point; `@import`s the full elevator theme, then `terminal.css`
- `terminal.css` — all terminal visual overrides using Bootstrap 5 CSS custom properties and `[data-bs-theme=light]` / `[data-bs-theme=dark]` selectors

## CSS strategy

The terminal theme does **not** duplicate elevator's CSS files. Instead `main.css` imports `../elevator/main.css` as the complete base (Bootstrap 5, all RT6 components) and `terminal.css` overrides only colors, fonts, and a few layout details.

Key CSS variable overrides are grouped by theme mode inside `[data-bs-theme=light]` and `[data-bs-theme=dark]` selectors. Bootstrap 5 picks up the `--bs-*` variable changes automatically, so most component styling (forms, tables, buttons) just works without per-component rules.

The monospace font (`Courier New` / system fallbacks) is applied at the `body` level and to form elements.

## RT6 vs RT5 differences handled here

| Area | RT5 | RT6 |
|------|-----|-----|
| Theme name | `terminal-light`, `terminal-dark` | `terminal` (single) |
| Dark mode | separate stylesheet | `data-bs-theme` attribute |
| CSS framework | Bootstrap 3/4 | Bootstrap 5 CSS variables |
| Nav IDs | `#app-nav`, `#page-menu` | `#main-navigation`, `#page-navigation` |
| Dynamic loading | none | HTMX `registerLoadListener` |
| Menu JS | Superfish | plain Bootstrap 5 dropdowns |
