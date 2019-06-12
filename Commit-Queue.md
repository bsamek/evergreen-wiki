Evergreen's commit queue merges changes after the code has passed a set of tests.

# Rationale 
Gating every merge on a green build means every commit on the tracked branch had a green build. This way: 
* No one bases their work on broken code.
* Every commit on the mainline branch is potentially releasable/deployable.

# Workflow
1. Changes are ready to be introduced to the repo.

[[images/repo_tree.png]]

2. The changes are added to the end of the commit queue.

[[images/queue.png]]

3. When the changes come to the head of the queue a test merge commit is created and tests are run against the merge commit.
 
[[images/merge_test.png]]

4. If the tests pass the merge commit is pushed to GitHub.

# Modes of Operation
The commit queue runs in two modes which are configured on a per project basis: PR or CLI.
## PR
For projects that use GitHub pull requests for code review, the commit queue integrates with the PR workflow. When the PR is approved and ready to be merged, changes are added to the queue through the PR and commit queue feedback is provided in status checks on the PR.
### Trigger
Add a PR to the commit queue by adding a comment on the PR: `evergreen merge` 

[[images/comment.png]]

### Feedback
Evergreen sends a status to the PR when the changes are enqueued, when the merge test has begun, and when the merge test is complete. Once the merge test has begun, the status links to the version.

[[images/github_status.png]]

### Set-Module
Add other PRs for repos defined as modules in the project's configuration file with the `--module MODULE:PR` option to the `evergreen merge` comment, where MODULE is the name of a module defined in the evergreen.yml project configuration file and PR is the PR number in the module's repo. When the test passes all the PRs will be merged. The option can be repeated for multiple modules. For instance: 

[[images/set-module_comment.png]]

The module can only be set when initially adding the main PR to the commit queue since the test is immediately finalized.

## CLI
For projects using other code review tools, changes can be uploaded to Evergreen using the `evergreen commit-queue` subcommands in the Evergreen CLI. 
### Trigger
`evergreen commit-queue merge --project <project_id> --message <message>`

Enqueue the branch checked out in Git on the project's commit queue to be merged into the project's tracked branch.

#### Options
* `--message [message], -m [message]`

Set the commit message for the merge commit pushed to GitHub. Default is the name of the checked out branch.

* `--pause`

Upload the changes, but don't begin to test the changes. Used in conjunction with `evergreen commit-queue set-module` when the changes include corresponding changes to a module.

* `--identifier [ID]`

Enqueue changes previously uploaded and paused. [ID] is the patch ID printed to the console when uploading the paused (unfinalized) patch.

### Feedback
The merge is performed by a "Commit Queue Merge" task added to the patch when the changes reach the head of the queue.

[[images/merge_task.png]]

### Set-Module
`evergreen commit-queue set-module --patch <id> --module <module>` 

Add changes to a project module.

#### Options
* `--patch [ID], --id [ID], -i [ID]` 

Add to patch ID created with `evergreen commit-queue merge --pause`

* `--module [module], -m [module]`

Module name as defined in the project configuration file.

#### Workflow
1. Upload the main changes to Evergreen. Retain the patch ID printed to the console.

`evergreen commit-queue merge --pause ...`

2. For each module to be merged, add the module to the patch

`evergreen commit-queue set-module --id <id> ...`

3. Enqueue the changes on the commit queue

`evergreen commit-queue merge --identifier <id> ...`

# Notifications
Adjust your notifications on commit queue milestones on the [Notifications page](https://evergreen.mongodb.com/notifications).

[[images/notifications.png]]

Milestones include:
* Changes reached the head of the queue and merge test has begun.
* Merge test passed/failed

# Configuration
Project admins configure the commit queue through the [Projects page](https://evergreen.mongodb.com/projects) in the Evergreen UI. On a per project basis they can
* Enable and disabled the commit queue
* Choose the [GitHub merge method](https://help.github.com/en/articles/about-merge-methods-on-github) (squash, merge, or rebase)
* Add/remove patch definitions for tests to be run against PRs (tags or variant and task regexes)
* Choose the commit queue mode (CLI or PR)

# Queue Monitoring
The Evergreen CLI exposes two subcommands under `evergreen commit-queue` to monitor the commit queue 

## List
`evergreen commit-queue list --project <project_id>`

List the items on one project's queue.

### Options
* `--project PROJECT, -p PROJECT` list the queue of PROJECT

## Delete
`evergreen commit-queue delete --project <project_id> --item <item>`

Delete an item from a queue. Must be the user who uploaded the item or a project admin.
If the item is already picked up by the commit queue it will not be aborted.

### Options
* `--project PROJECT, -p PROJECT`

delete from the queue of PROJECT

* `--item ITEM, -i ITEM` 

delete the ITEM specified. 
Item is the PR number in PR mode and patch ID in CLI mode

# FAQ
> Can I restart a failed task in a commit queue test?

Please don't. Instead, add the item to the queue again.

> The merge test for a PR failed.  How do I resubmit?

Fix the PR and type another triggering comment.

> The commit queue hasn't picked up my changes for a while.

Check your position in the queue with the CLI. Evergreen checks the head of each queue about once a minute.

> My project's branch is protected and only signed commits can be merged. Can I use the commit queue?

It depends: it works for PR mode, but not CLI mode.

> What will the commit message be with the squash merge strategy?

For PR mode, the commit message is the title of the PR with the PR number appended to the end. In CLI mode the message is specified with the `--message` option, or the branch name by default.