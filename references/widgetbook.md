# Widgetbook setup (Flutter)

Widgetbook is Flutter's equivalent of Storybook: a companion app that renders your
widgets in isolation. Current major version: **3.x** (verify latest on pub.dev before
pinning: https://pub.dev/packages/widgetbook).

## Recommended architecture: separate companion app

Widgetbook's docs recommend creating a **separate Flutter app inside the repo** rather
than embedding the catalog in the main app. This keeps catalog dependencies out of the
production build and lets the catalog run on web even if the app is mobile-only.

```
your_app/
├── pubspec.yaml          # the real app
├── lib/
└── widgetbook/           # the catalog (companion app)
    ├── pubspec.yaml
    └── lib/
```

## Step-by-step

### 1. Create the companion app

From the repo root:

```bash
flutter create widgetbook --empty --platforms=web,macos
```

(web is the main target; add macos/windows/linux if the team wants a desktop
catalog. Skip mobile platforms — nobody runs a catalog on a phone.)

### 2. Rename the package to avoid a name collision

The project is named widgetbook, which collides with the pub package. In
widgetbook/pubspec.yaml, change:

```yaml
name: widgetbook_workspace
```

### 3. Add dependencies

In widgetbook/pubspec.yaml (check pub.dev for latest versions; these are known-good
as of mid-2026):

```yaml
dependencies:
  flutter:
    sdk: flutter
  widgetbook_annotation: ^3.11.0
  widgetbook: ^3.25.0
  your_app:          # the real app's package name from its pubspec.yaml
    path: ../

dev_dependencies:
  build_runner:
  widgetbook_generator: ^3.24.0
```

Then cd widgetbook && flutter pub get.

Note: the path dependency means the catalog imports widgets straight from the app —
no copying of component code, ever.

### 4. Create the entry point

widgetbook/lib/main.dart:

```dart
import 'package:flutter/material.dart';
import 'package:widgetbook/widgetbook.dart';
import 'package:widgetbook_annotation/widgetbook_annotation.dart' as widgetbook;

// Generated in step 6 — will not exist yet.
import 'main.directories.g.dart';

void main() {
  runApp(const WidgetbookApp());
}

@widgetbook.App()
class WidgetbookApp extends StatelessWidget {
  const WidgetbookApp({super.key});

  @override
  Widget build(BuildContext context) {
    return Widgetbook.material(
      directories: directories,
      addons: [
        // Filled in during theme wiring — see below.
      ],
    );
  }
}
```

### 5. Write use-cases for seed components

A use-case is a function annotated with @UseCase that returns the widget in one
state. Put use-cases in the **widgetbook app** (e.g., widgetbook/lib/usecases/),
importing components from the real app:

```dart
import 'package:flutter/material.dart';
import 'package:widgetbook_annotation/widgetbook_annotation.dart' as widgetbook;
import 'package:your_app/components/primary_button.dart';

@widgetbook.UseCase(name: 'Default', type: PrimaryButton)
Widget defaultPrimaryButton(BuildContext context) {
  return PrimaryButton(label: 'Continue', onPressed: () {});
}

@widgetbook.UseCase(name: 'Disabled', type: PrimaryButton)
Widget disabledPrimaryButton(BuildContext context) {
  return const PrimaryButton(label: 'Continue', onPressed: null);
}
```

Use **knobs** where a designer would want to fiddle with a value live:

```dart
@widgetbook.UseCase(name: 'Playground', type: PrimaryButton)
Widget playgroundPrimaryButton(BuildContext context) {
  return PrimaryButton(
    label: context.knobs.string(label: 'Label', initialValue: 'Continue'),
    onPressed: context.knobs.boolean(label: 'Enabled', initialValue: true)
        ? () {}
        : null,
  );
}
```

### 6. Generate the directory tree

```bash
cd widgetbook
dart run build_runner build --delete-conflicting-outputs
```

This creates lib/main.directories.g.dart with the directories variable. Re-run it
whenever use-cases are added/renamed (or use watch instead of build during active
work). Put this command in the designer-facing README — it's the #1 "why doesn't my
new component show up" answer.

### 7. Wire the app's theme

Find the app's ThemeData (grep for ThemeData(, theme:, a theme/ or tokens
directory). Then add the theme addon:

```dart
addons: [
  MaterialThemeAddon(
    themes: [
      WidgetbookTheme(name: 'Light', data: YourAppTheme.light),
      WidgetbookTheme(name: 'Dark', data: YourAppTheme.dark),
    ],
  ),
  DeviceFrameAddon(devices: [Devices.ios.iPhone13, Devices.android.samsungGalaxyS20]),
  TextScaleAddon(min: 0.85, max: 2.0),
],
```

If the app only has one theme, wire that one; don't invent a dark theme. If the app
uses Cupertino or a fully custom theme system, use Widgetbook.cupertino /
ThemeAddon<T> with a custom builder respectively.

### 8. Run and verify

```bash
cd widgetbook
flutter run -d chrome        # interactive check
```

For a headless/CI-friendly verification (no display available):

```bash
flutter build web            # must complete without errors
flutter analyze              # must be clean
```

## Manual alternative (no codegen)

Older tutorials (including Flutter's "Package of the Week" video) build the catalog
tree by hand instead of using the generator: pass directories a manually constructed
list of WidgetbookCategory → WidgetbookComponent → WidgetbookUseCase objects, and
skip widgetbook_annotation, widgetbook_generator, and build_runner entirely. This
works and is fine for a ~5-component catalog, but doesn't scale — every new component
means hand-editing the tree. Default to the generator approach above. If the repo
already has a hand-built catalog, extend it in the same style rather than mixing both.

## Hosting the catalog for the team

flutter build web in the widgetbook app produces a static site in build/web/ that
can be hosted anywhere. If deploying, follow the SKILL.md deployment section: never
public. A common Flutter-shop pattern is Firebase Hosting with a sign-in gate
restricted to the company Google Workspace domain (a small auth shell that only loads
the catalog after domain-verified sign-in); Widgetbook Cloud is the managed
alternative with built-in team access and Figma review features.

## Known pitfalls

- **Name collision**: forgetting step 2 causes a confusing "widgetbook depends on
-   itself" pub error.
-   - **Generator finds nothing**: use-cases must be inside the widgetbook app's lib/,
    -   not the main app, and the file must import widgetbook_annotation.
    -   - **Missing main.directories.g.dart**: the entry point won't compile until
        -   build_runner has run once. Run the generator before the first flutter run.
        -   - **Monorepos (melos)**: put the widgetbook app alongside the other packages and path-
            -   depend on the design-system package, not the app shell.
            -   - **App widgets that require providers** (Riverpod/Bloc/Inherited widgets): wrap the
                -   use-case in the minimal provider with fake data, or skip the component for the seed
                -     set. Never boot the app's real dependency injection in the catalog.
                - 
