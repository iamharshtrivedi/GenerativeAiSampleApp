# Plugins

This composite module houses the gradle plugins that are used in the main SDK; to assist in a 
number of monotonous tasks. You can read more on these plugins and the tasks they create below.

## ChangelogPlugin

Creates and manages changelog files. These files are used to signify changes made to the repo that 
should invoke a release, alongside text to display in the release notes at release time.

Change files are (by default) created under the `.changes` directory at the root of the repo. 
These files are json encoded variants of a [Changelog](./src/main/java/com/google/gradle/types/Changelog.kt) instance- 
which is just an organization of what impact the change had (will it invoke a patch, minor, or 
major bump?) and an (optional) end-user readable message to show alongside the other changes at 
release time. By default, the files are saved as a random sequence of four words (to avoid 
collisions).

During a release cycle, the `.changes` directory will be filled with change files. When it comes to 
release time, these changes will be combined into a single `release_notes.md` file that contains 
all the changes made- in a consumable format. After a release, the `.changes` directory will be 
wiped, and the cycle will continue.

To assist in this endeavour, the ChangelogPlugin registers an internal plugin and a few tasks:

### APIPlugin

An internal plugin (automatically added, and cannot be explicitly applied) that facilitates the 
generation of `.api` files.

#### Tasks

The APIPlugin registers the two following tasks to facilitate this process:

- [buildApi](./src/main/java/com/google/gradle/plugins/ApiPlugin.kt) -> Creates a `.api` file 
containing the public API of the project.
- [exportApi](./src/main/java/com/google/gradle/plugins/ApiPlugin.kt) -> Exports the `.api` file 
generated by the `buildApi` task to a file at the project directory named `public.api`.

### Tasks

The ChangelogPlugin registers the four following tasks:

- [findChanges](./src/main/java/com/google/gradle/tasks/FindChangesTask.kt) -> Creates a new `.api` 
file for the current project, and compares it to the existing `released.api` to create a diff of 
changes made to the public api.
- [makeChange](./src/main/java/com/google/gradle/tasks/MakeChangeTask.kt) -> Creates a new `.json` 
file in the `.changes` directory with an optional message, including the version impact inferred 
from `findChanges`.
- [warnAboutApiChanges](./src/main/java/com/google/gradle/tasks/WarnAboutApiChangesTask.kt) -> If 
`findChanges` finds changes that will have version implications, this task will create a 
`api_changes.md` in the root project's build directory, with a message to display to the developer 
regarding what they changed, and the impact it will have on the public API. If no public API changes 
are found, this task will just skip itself.
- [deleteChangeFiles](./src/main/java/com/google/gradle/tasks/MakeReleaseNotesTask.kt) -> Deletes 
all the change files in the `.changes` directory, intended to be used after the release notes have 
been created- or otherwise during a cleanup stage of the release.
- [makeReleaseNotes](./src/main/java/com/google/gradle/tasks/MakeReleaseNotesTask.kt) -> Creates a 
`release_notes.md` in the root project's build directory, containing a collection of all the changes 
in the `.changes` directory; including the new project version at the top of file (inferred from the 
highest impact change).


## LicensePlugin

Handles the checking and adding of license headers to the top of all source files.

### Tasks

The LicensePlugin registers the two following tasks to facilitate this process:

- [validateLicense](./src/main/java/com/google/gradle/tasks/ValidateLicenseTask.kt) -> Validate that 
a license header in present in a set of files.
- [ApplyLicenseTask](./src/main/java/com/google/gradle/tasks/ApplyLicenseTask.kt) -> Applies a 
license header to a set of files.

## ReleasePlugin

Facilitates the procedures expected to be done during a release. 
While the `publishAllPublicationsToMavenRepository` task is used to actually *generate* the release 
artifact, this plugin registers tasks that should be ran *before* the release artifact is generated. 
Effectively "preparing" the project to be released.

### Tasks

The ReleasePlugin registers the three following tasks:

- [updateVersion](./src/main/java/com/google/gradle/tasks/VersionBumpTask.kt) -> Updates the project 
version declared in the `gradle.properties` file to reflect the version generated by the release 
notes.
- [createNewApiFile](./src/main/java/com/google/gradle/plugins/ReleasePlugin.kt) -> Creates a new 
`.api` file in the root `api` directory; aligning with the current state of the public api.
- [prepareRelease](./src/main/java/com/google/gradle/plugins/ReleasePlugin.kt) -> Does everything 
needed to prepare a release; creates the release notes and runs the above tasks.
