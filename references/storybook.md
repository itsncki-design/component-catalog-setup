# Storybook setup (React, Vue, Svelte, Angular, web)

Storybook is the standard component catalog for JS frameworks. Current major version:
10.x (ESM-only). Verify the latest with npm view storybook version before
telling the user what they're getting.

## Step-by-step

### 1. Initialize with the official CLI — don't hand-roll

From the package that contains the components (repo root for a single app; the shared
UI package in a monorepo):

```bash
npm create storybook@latest
```

(Or pnpm create storybook@latest / yarn create storybook — match the lockfile:
pnpm-lock.yaml → pnpm, yarn.lock → yarn, package-lock.json → npm.)

The CLI auto-detects the framework, renderer, and builder. Useful flags:

- --yes to accept defaults and skip prompts
- --no-dev to prevent the CLI from launching the dev server after install — always pass this when running as an agent, or the init blocks your shell
- --features docs test to preselect features non-interactively
- --type react|vue3|angular|svelte|sveltekit|nextjs|nuxt|web_components|... if auto-detection fails
- --package-manager=npm|pnpm|yarn to force one
- Skip the onboarding tutorial when prompted — this is a team setup, not a tutorial.

Verified non-interactive invocation:

npm create storybook@latest -- --yes --no-dev --features docs

What it creates: .storybook/ (config: main.ts, preview.ts), example stories in
src/stories/, and storybook/build-storybook scripts in package.json. **Delete
the example stories** (src/stories/) after seeding real ones — placeholder Button/
Header stories confuse designers about what's actually in the design system.

### 2. Prefer the agentic setup when available

On React + Vite projects, Storybook ships an agent-oriented setup:

```bash
npx storybook ai setup
```

This prints project-specific instructions (providers to wrap, global CSS to import,
portals, mocking) generated from analyzing the codebase. If the command exists and the
project qualifies, run it, read the output, and follow it — it encodes exactly the
"wire the real theme and providers" work described below. On other renderers/builders,
do that work manually per steps 3–4.

### 3. Wire global styles and theme

In .storybook/preview.ts:

- Import the app's global CSS (e.g., import '../src/index.css') — without this, Tailwind/reset/tokens are missing and every component looks broken.
- Wrap stories in the app's providers via a decorator: theme provider, i18n, router mock — whatever leaf components need to render. Keep it minimal; don't boot the app's real store or API clients.

```ts
import type { Preview } from '@storybook/react-vite'; // match your framework package
import '../src/index.css';
import { ThemeProvider } from '../src/theme';

const preview: Preview = {
  decorators: [
    (Story) => (
      <ThemeProvider>
        <Story />
      </ThemeProvider>
    ),
  ],
};

export default preview;
```

- Dark mode: if the app has one, expose it. With CSS-class-based themes use @storybook/addon-themes' withThemeByClassName; with provider-based themes use withThemeFromJSXProvider. Add a toolbar toggle rather than duplicating every story.

### 4. Seed stories for real components (CSF3)

Stories live next to components as ComponentName.stories.tsx (framework-appropriate
extension). CSF3 format:

In Storybook 10, import types from the framework package the init installed
(@storybook/react-vite, @storybook/nextjs, @storybook/vue3-vite, ...), not the
bare renderer — match whatever .storybook/main.ts uses:

```tsx
import type { Meta, StoryObj } from '@storybook/react-vite';
import { Button } from './Button';

const meta = {
  title: 'Components/Button',
  component: Button,
  tags: ['autodocs'],
} satisfies Meta<typeof Button>;

export default meta;
type Story = StoryObj<typeof meta>;

export const Primary: Story = { args: { variant: 'primary', children: 'Continue' } };
export const Disabled: Story = { args: { disabled: true, children: 'Continue' } };
export const LongLabel: Story = {
  args: { children: 'A very long label that should truncate or wrap gracefully' },
};
```

Title convention: Components/... for leaf components, Patterns/... for composites.
Add tags: ['autodocs'] so each component gets an auto-generated docs page — designers
love this.

### 5. Verify it runs

```bash
npm run storybook          # interactive; serves on :6006
```

Headless/CI-friendly check (no display):

```bash
npm run build-storybook    # must complete without errors
```

If dev errors are cryptic, run the build — its errors are usually more legible. Also
npx storybook doctor diagnoses duplicate/mismatched dependencies.

## Framework-specific notes

- Next.js: the CLI installs @storybook/nextjs; next/image, next/font, and the router are mocked automatically. App-router server components can't be rendered directly — story the client components.
- SvelteKit / Nuxt: dedicated frameworks exist (@storybook/sveltekit, @storybook-vue/nuxt); the CLI picks them. $app/* / Nuxt auto-imports are shimmed.
- Angular: stories go in .stories.ts; the CLI wires angular.json. Standalone components work directly; NgModule-era components need moduleMetadata decorator.
- Monorepo: install in the shared UI package. If stories must span packages, add their globs to stories: in .storybook/main.ts.
- React Native: different beast — uses @storybook/react-native, renders on-device. Init: npx storybook@latest init in the RN project detects RN and scaffolds .rnstorybook/, then you conditionally render Storybook's UI from the app entry point (commonly behind an env flag). For Expo there's a ready template: npx create-expo-app --template expo-template-storybook. If the team mainly wants a browser-viewable catalog, consider react_native_and_rnw (RN + react-native-web) so the same stories also run in a web Storybook.

## Known pitfalls

- Storybook 10 is ESM-only: .storybook/main.ts must be valid ESM. On older repos with CommonJS configs, the CLI handles it, but hand-edits using module.exports will break.
- Missing global CSS is the most common "everything renders unstyled" bug — always import it in preview.ts.
- Version alignment: all @storybook/* packages and storybook itself must be on the same major.minor. npx storybook doctor catches drift.
- Webpack 4 projects are unsupported; flag this to the user rather than fighting it.
- Components with data fetching: mock at the preview level (MSW via msw-storybook-addon) rather than per-story hacks — but for the initial seed, just pick components that don't fetch.
