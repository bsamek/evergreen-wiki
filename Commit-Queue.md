Evergreen's commit queue merges changes after the merged code has passed a set of tests.

# Rationale 
Gating every merge on a green build means the tracked branch is guaranteed to always be green. This way: 
* No one bases their work on broken code.
* Every commit on the mainline branch is potentially releasable/deployable.

# Workflow
1. Changes are ready to be introduced to the repo.

[[images/repo_tree.png]]

2. The changes are added to the commit queue.


3. When the changes come to the head of the queue a test merge commit is created and tests are run against the merge commit.
 
[[images/merge_test.png]]

4. If the tests pass the merge commit is pushed to GitHub.

# Trigger
Add a PR to the commit queue by adding a comment on the PR: `evergreen merge` 
# Set-Module
Add other PRs for repos defined as modules in the project's configuration file with the `--module MODULE:PR` option to the `evergreen merge` comment. When the test passes all the PRs will be merged. The option can be repeated for multiple modules. For instance: 
> `evergreen merge --module module1:1234 --module module2:2345`

The module can only be set when initially adding the main PR to the commit queue since the test is immediately finalized.

# Notifications
Patch notifications for merge test patches are delivered according to users' notification settings. Merge status is also posted to the PR ([GitHub status checks](https://help.github.com/en/articles/about-status-checks)) at the following milestones:
* PR is queued
* merge test has begun
* merge test passed/failed
* merge failed

# Configuration
Project admins configure the commit queue through the [Projects page](https://evergreen.mongodb.com/projects) in the Evergreen UI. On a per project basis they can
* Enable and disabled the commit queue
* Choose the [GitHub merge method](https://help.github.com/en/articles/about-merge-methods-on-github) (squash, merge, or rebase)
* Add/remove patch definitions for tests to be run against PRs (tags or variant and task regexes)

# CLI
The Evergreen CLI exposes two subcommands under `evergreen commit-queue` to interact with the commit queue 
## List
List the items on one project's queue.
### Options
* `--project PROJECT, -p PROJECT` list the queue of PROJECT
## Delete
Delete an item from a queue. Must be a project admin.
If the PR is already picked up by the commit queue it will not be aborted.
### Options
* `--project PROJECT, -p PROJECT` delete from the queue of PROJECT
* `--item ITEM, -i ITEM` delete the ITEM specified

# FAQ
> The merge test failed. How do I resubmit?

Fix the PR and type another triggering comment.

> The commit queue hasn't picked up my PR for a while.

Check your PR's position in the queue with the CLI. Evergreen checks the head of the queue about once a minute.

> My project's branch is protected and only signed commits can be merged. Can I use the commit queue?

Yes. As long as all the commits in the PR are signed GitHub will do the right thing.

> What will the commit message be with the squash merge strategy?

The commit message is the title of the PR with the PR number appended to the end.