# easybook-app-build
Repository used to build &amp; deploy easybook app

Release jobs use Node.js 22, configured once in `.github/workflows/build.yml`.
The workflow controls its runtime independently of `.node-version` in each shop
branch: older branches pin Node 20.14.0, which cannot install current transitive
dependencies of EAS CLI 16.20.0. The same runtime is used for iOS, Android and
first-time Play Store creation.

After changing the workflow, start a new `workflow_dispatch` on `master` with
the intended shop branch. Re-running an older failed run reuses its original
workflow revision. A workflow-only retry does not require a new app version when
the previous attempt failed before building or submitting either platform.

Before `deliver` runs, `Submit iOS` prepares
`source/fastlane/metadata/ios_rating_config.json` with the two social-media
declarations required by App Store Connect:

```json
{
  "socialMedia": false,
  "socialMediaAgeRestricted": false
}
```

These defaults describe the standard Easybook booking app, which has external
social links but no built-in social feed. A customized shop must declare its
actual values in its rating JSON. The workflow preserves explicit values and
all other age-rating answers, and rejects nulls, strings or other non-boolean
values before contacting Apple. Only missing keys receive the defaults.
This preparation also covers existing shop branches without merging or pushing
each branch, and does not change the IPA.

The error `missing a required attribute ... socialMedia` comes from the age-rating
metadata request. Fastlane 2.242.0 already supports both fields; updating the gem
alone does not add missing answers to the shop's JSON. See the
[Fastlane maintainer's explanation](https://github.com/fastlane/fastlane/issues/30499)
and [Apple's attribute definitions](https://developer.apple.com/documentation/appstoreconnectapi/ageratingdeclarationupdaterequest/data-data.dictionary/attributes-data.dictionary).

Re-running an existing failed run uses its original workflow revision and the
source commit saved with the IPA, so it will not pick up this workflow fix.
Before retrying a partially completed release, check whether the binary was
already uploaded and reuse the existing build where possible; an age-rating
metadata correction alone does not require rebuilding the app.
