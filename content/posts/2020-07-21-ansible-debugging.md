---
layout: post
title:  "Ansible debugging"
date:   2020-07-21 19:39:30 +0100
---

This post serves as a cheat sheet for debugging Ansible playbooks. It covers common troubleshooting techniques, useful modules, and configuration options to help you quickly identify and resolve issues in your automation workflows.

## Check Python and Ansible versions

Before debugging, ensure you are using compatible versions of Python and Ansible. Version mismatches can cause unexpected errors.

```bash
python --version
ansible --version
```

## Syntax check

Validate your playbook’s syntax before running it. This helps catch YAML formatting errors and invalid Ansible constructs early.

```bash
ansible-playbook --syntax-check <playbook.yml>
```

## Dry run

Use check mode to simulate playbook execution without making any changes to your systems. This is useful for verifying what changes would be made.

```bash
ansible-playbook --check <playbook.yml>
```

## Verbosity

Add verbosity flags (`-v`, `-vv`, `-vvv`, `-vvvv`) to get more detailed output. The highest level (`-vvvv`) shows all communication between Ansible and remote hosts, which is invaluable for troubleshooting.

```bash
ansible-playbook -vvvv <playbook.yml>
```

## Debug module

The `debug` module allows you to print variables and custom messages during playbook execution. This is essential for inspecting values and understanding playbook flow.

```yaml
---
- name: Debug example
  hosts: localhost
  tasks:
    - name: Get date
      command:
        cmd: /usr/bin/date
      register: date_result
    
    - name: Display variable
      debug:
        var: date_result.stdout
    
    - name: Print message
      debug:
        msg: "Date is: {{ date_result.stdout }}"
```

## Assert module

Use the `assert` module to enforce conditions within your playbook. If an assertion fails, the playbook stops and displays a custom error message.

```yaml
---
- name: Assert example
  hosts: localhost
  vars:
    - my_var: 21
  tasks:
    - name: Assert variable
      assert:
        that:
          - my_var == 21
        fail_msg: "'my_var' has wrong value"
        success_msg: "'my_var' has correct value"
```

## Fail module

The `fail` module lets you intentionally stop playbook execution with a custom message when certain conditions are met. This is useful for enforcing critical requirements.

```yaml
---
- name: Fail example
  hosts: localhost
  vars:
    - my_var: 1
  tasks:
    - name: Fail when my_var has wrong value
      fail:
        msg: Failed because my_var has wrong value
      when: my_var == 1
```

## Meta tasks

Meta tasks allow you to control playbook execution flow and manage facts. For example, you can clear facts and force Ansible to gather them again.

```yaml
---
- name: Meta example
  hosts: localhost
  tasks:
    - name: Display fact - IP address
      debug:
        var: ansible_default_ipv4.address

    - name: Clear facts
      meta: clear_facts

    - name: Display fact again
      debug:
        var: ansible_default_ipv4.address
```

## Task debugger

Ansible includes a built-in debugger that can be triggered on failed, unreachable, or skipped tasks. This allows you to inspect and modify variables, re-run tasks, or continue execution interactively.

### Enable task debugger

The easiest way to enable the debugger is through the configuration file or by using an environment variable.

In the configuration file (`ansible.cfg`):
```ini
[defaults]
enable_task_debugger = True
```

Using an environment variable:
```bash
ANSIBLE_ENABLE_TASK_DEBUGGER=True ansible-playbook <playbook.yml>
```

By default, the debugger starts on failed or unreachable tasks unless explicitly disabled in the playbook. You can control when the debugger is triggered using the `debugger` keyword with values: `always`, `never`, `on_failed`, `on_unreachable`, and `on_skipped`.

```yaml
---
- name: My playbook
  hosts: localhost
  debugger: never
  tasks:
    - name: This will fail
      command: something
      debugger: on_failed
```

### Common debugger commands

| Command                           | Description                                 |
|:----------------------------------|:--------------------------------------------|
| print task/task_vars/result/host  | Print values of tasks, variables, results   |
| task.args[key]                    | Set arguments for the current task          |
| task_vars[key]                    | Set variables for the current task          |
| update_task                       | Recreate the task with updated variables    |
| redo                              | Run the task again                          |
| continue                          | Continue with playbook execution            |
| quit                              | Exit the debugger                           |

## Linting

Linting helps you catch best practice violations and common mistakes before running your playbooks. Use `ansible-lint` and configure it with a `.ansible-lint` file.

**Check if installed and review configuration:**

| Config Section    | Description                                      |
|:------------------|:-------------------------------------------------|
| exclude_paths     | Directories or files to skip                     |
| parseable         | Output in PEP8 parseable format                  |
| quiet             | Suppress output                                  |
| rulesdir          | Custom rules directories                         |
| skip_list         | Skip rules matching these values                 |
| tags              | Only check rules matching these tags             |
| use_default_rules | Use default rules from ansible-lint library      |
| verbosity         | Set verbosity level                              |

**Run ansible-lint:**
```bash
ansible-lint <playbook.yml>
```

## Additional Tips

- check logs in `/var/log/ansible.log` if logging is enabled
- use `ansible-config dump` to review current configuration settings
- use `set_fact` to create or override variables for debugging
- use `pause` module to halt execution and inspect the environment
