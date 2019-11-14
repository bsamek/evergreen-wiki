If a test fails on a platform other than the one you develop on locally, you'll likely want to get access to a machine of that type in order to investigate the source of the failure. You can accomplish this using the spawn hosts feature of evergreen.

### Making a distro "spawnable"

Evergreen administrators can choose to make a distro available to users for spawning by checking the box on the distro configuration panel labeled *"Allow users to spawn these hosts for personal use"*

Only distros backed by a provider that supports dynamically spinning up new hosts (static hosts, of course, do not) allow this option.

### Spawning a Host

Visit `/spawn` to view the spawn hosts control panel. Click on "Spawn Host" and choose the distro you want to spawn, and hoose the key you'd like to use (or provide a new one).

Alternately, for a task that ran on a distro where spawning is enabled, you will see a "Spawn..." link on its task page. Clicking it will pre-populate the spawn host page with a request to spawn a host of that distro, along with the option to fetch binaries and artifacts associated with the task (this can also be performed manually; see [fetch](https://github.com/evergreen-ci/evergreen/wiki/Using-the-Command-Line-Tool#fetch) in the Evergreen command line tool documentation.

EC2 spawn hosts can be stopped/started and modified from the Spawn Host page, or via the command line, which is documented in [Basic Host Usage](https://github.com/evergreen-ci/evergreen/wiki/Using-the-Command-Line-Tool#basic-host-usage) in the Evergreen command line tool documentation.