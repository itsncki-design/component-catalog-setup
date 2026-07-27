---
name: component-catalog-setup
description: >
  Set up a component catalog (Widgetbook for Flutter, Storybook for React/Vue/Svelte/Angular/web)
    in an existing codebase so designers and developers can browse reusable UI components in
      isolation. Use this skill whenever the user mentions Widgetbook, Storybook, a "component
        library", "component catalog", "design system viewer", "UI kit", "component playground",
          wanting to "see all our components in one place", or wanting to preview/test/document
            reusable components outside the running app — even if they don't name a specific tool.
              Also use when the user wants to build the catalog FROM a Figma design system — i.e.,
                components/tokens live in Figma and need to be turned into coded components first
                  ("build our design system from Figma", "generate components from our Figma file").
                    The skill detects which framework the codebase uses and picks the right tool automatically,
                      so trigger it even when the user doesn't know whether they need Widgetbook or Storybook.
                      ---

                      # Component Catalog Setup

                      Set up a component catalog in an existing codebase. A component catalog is an isolated
                      environment where every reusable UI component can be viewed in its different states
                      (default, hover, disabled, empty, error, dark mode...) without running the full app.

                      Two tools, chosen by stack:

                      | Stack | Tool | Reference file |
                      |---|---|---|
                      | Flutter / Dart | Widgetbook | references/widgetbook.md |
                      | React, Vue, Svelte, Angular, Next.js, SvelteKit, Nuxt, web components, HTML | Storybook | references/storybook.md |
                      | React Native | Storybook (React Native variant) | references/storybook.md (see RN section) |

                      The user is often a designer, not an engineer. Explain what you're doing in plain
                      language, avoid unexplained jargon, and never assume they know git/npm/pub mechanics.
                      When something needs a decision, present it as a design-relevant choice ("do you want
                      the catalog to show your app's dark theme?") rather than a technical one.

                      ## Workflow

                      ### Step 0 — Where do the components live?

                      Two possible sources of truth:

                      - Code-first (components already exist in the codebase) → proceed to Step 1.
                      - Figma-first (components/tokens live in a Figma design system; code has few or no
                        components yet, or the user says "from Figma" / "from our design system file") →
                          read references/figma.md FIRST. It covers extracting tokens and generating the
                            initial components from Figma. The catalog tool is still chosen by code stack in
                              Step 1 — Figma changes where components come from, not which catalog you install.
                              - Mixed (some components coded, more only in Figma) → catalog what exists via the
                                normal path, then use references/figma.md to grow the set.

                                ### Step 1 — Detect the stack

                                Scan the repo root (and one level down for monorepos). Do not ask the user what
                                framework they use — detect it:

                                - pubspec.yaml with a flutter: section → Flutter → Widgetbook
                                - package.json → inspect dependencies/devDependencies:
                                  - react-native (without react-dom) → React Native → Storybook RN
                                    - react, vue, svelte, @angular/core, next, nuxt → Storybook
                                    - Both pubspec.yaml and package.json (e.g., Flutter app + docs site) → the one
                                      containing the UI components wins; check where components actually live, then confirm
                                        with the user.
                                        - Monorepo (melos.yaml, pnpm-workspace.yaml, turbo.json, packages/, apps/):
                                          identify the package that holds the shared/reusable components — that's where the
                                            catalog attaches. Confirm the target package with the user before proceeding.
                                            - Nothing recognizable → tell the user what you found and ask where the UI code lives.

                                            Also check whether a catalog already exists (.storybook/ directory, a widgetbook/
                                            sub-app, widgetbook in pubspec, storybook in package.json). If so, say so and switch
                                            to improving/extending it instead of re-installing.

                                            ### Step 2 — Confirm the plan (briefly)

                                            Before touching anything, tell the user in 2–4 sentences:

                                            - which tool you'll install and why (detected stack),
                                            - where it will live (e.g., "a small companion app in a widgetbook/ folder" or
                                              "a .storybook/ config folder plus story files next to your components"),
                                              - that you'll work on a new git branch so nothing touches their main code until reviewed.

                                              Then create a branch: git checkout -b setup-component-catalog (if the repo uses git;
                                              if not, proceed but say changes are unversioned).

                                              ### Step 3 — Install and configure

                                              Read the reference file for the detected tool now — it has the exact current
                                              commands, dependency versions, file templates, and known pitfalls:

                                              - Flutter → read references/widgetbook.md
                                              - Everything else → read references/storybook.md

                                              Follow it. Do not improvise install commands from memory; versions and CLIs change.

                                              ### Step 4 — Wire up the app's real theme

                                              A catalog that renders components in a default Material/plain theme is useless to a
                                              designer. Find the app's theme (ThemeData, design tokens, global CSS, Tailwind config,
                                              theme provider) and make the catalog render components inside it. If the app has
                                              light + dark themes, wire both and add the theme-switching addon so the user can toggle.
                                              Details are in the tool reference files.

                                              ### Step 5 — Seed it with real components

                                              An empty catalog convinces nobody. Find 3–5 real components from the codebase to seed:

                                              - Prefer leaf components with simple props: buttons, chips, badges, tags, inputs,
                                                cards. Avoid screens/pages and anything needing network, routing, or app state.
                                                - For each, create stories/use-cases covering its meaningful states (variants, disabled,
                                                  long-text overflow, empty). 2–4 states per component is plenty for the seed.
                                                  - If a component needs data, pass realistic hardcoded sample data — never wire real
                                                    services into the catalog.

                                                    ### Step 6 — Verify it runs

                                                    Actually run the catalog (npm run storybook / flutter run in the widgetbook app —
                                                    see references for details, including headless/CI-friendly checks when there's no
                                                    display). Fix errors before declaring success. A skill run that ends with broken setup
                                                    is worse than no run.

                                                    ### Step 7 — Hand off

                                                    Finish with:

                                                    1. Designer-facing docs: how to start the catalog, how to add a new component (with a
                                                       copy-pasteable template), and the regenerate command (Widgetbook) if applicable.
                                                          Put these where the repo already keeps workflow docs — an existing docs/
                                                             folder, a tooling/ or skills directory, or a README section. Only create a new
                                                                CATALOG.md when no established docs location exists. Docs nobody finds are docs
                                                                   that don't exist; match the team's conventions rather than imposing a new file.
                                                                   2. A plain-language summary in chat: what was installed, how to launch it, which
                                                                      components are in it, and suggested next components to add.
                                                                      3. If the repo uses git: commit on the branch with a clear message and tell the user
                                                                         the branch name so an engineer can review/merge (or open a PR if that's the
                                                                            team's workflow and tooling is available).
                                                                            4. Mention natural next steps without doing them uninvited: growing coverage
                                                                               component-by-component, and optionally a Prototypes/ section (Storybook titles)
                                                                                  or prototypes folder (Widgetbook) where designers stage speculative states of
                                                                                     real components before they're specced — the catalog doubles as a prototyping
                                                                                        sandbox. Ongoing design→spec→implement workflows are out of scope for this setup
                                                                                           skill and belong in a companion skill.

                                                                                           ### If the catalog will be deployed (not just run locally)

                                                                                           The default assumption is a locally run catalog. If the user wants it hosted so the
                                                                                           team can browse without running code, treat access control as **mandatory, not
                                                                                           optional**: a component catalog exposes unreleased UI, feature names, and product
                                                                                           direction. Never deploy one publicly.

                                                                                           - Both tools build to static sites (npm run build-storybook → storybook-static/;
                                                                                             flutter build web in the widgetbook app → build/web/), so any static host works.
                                                                                             - Gate it with company-restricted access: e.g., Firebase Hosting fronted by Google
                                                                                               sign-in restricted to the company email domain, Cloudflare Access, or Vercel
                                                                                                 password protection — or the tools' hosted products (Chromatic for Storybook,
                                                                                                   Widgetbook Cloud), which ship with team access control built in.
                                                                                                   - The auth gate is real engineering work: propose the approach, confirm with the
                                                                                                     user, and involve their engineer for identity-provider specifics rather than
                                                                                                       improvising security configuration.
                                                                                                       
                                                                                                       ## Guardrails
                                                                                                       
                                                                                                       - Version numbers in the reference files are known-good fallbacks, not gospel. Before
                                                                                                         installing, check what's current (npm view <pkg> version, pub.dev) and — if a
                                                                                                           catalog or related tooling already exists in the repo — match the versions already
                                                                                                             in use there instead of introducing new ones.
                                                                                                             - Never modify the components themselves while setting up the catalog. If a component
                                                                                                               can't be rendered in isolation (hard dependency on app state), skip it, note why, and
                                                                                                                 pick a simpler one.
                                                                                                                 - Never add the catalog to the production build (keep Storybook deps in
                                                                                                                   devDependencies; keep the Widgetbook app out of the main app's release targets).
                                                                                                                   - Keep all changes on the branch; never commit directly to main/master.
                                                                                                                   - If install commands fail from network restrictions or missing SDKs, stop and explain
                                                                                                                     exactly what's missing and what command to run where — don't half-install.
                                                                                                                     
