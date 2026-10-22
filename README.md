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
