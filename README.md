# Versionize

A tool that *(somewhat)* securely evaluates the version of a project.


## Background

In a perfect world only the chosen CI-tool would build/compile a software
and sets the version-info correctly during such build-run.  
Unfortunately there always be some reason to quickly build/compile
a software locally.
In such cases setting the correct version often gets overlooked and
e.g. in a public demo, customers may spot an incorrect version.
Therefore I created this project to (somewhat) reliably determine
the actual software version.


### The power of GIT

Without GIT, this project wouldn't make much sense, since there are many other libraries out there
to determine a version from any project meta file. They were just added to keep projects working,
if e.g. someone decides to download a ZIP on a machine without GIT installed.


## Features

This tool works as follow:

1. Check if [git](https://git-scm.com/) is installed on the local system
2. Try to fetch the latest state, if the current or the next ancestor is a valid git-project and base the response on that
3. If git isn"t available or no repo can"t be found, try to find and parse `package.json` for a version response
4. If no "package.json" can"t be found, try to find and parse a `composer.json for a version response
5. If no `composer.json` can"t be found, try to find and parse a `pom.xml` for a version response
6. Finally try to determine the version by the content of a `VERSION` file (case-insensitive, with a possible `.txt` file-ending)
7. If everything fails, set the version to `null`


### Options

- `skip` ***(Array=)***: An optional array for skipping any source.  
  *Valid options are: `"git"`, `"package.json"`, `"composer.json"`, `"pom.xml"` and `"file"`*
- `update` ***(Array=)***: An optional array to populate the determined version-info into one or more files.  
  *Valid options are: `"package.json"`, `"composer.json"`, `"pom.xml"` or any file-name (e.g. `VERSION.txt`)*
- `writeVersionPrefix` ***((Array=))***: If the determined version is an actual release,
   ensure a "v"-prefix (e.g. "v1.3.2") is added to any of the specified files.
   *Valid options are: `"response"`, `"package.json"`, `"composer.json"`, `"pom.xml"` and/or `"file"`*


### Response

The response consists of an object containing following key/values:

- `"version"` **(String=)**: Version on GIT-based project is determined as follow:
  - When the current commit matches a release-tag: The release-tag without a version-prefix, even if the tag itself has one, since most meta-files don't support it.
  - Otherwise it tries to stumble the info together: `<CURRENT_BRANCH_NAME>_<PREVIOUS_TAG>-<CURRENT_DATETIME_STAMP>git<CURRENT_COMMIT_SHA>`
- `"isClean"` **(Boolean=)**: `true` when everything has been added, committed and pushed, `false` if not and `undefined` if GIT is not installed
- `"isRelease"` **(Boolean=)**: `true` when latest commit match a [GIT tag](https://git-scm.com/book/en/v2/Git-Basics-Tagging), `false` if not and `undefined` if GIT is not installed
- `"commitSHA"` **(String=)**: The current (short) commit SHA-1 or `undefined` if GIT is not installed
- `"fullCommitSHA"` **(String=)**: The full commit SHA-1  or `undefined` if GIT is not installed
- `"branch"` **(String=)**: The currently checked out branch  or `undefined` if GIT is not installed
- `"source"` **(String=)**: Information from where the version-info was determined.
  Can either be: "git", "package.json", "composer.json", "pom.xml", "file" or `undefined` if it couldn't be determined.


## Usage - Example

```js
const versionize = require('versionize')

// Assumption:
// - we're on the 'feat/assimilate' branch
// - everything has been committed and pushed
// - the latest release in the main-branch was `v1.0.3`
const versionInfo = versionize.determine({
 skip: ["pom.xml", "composer.json"], // skip Java, PHP info, since we don't use any
 update: [
    "package.json", // Will write to the package.json or throw an error if can't be found.
    "version" // will create/update a file called `version` containing one line with the determined version
  ]
})

// --- Sample Response ------------------------------------
versionizeResponse = {
  version: "feat/assimilate_1.0.3-202204111448gitb412af3", // snapshot version-info, since latest: latest-tag !== commit
  isClean: true, // Info if there are uncommitted or non pushed files
  isRelease: false, // Info if the latest tag matches the current commit
  commitSHA: "b412af3", // short commit SHA-1
  fullCommitSHA: "b412af333e500999495f5d7780d59d6920021728", // full commit SHA-1
  branch: "feat/assimilate", // current GIT branch
  source: "git" // can either be: "git", "package.json", "composer.json", "pom.xml" or "file"
}
// ========================================================
```
