# Building the catalog FROM a Figma design system

Use this when components and tokens live in Figma and the codebase has few or none.

The pipeline is: **tokens → theme → components (one at a time) → stories/use-cases →
catalog**. Never generate components before tokens exist in code, or every component
will hardcode hex values and drift from the design system immediately.

## Prerequisites — check before promising anything

1. Figma MCP access. The user needs the Figma MCP server connected (remote at https://mcp.figma.com/mcp, or the desktop server at http://127.0.0.1:3845/mcp via Figma desktop → Dev Mode → Enable desktop MCP server). If no Figma tools are available in this session, stop and tell the user how to connect it — do not scrape Figma URLs or guess values from screenshots the user pastes (screenshots are fine as reference, not as a token source).

2. Seat check. Starter-plan users and View/Collab seats get ~6 MCP tool calls per month — useless for this pipeline. A Dev or Full seat on a paid plan is effectively required. Ask the user which seat they have before starting.

3. Links, not descriptions. Ask the user for links to: the foundations/tokens page (colors, type, spacing) and the components page of their Figma file. Right-click → "Copy link to selection" in Figma. The remote server works from links; the desktop server can also use the user's live selection.

4. A host codebase. A catalog needs a codebase. If the repo is empty or there's no repo, ask what stack the team ships in and scaffold the minimum first (e.g. flutter create a package, or npm create vite@latest for React, etc.), then proceed. Don't pick a framework for them — this decides who can maintain it.

## Key Figma MCP tools

- get_variable_defs — variables and styles used in a selection (colors, spacing, typography). The token source of truth. If it returns raw code instead of tokens, re-prompt: "Get the variable names and values used in this frame."
- get_design_context — structured representation of a selection (default React + Tailwind; translate to the target framework). The component source of truth.
- get_screenshot — visual reference; always grab one per component to verify against.
- get_metadata — sparse node map; use when a selection is too large and you need to re-fetch only specific children.
- get_code_connect_map — mapping of Figma nodes to existing code components; check it first in mixed codebases so you reuse instead of regenerate.

## Step A — Extract tokens

Run get_variable_defs against the foundations page/frames. Convert to the stack's
token format:

- Web: CSS custom properties in a tokens.css (or the Tailwind config if the project uses Tailwind). Preserve Figma variable names as kebab-case (color/bg/surface → --color-bg-surface). Include both modes if the Figma file has light/dark variable modes.
- Flutter: a theme file — ThemeData plus a ThemeExtension for tokens that don't map to Material slots. One ThemeData per Figma mode.

Show the user the extracted token table before writing code — designers will spot a
stale or wrong value instantly, and it's much cheaper to fix here.

## Step B — Generate components, strictly one at a time

For each component (start with 3–5 leaf components — buttons, chips, badges, inputs):

1. get_design_context + get_screenshot for the component set (the set, not one variant, so you see all variants/properties).
2. Translate to the target framework. **Every color/spacing/radius/type value must resolve to a token from Step A** — a raw hex in generated component code is a bug.
3. Map Figma component properties → code props (Figma variant "State=Disabled" → disabled prop; "Type=Primary/Ghost" → variant prop). Keep the prop names designer-recognizable.
4. Immediately write the story/use-case covering each Figma variant, then verify the rendered result against the screenshot.
5. Only then move to the next component.

Do NOT batch-generate the whole design system in one pass. Quality collapses, token
mapping gets skipped, and the user can't review 40 components at once. Seed small,
hand off a growth recipe.

## Step C — Keep Figma linked in the catalog

- Storybook: @storybook/addon-designs embeds the Figma frame beside each story via a design parameter with the Figma URL — designers can compare implementation vs. design in one screen. Check the addon's compatibility with the installed Storybook major before adding; it's community-maintained and can lag major releases. Record each component's Figma URL in its story regardless (even just in the description) so the link back to design is never lost.
- Widgetbook: Figma comparison is a Widgetbook Cloud (paid) feature — mention it, don't set it up unless asked. At minimum, note the Figma URL in a comment above each use-case.
- If the team uses Code Connect, offer to set up mappings for the generated components so future Figma MCP output reuses them instead of regenerating.

## Guardrails

- Tokens before components, always.
- No raw hex/px values in generated components — tokens only.
- One component at a time, verified against get_screenshot, story written immediately.
- Show extracted tokens to the user for review before generating components.
- Respect the general skill guardrails: branch, no production-build pollution, plain-language communication.
