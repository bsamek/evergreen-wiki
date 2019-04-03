### In general, use [subprocess.exec](https://github.com/evergreen-ci/evergreen/wiki/Project-Commands#subprocess-exec) instead of shell.exec, and avoid inline shell scripts ###

The reasons to prefer subprocess.exec include:
1. Evergreen uses expansions with the same syntax as shell expansions.
2. The shell isn't always bash. Debian/Ubuntu sh is dash.
3. Debugging inline shell scripts is hard.
4. When they're inline, it becomes hard for people to execute them themselves, which makes it difficult to reproduce what Evergreen has done locally.

You can pass environment variables to subprocess.exec if you'd like to pass expansions.

### Use [task tags](https://github.com/evergreen-ci/evergreen/wiki/Project-Files#task-tags) to reduce repetition in your Evergreen configuration file ###