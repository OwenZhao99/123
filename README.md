# Expo iOS Preview Flow

A reusable Codex skill for Expo / React Native iOS projects.

It turns a vague request like "implement this and give me an iPhone preview link" into a concrete workflow:

1. implement the code change
2. run local validation
3. choose the fastest valid preview path
4. hand back the exact preview link, install link, or TestFlight location

## What the skill handles

- Expo + EAS project detection
- package manager detection: `pnpm`, `yarn`, `bun`, or `npm`
- local validation using project scripts
- Expo development build preview
- fallback to tunnel mode when LAN discovery fails
- EAS development / preview / production iOS builds
- EAS Update for JS-only follow-up changes
- TestFlight handoff when the user explicitly asks for it

## Decision model

The skill prefers the fastest path that preserves correctness.

- JS / UI only change + existing development build: start the dev server and return the live development URL
- native-affecting change: rebuild the iOS development client
- user wants a more stable installable build: use preview/internal distribution
- user wants release-like distribution: use production/TestFlight

## Install

Copy or symlink the `expo-ios-preview-flow` folder into your Codex skills directory.

Typical location:

```bash
~/.codex/skills/expo-ios-preview-flow
```

If you cloned this repository somewhere else, one simple install method is:

```bash
mkdir -p ~/.codex/skills
ln -s "$(pwd)/expo-ios-preview-flow" ~/.codex/skills/expo-ios-preview-flow
```

If you already have a folder at that path, remove or rename it first.

## Use

Explicit invocation:

```text
Use $expo-ios-preview-flow to implement this change and give me a preview link.
```

Natural-language prompts that should also trigger it:

- `实现这个需求，改完后直接启动预览并把链接发我`
- `直接给我 iPhone 预览链接`
- `给我 development build`
- `推一版 TestFlight 我看看`

## Project assumptions

This skill is intentionally scoped to Expo / React Native iOS apps that use Expo tooling.

Expected repo signals include:

- `app.json` or `app.config.*`
- `eas.json`
- `expo` in `package.json`

If the project is native Swift/Xcode or Flutter, this skill should not be used.

## Recommended repo scripts

The skill works best when the project already exposes wrapper scripts such as:

```json
{
  "scripts": {
    "dev:client": "expo start --dev-client",
    "dev:client:tunnel": "expo start --dev-client --tunnel",
    "build:ios:development": "eas build --platform ios --profile development",
    "build:ios:preview": "eas build --platform ios --profile preview",
    "build:ios:production": "eas build --platform ios --profile production",
    "update:preview": "eas update --channel preview --auto",
    "update:production": "eas update --channel production --auto"
  }
}
```

If these scripts are missing, the skill falls back to direct `expo` / `eas` commands.

## Files

- `expo-ios-preview-flow/SKILL.md`: the skill definition
- `expo-ios-preview-flow/agents/openai.yaml`: UI metadata for Codex skill lists and chips

## Notes

- A development build is still the fastest day-to-day preview path for iPhone work.
- `EAS Update` should only be used when the change does not require a new native binary.
- For pure native iOS projects, create a separate skill instead of stretching this one.
