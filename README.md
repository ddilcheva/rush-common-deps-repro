Issue with Rush update:

## Env
Rush version 5.61.4
pnpm version pnpm-6.7.1

## Issue
Preferred version is not respected. 
When a package specifies a range, 
the latest version is installed instead of the preferred version 
(which is lower but within the range).

## Expected behavior
From the docs, this is how `preferredVersions` should work:

```
  "preferredVersions": {
    /**
     * When someone asks for "^1.0.0" make sure they get "1.2.3" when working in this repo,
     * instead of the latest version.
     */
    // "some-library": "1.2.3"
  },
```
https://rushjs.io/pages/configs/common-versions_json/

## Repro
Project "a" depends on `core-js@^3.0.0`
preferredVersion is `"core-js": "3.6.5"`
installed version is `core-js: 3.21.0`

### 
Steps I took to install `core-js`:

$ cd a
$ rush add -p core-js@^3.0.0

<!-- pnpm-lock.yaml -->
  ../../a:
    specifiers:
      core-js: ^3.0.0
      object.pick: ^1.3.0
    dependencies:
      core-js: 3.21.0
      object.pick: 1.3.0

add to common-versions.json

```
  "preferredVersions": {
    "core-js": "3.6.5"
  },
```

  $ rush update

I see no change to pnpm-lock.
common/config/rush/pnpm-lock.yml:  

```
../../a:
  specifiers:
    core-js: ^3.0.0
    object.pick: ^1.3.0
  dependencies:
    core-js: 3.21.0
    object.pick: 1.3.0
```

### What else I tried

### clearing cache
$ rush purge && rush update --full
-> no change

$ rm rm common/config/rush/pnpm-lock.yaml && rush update
-> no change

### switching to npm
same results, except when I try `rush install`, I get error:
`The shrinkwrap file contains the following issues:
  Missing dependency "core-js" (3.6.5) required by the preferred versions from common-versions.json`

### specifying preferredVersion *first*, then adding dependency to the package
Works correctly and I get in pnpm-lock.json

```
  ../../a:
    specifiers:
      core-js: 3.6.5
    dependencies:
      core-js: 3.6.5
```

However, if I then modify the version in preferredVersions and `npm update`, the installed dependency does not change

## Github Issue


## Summary

Preferred version is not respected. 
When a package specifies a range, 
the latest version is installed instead of the preferred version 
(which is lower but within the range).

Example from the repro repo:
Project "a" depends on `core-js@^3.0.0`
preferredVersion in common-versions.json is `"core-js": "3.6.5"`
installed version is `core-js: 3.21.0`

## Repro steps

clone https://github.com/ddilcheva/rush-common-deps-repro
```
rush install
> succeeds

rush check

> logs:
Node.js version is 14.18.2 (LTS)


Starting "rush check"

core-js
  ^3.0.0
   - a
  3.6.5
   - preferred versions from common-versions.json

Found 1 mis-matching dependencies!

rush update
> no updates 
```

  **Expected result:** `rush update` pins core-js@3.6.5 in common/rush/config/pnpm-lock.yaml

  **Actual result:** `core-js: 3.21.0` is installed

## Details

Specifying preferredVersion *first*, 
then adding dependency to the package (`rush add -p core-js@^3.0.0`)
works correctly and installs v3.6.5

## Standard questions

Please answer these questions to help us investigate your issue more quickly:

| Question | Answer |
| -------- | -------- |
| `@microsoft/rush` globally installed version? | 5.61.4 |
| `rushVersion` from rush.json? | 5.61.4 |
| `useWorkspaces` from rush.json? | no |
| Operating system? | Mac |
| Would you consider contributing a PR? | No|
| Node.js version (`node -v`)? | v14.18.2 |