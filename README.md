# sync-e2e-app

A repository shaped like a real Next.js application, used to exercise Better
i18n's sync end to end. It is not a fixture grid — `i18n-sync-fixtures` already
covers isolated layouts. This one exists for the paths that only run against a
repository the platform has never seen, and for the ordinary shape a customer
actually arrives with.

## What makes it a realistic test

| Property | Value | Why it is here |
|---|---|---|
| Path | `apps/web/src/i18n/locales` | Monorepo depth, not a top-level `locales/` |
| Structure | locale folders — `{locale}/{namespace}.json` | next-intl / next-i18next default |
| Locale tokens | `en-US`, `ja-JP`, `zh-TW` | Region-suffixed, BCP-47 cased |
| Key format | nested objects | What most apps ship; importing it as flat flattens the customer's files |
| Namespaces | `common`, `auth` | More than one, so namespace routing is real |
| Coverage | deliberately incomplete | `ja-JP` misses `errors.rateLimited`; `zh-TW` has no `auth.json` at all |
| Noise | `package.json`, `tsconfig.json`, `apps/web/src/i18n/config.json` | None of these may be read as a locale |

## Expected detection

Connecting with source language `en` should propose, without anyone correcting it:

```
path        apps/web/src/i18n/locales
structure   locale folders
key format  nested
languages   en-US, ja-JP, zh-TW
source      apps/web/src/i18n/locales/en-US/{common,auth}.json   (matched, not exact)
new fr      apps/web/src/i18n/locales/fr-FR/common.json          (not fr-fr, not fr)
```

`en` reaching `en-US/` is CLDR maximization, and `zh-hant` reaching `zh-TW/` is
an alias — neither is a guess, and both are the direction RFC 4647 Lookup does
not cover.

## Rules

- **Do not "fix" the incomplete translations.** The gaps are the test.
- **Do not rename the locale folders.** Their casing is what proves the naming
  policy is read from the repository rather than assumed.
- Changes made by a sync arrive as pull requests. Read the file list against the
  expectation before merging anything — a green sync job that wrote the wrong
  file looks identical to a correct one.
