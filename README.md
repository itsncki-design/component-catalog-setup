# Component Catalog Setup — AI Skill

A Claude skill that sets up a component catalog — Widgetbook for Flutter, Storybook
for React/Vue/Svelte/Angular/web — in an existing codebase, so designers and
developers can browse every reusable UI component in isolation, in every state,
without running the full app. Works with Claude Code and any skills-compatible tool.

## What it does

- Detects your stack automatically (Flutter, React, Vue, Svelte, Angular, Next.js,
-   Nuxt, SvelteKit, React Native, monorepos) and picks Widgetbook or Storybook —
-     you never have to know which one you need.
- - Installs and configures the catalog using current, verified commands rather than
  -   memorized/stale ones.
  -   - Wires the catalog into your app's real theme (including light/dark) instead of
      -   leaving it on a generic default theme.
      -   - Seeds the catalog with 3–5 real components from your codebase, each with a few
          -   meaningful states (default, disabled, empty, long text, etc.) so it's never handed
          -     off empty.
          - - Can build the catalog straight from a Figma design system — extracting tokens
            -   first, then generating components one at a time — when components live in Figma
            -     and not yet in code.
            - - Verifies the catalog actually runs before declaring the job done.
              - - Does all of this on a new git branch, so nothing touches your main code until
                -   it's reviewed.
               
                -   ## How to install
               
                -   ### Option 1 — Terminal (works everywhere)
               
                -   ```bash
                    npx skills add https://github.com/itsncki-design/component-catalog-setup
                    ```

                    Run this in your terminal, use arrow keys + space to select your AI tool(s), hit Enter.

                    ### Option 2 — Download and install

                    Download the packaged skill from the latest release and install it in your AI tool's
                    settings.

                    For Claude / Claude.ai: Settings → Skills → Install skill → upload the file.

                    ## Compatible tools

                    | Tool | Install method |
                    |---|---|
                    | Claude Code | Terminal or `.skill` file |
                    | Cursor | Terminal |
                    | GitHub Copilot | Terminal |
                    | Windsurf | Terminal |
                    | Any skills-compatible tool | Terminal |

                    ## How to use

                    Once installed, just say something like:

                    "Set up a component catalog for our design system"

                    Or mention Widgetbook, Storybook, a "component library", "UI kit", or "component
                    playground" — the skill triggers even if you don't know which tool you need.

                    Building from Figma instead of existing code:

                    "Build our design system from Figma"

                    ## Skill structure

                    ```
                    component-catalog-setup/
                    ├── SKILL.md                    — main skill instructions and workflow
                    └── references/
                        ├── widgetbook.md            — Flutter / Widgetbook setup, step by step
                        ├── storybook.md             — React/Vue/Svelte/Angular/web Storybook setup
                        └── figma.md                 — building the catalog from a Figma design system
                    ```

                    ## Built by

                    @itsncki-design — if this helped your team ship a design system faster, give it a
                    star and share it with someone building component libraries.
                    
