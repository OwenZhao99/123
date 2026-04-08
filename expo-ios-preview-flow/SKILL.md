---
name: expo-ios-preview-flow
description: Use when working on an Expo or React Native iOS app and the user wants code changes implemented, validated, and delivered with a working preview path. Trigger for requests like "改完直接给我预览链接", "直接推 TestFlight", "用 development build 给我看", or any Expo iOS iteration where preview speed matters.
metadata:
  short-description: Implement Expo iOS changes and hand back a preview link
---

# Expo iOS Preview Flow

## Overview

This skill standardizes Expo iOS development so the agent does not stop at code changes. It implements the request, validates the app, chooses the fastest viable preview path, and returns the exact link or command the user needs on iPhone.

Use this only for Expo/EAS-based iOS projects. If the repo is native Swift/Xcode or Flutter, say this skill does not apply and fall back to the correct project workflow.

## Default Outcome

For every qualifying request, do all of the following unless the user explicitly narrows scope:

1. Implement the requested change.
2. Run repo validation that fits the project, preferring existing project commands for typecheck, tests, and lint.
3. Choose the fastest preview path that is actually valid for the change.
4. Return the preview link, build link, or exact install/open step instead of only describing the code change.

## Workflow

### 1. Confirm this is the right project shape

Before editing, inspect the repo for Expo/EAS signals such as:

- `app.json` or `app.config.*`
- `eas.json`
- `expo` in `package.json`
- scripts like `expo start`, `eas build`, or `eas update`

If these signals are missing, do not force this workflow.

### 2. Implement the feature

Make the requested code changes first. Do not stop at a plan unless the user explicitly asked for planning only.

### 3. Detect package manager and scripts

Prefer the package manager already used by the repo.

Use this detection order:

- `pnpm-lock.yaml` -> `pnpm`
- `yarn.lock` -> `yarn`
- `bun.lock` or `bun.lockb` -> `bun`
- otherwise `npm`

Prefer project scripts from `package.json`. If the repo already defines wrappers like `dev:client` or `build:ios:development`, use them. If not, derive equivalent `expo` / `eas` commands directly.

### 4. Validate locally

Prefer project-native validation commands. For an Expo app, typical examples are:

```bash
pnpm check
pnpm test
pnpm lint
```

Use the detected package manager instead of hardcoding `pnpm` when needed. If a command does not exist, use the closest equivalent and say what was run.

### 5. Pick the preview path

Use this decision order.

#### Path A: Existing development build for JS/UI changes

Use this when the changes are limited to JavaScript, TypeScript, styles, routing, copy, layout, or other non-native logic.

Preferred commands are project wrappers such as:

```bash
pnpm dev:client
```

Equivalent direct fallback:

```bash
npx expo start --dev-client
```

If LAN discovery fails or the phone is on a different network, prefer the repo wrapper if it exists. Otherwise use:

```bash
npx expo start --dev-client --tunnel
```

After starting the dev server:

- read the terminal output
- capture the Expo development URL or deep link
- give the user the exact URL to enter or tap in the installed development build
- if the app is already installed, prefer this path over any rebuild

#### Path B: Rebuild the development client

Use this when native state changed, for example:

- new native dependency
- changed Expo plugin set
- changed `app.config.*`
- changed permissions, icons, splash, bundle identifiers, or other binary-affecting config

Preferred repo wrapper:

```bash
pnpm build:ios:development
```

Equivalent direct fallback:

```bash
npx eas-cli build --platform ios --profile development
```

Then:

- monitor the EAS build until it finishes or reaches a stable remote state
- return the install link and the EAS build page
- once installed, resume Path A for day-to-day previewing

#### Path C: Preview/internal distribution build

Use this when the user wants a more stable installable binary but does not need App Store/TestFlight distribution.

Preferred wrapper:

```bash
pnpm build:ios:preview
```

Equivalent direct fallback:

```bash
npx eas-cli build --platform ios --profile preview
```

For JS-only follow-up changes, prefer a wrapper such as:

```bash
pnpm update:preview
```

Or fall back to:

```bash
npx eas-cli update --channel preview --auto
```

#### Path D: Production/TestFlight

Use this only when the user explicitly asks for TestFlight, App Store submission, or a release-like preview.

Preferred wrapper:

```bash
pnpm build:ios:production
```

Equivalent direct fallback:

```bash
npx eas-cli build --platform ios --profile production
```

If the repo is already configured for submit, continue with the project's EAS submit flow and return the App Store Connect / TestFlight location.

## Preview Rules

- Default to the fastest path that preserves correctness.
- Do not send the user to TestFlight for a pure UI tweak if a development build is already installed.
- Do not use `EAS Update` when the change requires a new native binary.
- If the installed dev build is enough, return a live preview path in the same turn.
- When the user asks for "preview link", they mean something they can open now on iPhone, not just a build log.

## What To Return

For development build preview:

- say which command started the dev server
- provide the exact URL or deep link from the running server
- tell the user whether to open `Home`, `Enter URL manually`, or just tap the discovered server

For build-based preview:

- provide the install artifact link
- provide the EAS build page
- if relevant, provide the TestFlight or App Store Connect page

Always include a short note if the user must do a manual step such as enabling iPhone Developer Mode or registering the device.

## Practical Defaults

When this skill is used in a repo that already has wrappers like the examples below, prefer them instead of rebuilding the command from scratch:

```bash
pnpm dev:client
pnpm dev:client:tunnel
pnpm build:ios:development
pnpm build:ios:preview
pnpm build:ios:production
pnpm update:preview
pnpm update:production
```

If these scripts do not exist, derive the equivalent Expo/EAS commands from `package.json` and `eas.json`.

## Failure Handling

If preview is blocked, report the specific blocker and keep pushing the chain forward.

Examples:

- missing `expo-dev-client`: install it and continue
- missing `expo-updates` but preview channels are expected: install/configure it and continue
- missing Apple device registration: generate the device registration link and tell the user to complete it on the phone
- build queue delay: keep monitoring and return the live status link
- LAN connection failure: switch to tunnel instead of stopping

Do not just say "I couldn't preview it" unless every viable path has been exhausted.

## Trigger Examples

Use this skill for requests like:

- `实现这个需求，改完后直接启动预览并把链接发我`
- `直接给我 iPhone 预览链接`
- `推一版 TestFlight 我看看`
- `给我 development build`
- `改完别只说代码，直接让我能在手机上看`
