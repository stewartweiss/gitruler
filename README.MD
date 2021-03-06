# GitRuler

## Introduction

GitRuler helps the teaching and assessment of git skills.

GitRuler allows you to easily create exercises where leaners follow a set of instructions to carry out on a repository, and will be scored, with feedback on their progress

![](docs/images/longer-screenshot.png)

## Installation and Running

1. Download the jar from Releases
2. Run `java -jar <path-to-jar>` specifying the repository and config file if necessary. 

GitRuler needs a configuration file (syntax described below) and a repository to test. By default it will look for a git repository in the current directory and will look for a config called `gitrules.json` in the root of the repository's working copy.

### Command Line Options

- `-r` `--repo`: specify the path to the repo to test
- `-c` `--config`: specify the path to the rule config.
- `-v` `--verbose`: provide more information about failed rules.

e.g. `java -jar <path-to-jar> -c ~/myotherrules.json -r ~/another/repository/`

## Creating Exercises

An exercise consists of:

1. [optionally] A starter repository that the student will do their work within.
2. Instructions for student on the actions to take on the repository.
3. A configuration file that describes the checks that should be made to the repository to see if the instructions have been carried out. Optionally this can also contain a list of files that should be created at the start of the exercise, for example so there are untracked files for the student to commit or ignore.

## Rules file

The rules files describes the setup of the exercise and the tests that must pass on the repository.

```json
{
  "setup-files": [
    {"path":"folder/a", "contents":"a"},
    {"path":"b", "contents":"This file should not be committed"}
  ],
  "rules":[
    { "rule": "head-exists", "stop-on-fail":true, "alternative-title":"Did you create the Repo?", "pre-text":"Step 1."},
    { "rule": "file-tracked-in-head", "path": "folder/a", "failure-message": "Did you stage and commit a?"},
    { "rule": "file-untracked-in-head", "path": "b"},
    { "rule": "last-commit-message-for-file-contains", "path":"folder/a", "contents":"added", "ignore-case":false}
  ]
}
```

### `Setup-files`

Setup files allow you to create files with given content. This is useful to allow you to create files necessary for the exercise that you don't wish to include in the repository that the student will clone.

- `path`: the path of the file to be created. Folders will be created if necessary.
- `contents`: the text to include in the file.

### Rules

Rules are the checks that will be made on the repository each time the application is run. The tool will list the current status for each test. Each different rule type as a set of mandatory and optional parameters. There are also common parameters that can be added to any rule which will alter its appearance or behaviour.

| Rule Name        | Behaviour           | Mandatory Parameters  | Optional Parameters |
| ------------- |-------------| -----:| ----: |
| `text` |  Prints a line in the output containing a header and a row of characters | - | `separator`, `heading`, `width`, `double-space` |
| `head-exists` | Checks there is a repository with one commit | - | - |
| `file-tracked-in-head` | Checks that a file is tracked/committed in the last commit | `path` | - |
| `file-untracked-in-head` | Checks that a file is not tracked/committed in the last commit | `path` | - |
| `file-has-hash-in-head` | Checks a file has a given hash in the last commit. Useful for checking a file has been moved. | `path`, `hash` | - |
| `file-contains-in-head` | Check that a file contains a phrase within its contents in the last commit | `path`, `contents` | `ignore-case` |
| `last-commit-message-for-file-contains` | Check that the last commit message for a file contains a phrase | `path`, `contents` | `ignore-case` |
| `any-commit-message-for-file-contains` | Check that there is any commit message for a file containing a phrase | `path`, `contents` | `ignore-case` |
| `any-commit-message-contains` | Check that there is a commit that contains a phrase | `contents` | `ignore-case` |
| `commit-with-message-updated-file` | Check that a commit whose message contains a phrase had an update to a file| `path`, `contents` | `ignore-case` |
| `commit-with-message-doesnt-update-file` | Check that a commit whose message contains a phrase does not update a file| `path`, `contents` | `ignore-case` |
| `ignored` | Check that a file at a given path would be ignored by git | `path` | - |
| `file-tracked-in-branch` | Checks if a file exists in the given branch | `path`, `branch` | - |
| `file-contains-in-branch` | Checks if a file exists with given contents in the given branch | `path`, `branch`, `contents` | `ignore-case` |
| `branch-exists` | Checks if a given branch exists | `branch` | - |
| `commit-with-message-was-made-on-branch` | Check if a certain commit was made on the given branch| `branch`, `contents` | `ignore-case` |
| `commit-with-message-was-merged-into-branch` | Check if a certain commit merged into a branch| `branch`, `contents` | `ignore-case` |
| `tag-exists` | Check if there is a tag with the given name anywhere in the repository | `tag` | - |
| `commit-with-message-has-tag` | Check that a given commit is tagged| `tag`, `contents` | `ignore-case` |
| `tagged-commit-added-text-to-file` | Check if a commit with the given tag added certain text to a file | `tag`, `contents`, `path` | `ignore-case` (of file contents) |
| `at-least-a-number-of-commits` | Ensure a certain number of commits are in the repository | `number` (integer) |  |

#### Common Rule Options

The following options can be added to any rule to affect its behaviour or the output.

- `pre-text`: Text that will appear before the rule's name in the output. e.g. "[Step 1]"
- `post-text`: Text that will appear after the rule's name in the output. e.g. "[See git help log"]"
- `failure-message`: A specific message if the rule fails. e.g. "Did you remember to commit the file?"
- `alternative-title`: An alternative description of the rule to appear in the output. e.g. "You created the repository"
- `stop-on-fail`: Causes all following rules to be skipped if this one fails. Good if other rules make no sense if a prior one fails.
- `score-if-correct`: Allocate a score for this rule which will be shown in the output. The final score will also be shown if there is one.
