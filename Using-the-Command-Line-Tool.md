.. contents:: `Table of contents`
   :depth: 4
   :local:

How to set up and use the command-line tool
==

Downloading the Command Line Tool
--

Go to your [evergreen user settings page](https://evergreen.mongodb.com/settings) (accessible using the user drop-down in the upper right, or at `/settings`) to find links to download the binaries, if the server admin has made them available.
Copy and paste the text in the configuration panel on the settings page into a file in your *home directory* called `.evergreen.yml`, which will contain the authentication information needed for the client to access the server.

Basic Patch Usage
--

To submit a patch, run this from your local copy of the mongodb/mongo repo:
```bash
evergreen patch -p <project-id>
```
    
Variants and tasks for a patch can be specified with the `-v` and `-t`:
```bash
evergreen patch -v enterprise-suse11-64 -t compile
```

Multiple tasks and variants are specified by passing the flags multiple times:
```bash
evergreen patch -v enterprise-suse11-64 -v solaris-64-bit -t compile -t unittest -t jsCore
```

And _every_ task or variant can be specified by using the "all" keyword:
```bash
evergreen patch -v all -t all
```

Working Tree Changes
---
By default patches will include only committed changes, not changes in Git's working tree or index. To include changes from the working tree use the `--uncommitted, -u` flag or set a default by inserting `patch_uncommitted_changes: true` in the `~/.evergreen.yml` file.

Defaults
---
The first time you run a patch, you'll be asked if you want to set the given parameters as your default project and variants.
After setting defaults, you can omit the flags and the default values will be used, so that just running `evergreen patch` will work.

Defaults may be changed at any time by editing your `~/.evergreen.yml` file.


Advanced Patch Tips
--

##### Multiple Defaults
While the `evergreen` program has no built-in method of saving multiple configurations of defaults for a project, you can easily mimic this functionality by using multiple local evergreen configurations.
The command line tool allows you to pass in a specific config file with `--config`:
```bash
evergreen --config ~/.many_compiles.yml patch
```
You can use this feature along with shell aliasing to easily manage multiple default sets.

For example, an enterprising server engineer might create a config file called `tests.yml` with the content
```yaml
api_server_host: #api
ui_server_host: #ui
api_key: #apikey
user: #user
projects:
- name: mongodb-mongo-master
  variants:
  - windows-64-2k8-debug
  - enterprise-rhel-62-64-bit
  tasks:
  - all
```
so that running `evergreen --config tests.yml patch` defaults to running all tasks for those two variants.

You might also want to create a config called `compile.yml` with
```yaml
api_server_host: #api
ui_server_host: #ui
api_key: #apikey
user: #user
projects:
- name: mongodb-mongo-master
  variants:
  - windows-64-2k8-debug
  - enterprise-rhel-62-64-bit
  - solaris-64-bit
  - enterprise-osx-108 #and so on...
  tasks:
  - compile
  - unittests
```
for running basic compile/unit tasks for a variety of platforms with `evergreen --config compile.yml patch`.
This setup also makes it easy to do scripting for nice, automatic patch generation.

##### Git Diff
Extra args to the `git diff` command used to generate your patch may be specified by appending them after `--`.  For example, to generate a patch relative to the previous commit:

      evergreen patch -- HEAD~1

Or to patch relative to a specific tag:

      evergreen patch -- r3.0.2

Though keep in mind that the merge base must still exist in the canonical GitHub repository so that Evergreen can apply the patch.


The `--` feature can also be used to pass flags to `git diff`.


Operating on existing patches
--

To list patches you've created:

      evergreen list-patches


To cancel a patch:
 
```
evergreen cancel-patch -i <patch_id>
```
    
To finalize a patch:
 
```
evergreen finalize-patch -i <patch_id>
```
Finalizing a patch actually creates and schedules and tasks. Before this the patch only exists as a patch "intent". You can finalize a patch either by passing --finalize or -f or by clicking the "Schedule Patch" button in the UI of an un-finalized patch.



To add changes to a module on top of an existing  patch:

```
cd ~/projects/module-project-directory
evergreen set-module -i <patch_id> -m <module-name>
```
Note: `set-module` must be run before finalizing the patch.

##### Validating changes to config files

When editing yaml project files, you can verify that the file will work correctly after committing by checking it with the "validate" command.

```
evergreen validate <path-to-yaml-project-file>
```

The validation step will check for
   * valid yaml syntax
   * correct names for all commands used in the file
   * logical errors, like duplicated variant or task names
   * invalid sets of parameters to commands
   * warning conditions such as referencing a distro pool that does not exist

Note: validation is server-side and requires a valid evergreen configuration file (by default located at ~/.evergreen.yml). If the configuration file exists but is not valid (malformed, references invalid hosts, invalid api key, etc.) the `evergreen validate` command [will exit with code 0, indicating success, even when the project file is invalid](https://jira.mongodb.org/browse/EVG-6417). The validation is likely not performed at all in this scenario. To check whether a project file is valid, verify that the process exited with code 0 and produced the output "<project file path> is valid".

Additionally the `evaluate` command can be used to locally expand task tags and return a fully evaluated version of a project file.

```
evergreen evaluate <path-to-yaml-project-file>
```

Flags `--tasks` and `--variants` can be added to only show expanded tasks and variants, respectively.

Basic Host Usage
--
Evergreen Spawn Hosts can now be managed from the command line, and this can be explored via the command line `--help` arguments. 

### Attaching an EBS Volume

To create a new EBS volume: 
```
evergreen volume create --size <size> --type <default=gp2> --zone <default=us-east-1a> 
```
While the Availability Zone does have a default, this must be in the _same zone as the host_. If you don't know your host's availability zone, this can be found easily at `evergreen host list --mine`. 

To attach the volume to your host (assuming the same availability zone), use:
```
evergreen host attach --host <host_id> --volume <volume_id>
```
If you forget your volume ID, you can find this with `evergreen volume list`. If the volume is already attached, you will see a host ID given with this volume, and a volume can only have one host. 

A volume can only be deleted if detached, so removing a volume would for example be: 
``` 
evergreen host detach --host <host_id> --volume <volume_id>
evergreen volume delete --id <volume_id>
```

### Modify Hosts

Tags can be modified for hosts using the following syntax:
```
evergreen host modify --tag KEY=VALUE
evergreen --delete-tag KEY
```
Note these tags cannot overwrite Evergreen tags. 

Hosts can be set to never expire using the `--no-expire` tag (although each user has a limit for these kinds of hosts). Hosts can be set to expire again using the `--expire` tag, which will set the host to expire in 24 hours (this can be extended using `--extend <hours>`. 


### Stop/Start Host to Change Instance Type

Instance type can only be changed if the host is stopped. Hosts can be stopped and started using `evergreen host start/stop --host <id> --wait <set-to-block>`. To change instance type, `host modify --type` (approved types can be configured from the admin settings). 

Other Commands
--

#### Fetch

The command `evergreen fetch` can automate downloading of the binaries associated with a particular task, or cloning the repo for the task and setting up patches/modules appropriately.

Example that downloads the artifacts for the given task ID and cloning its source:
```
evergreen fetch -t <task-id> --source --artifacts
```

Specify the optional `--dir` argument to choose the destination path where the data is fetched to; if omitted, it defaults to the current working directory.
 
#### List

The command `evergreen list` can help you determine what projects, variants, and tasks are available for patching against.
The commands
```
evergreen list --projects
evergreen list --tasks -p <project_id>
evergreen list --variants -p <project_id>
```
will all print lists to stdout.

The list command can take an optional `-f/--file` argument for specifying a local project configuration file to use instead of querying the Evergreen server for `-p/--project`.


#### Last Green

The command `evergreen last-green` can help you find an entirely successful commit to patch against.
To use it, specify the project you wish to query along with the set of variants to verify as passing. 
```
evergreen last-green -p <project_id> -v <variant1> -v <variant2> -v <variant...>
```

A run might look something like
```
evergreen last-green -p mci -v ubuntu

   Revision : 97ac269b1e5cf0961fce5bcf985f01c263911efb
    Message : EVG-795 no longer treat conflicting targets as system failures
       Link : https://evergreen.mongodb.com/version/mci_97ac269b1e5cf0961fce5bcf985f01c263911efb

```

#### Commit Queue
The command `evergreen commit-queue` contains subcommands for interacting with the commit queue. See [Commit Queue CLI](https://github.com/evergreen-ci/evergreen/wiki/Commit-Queue#cli).

### Server Side (for Evergreen admins)

To enable auto-updating of client binaries, add a section like this to the settings file for your server:


```yaml
api:
    clients:
        latest_revision: "c0110ba937047f99c9a68470f6ec51fc6d98c5cc"
        client_binaries:
           - os: "darwin"
             arch: "amd64"
             url: "https://.../evergreen"
           - os: "linux"
             arch: "amd64"
             url: "https://.../evergreen"
           - os: "windows"
             arch: "amd64"
             url: "https://.../evergreen.exe"
```

The "url" keys in each list item should contain the appropriate URL to the binary for each architecture. The "latest_revision" key should contain the githash that was used to build the binary. It should match the output of `evergreen version` for *all* the binaries at the URLs listed in order for auto-updates to be successful.