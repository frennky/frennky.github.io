---
layout: post
title:  "Ansible Debugging tips"
date:   2020-07-21 19:39:30 +0100
---

# Ansible debugging tips

First check python and ansible versions.

## Syntax

```
ansible-playbook --syntax-check ...
```

## Dry run

```
ansible-playbook --check ...
```

## Verbosity

```
ansible-playbook -vvvv
```

## Debug module

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

## Task Debugger

Enable in configuration file - ansible.cfg:

```ini
[defaults]
enable_task_debugger = True
```

Enable using environment variable:
```
ANSIBLE_ENABLE_TASK_DEBUGGER=True ansible-playbook ...
```

Any failed or unreachable task will start the debugger, unless it is explicitly disabled in the playbook. Available values are `always`, `never`, `on_failed`, `on_unreachable`, and `on_skipped`.

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

| Command | Description |
|:----|:----------------|
|print task/task_vars/result/host|Print values|
|task.args[key]|Set arguments|
|task_vars[key]|Set vars|
|update_task|Recreates task with updated task_vars|
|redo|Run task again|
|continue|Contine with palybook|
|quit|Exit debugger|

## Lint

Check if installed and check configuration file - .ansible-lint.

| Config section | Description |
|:----|:-----------------------|
|exclude_paths|Dirs or files to skip|
|parseable|Parseable output in pep8 format|
|quiet|Quiet output|
|rulesdir|Rules dirs that override default rules|
|skip_list|Skip rules that match these values|
|tags|Check rules that match these values|
|use_default_rules|Use default rules under 'ansible-lint/lib/ansiblelint/rules'|
|verbosity|Verbosity level|

```
ansible-lint ...
```