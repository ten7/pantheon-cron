# Pantheon Cron

An Ansible role to automate running tasks on Pantheon sites via terminus

## Requirements

* The `terminus` command must be installed and executable by the user running the role.
* PHP must be installed for the `terminus` command to function. 

Please see [Pantheon's Terminus documentation](https://pantheon.io/docs/terminus/install) on how to install the command.

## Dependencies

The following collections must be installed:

* community.general

## Role Variables

This role requires one dictionary as configuration, `pantheon_cron`:

```yaml
    pantheon_cron:
      terminusPath: "/usr/local/bin/terminus"
      debug: true
      stopOnFailure: false
      sites: {}
      tasks: []
```

Where:
* `terminusPath` is the full path to the `terminus` executable. Optional, defaults to `terminus`.
* `debug` is `true` to enable debugging output. Optional, defaults to `false`.
* `stopOnFailure` is `true` to stop the entire role if any one task fails. Optional, defaults to `false`.
* `sites` is a dictionary of Pantheon sites. Required.
* `tasks` is a list of tasks to perform. Required.

### Specifying sites

In this role, `sites` specifies site configurations on which to execute tasks. Each must have a unique key which is later used in the `pantheon_cron.tasks` list.

```yaml
pantheon_cron:
  sites:
    example.com:
      site_id: '12345678-abcd-ef01-2345-67890abcdef0'
      machineTokenFile: "/path/to/pantheon-machine-token.txt"
      keyFile: "/config/example-com/id_pantheon"
      pubKeyFile: "/config/example-com/id_pantheon.pub"
      retryCount: 3
      retryDelay: 30
```

Where, in each entry:
* `site_id` is the Pantheon site ID. Required.
* `machineTokenFile` is the path to a file containing the Pantheon machine token. Optional if `machineToken` is specified.
* `machineToken` contains the Pantheon Machine token. Ignored if `machineTokenFile` is specified.
* `keyFile` is the path to an SSH private key used to access the Pantheon site.
* `pubKeyFile` is the path to an SSH public key used to access the Pantheon site.
* `retryCount` is the number of time to retry `terminus` commands if they fail. Optional, defaults to `3`.
* `retryDelay` is the time in seconds to wait before retrying a failed `terminus` command. Optional, defaults to `30`.

### Specifying tasks

The `pantheon_cron.tasks` list specifies the tasks to ultimately perform, referencing the `pantheon_cron.sites` section for connectivity details.

```yaml
pantheon_cron:
  tasks:
    - name: "example.com cron and cr"
      site: "example.com"
      env: "live"
      disabled: false
      commands: []
```

Where:
* `name` is the display name of the backup. Optional, but makes the logs easier.
* `site` is the name of the key under `pantheon_crons.sites` on which to execute tasks. Required.
* `env` is the Pantheon environment of the site on which to execute the task. Required.
* `disabled` is `true` to disable (skip) the task. Optional, defaults to `false`.
* `commands` is a list of `terminus` commands and their arguments to perform in sequence. Required.

### Commands

The `commands` list under each item in `pantheon_cron.tasks` list specifies the `terminus` commands to execute.

```yaml
pantheon_cron:
  tasks:
    - name: "example.com cron and cr"
      source: "example.com"
      env: "live"
      commands:
        - command: "remote:drush"
          arg: "cron"
          disabled: false
        - command: "remote:drush"
          arg: "cr"
          disabled: false
```

Where:
* `command` is the `terminus` command to perform, typically `remote:drush` or `remote:wp`. Required.
* `arg` is a string to pass to the Terminus command. Required
* `disabled` is `true` to skip executing the commad. Optional, defaults to `false`.

## Example Playbook

```yaml
    - hosts: servers
      vars:
        pantheon_cron:
          sites:
            example.com:
              site_id: '12345678-abcd-ef01-2345-67890abcdef0'
              machineTokenFile: "/path/to/pantheon-machine-token.txt"
              keyFile: "/config/example-com/id_pantheon"
              pubKeyFile: "/config/example-com/id_pantheon.pub"
              retryCount: 3
              retryDelay: 30
        tasks:
          - name: "example.com cron and cr"
            source: "example.com"
            env: "live"
            commands:
              - command: "remote:drush"
                arg: "cron"
                disabled: false
              - command: "remote:drush"
                arg: "cr"
                disabled: false
      roles:
         - { role: ten7.pantheon_deploy }
```

## License

GPL v3

## Author Information

This role was created by [TEN7](https://ten7.com/).
