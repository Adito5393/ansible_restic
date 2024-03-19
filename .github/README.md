# Ansible Role: mikroways.restic (forked)

See the original [README.md](../README.md) for usage & other details.

This fork will implement the following features:

- [x] Add docs on How to develop
- [x] Run restic as non-root, following the [docs recommendations](https://restic.readthedocs.io/en/stable/080_examples.html#backing-up-your-system-without-running-restic-as-root).
   I don't plan to create the user within this role.
- [x] Add `ExecStartPre=` & `ExecStartPost=` to the systemd_service.unit as a list of commands that a user can pass as a list. If the user calls a bash file, then this should already be available on the system (with the correct permissions). Use case: stop a service before backing up & start it after it finishes it.
- [x] Add the ability to [set the GOMAXPROCS in the backup script](https://github.com/arillso/ansible.restic/issues/109).
- [x] Add the [bash-completion file](https://restic.readthedocs.io/en/latest/020_installation.html#autocompletion) if package is installed.

Check out the files changed summary via the [complete comparison](https://github.com/Mikroways/ansible_restic/compare/main...Adito5393:ansible_restic:dev) between the original fork and this branch.

## How to use

Add the version you want (e.g. `1.0.0`) to use inside your `requirements.yml` file (see the [Ansible docs](https://docs.ansible.com/ansible/latest/galaxy/user_guide.html#installing-roles-and-collections-from-the-same-requirements-yml-file) for more info about installing roles):

```yml
collections:
...
roles:
  - name: adrlzr.ansible_restic
    src: https://github.com/Adito5393/ansible_restic
    version: 1.0.0
    # Or use the latest version from the branch:
    # version: dev
```

And run: `ansible-galaxy install -r ./requirements.yml`

And use it inside your playbook:

```yml
- name: Setup Restic
  hosts: all
  become: true

  pre_tasks:
    - name: Pre-run | update package cache (debian, etc)
      tags: always
      ansible.builtin.apt:
        update_cache: true
      changed_when: false
      when: ansible_distribution in ["Debian", "Ubuntu"]

  tasks:
    - name: Provision Restic
      tags: restic
      ansible.builtin.import_role:
        name: adrlzr.ansible_restic
```

### Examples ExecStartPre and ExecStartPost

Given the following vars defined:
```yml
restic_backups:
  - name: opt_stuff
    src: /opt
    repo: local
    init: true
    restic_systemd_service_pre:
      - /usr/bin/echo "This is the command before systemd starts."
      - /usr/bin/echo "This is the 2nd command before systemd starts."
    restic_systemd_service_post:
      - /usr/bin/echo "This is the command AFTER systemd service finishes."
```

It will generate this part of systemd service:

```bash
cat /etc/systemd/system/restic-opt_stuff.service

[Unit]
...
[Service]
...
ExecStartPre=/usr/bin/echo "This is the command before systemd starts."
ExecStartPre=/usr/bin/echo "This is the 2nd command before systemd starts."
ExecStartPost=/usr/bin/echo "This is the command AFTER systemd service finishes."
```

Check the manual for more info about [ExecStartPre= and ExecStartPost=](https://www.freedesktop.org/software/systemd/man/latest/systemd.service.html#ExecStartPre=).


## How to develop

Here are the steps to setup a Debian/Ubuntu local environment:

1. Upgrade pip to latest & install [pipx](https://pypi.org/project/pipx/) (requires pip 19.0 or later) or [pipx-in-pipx](https://github.com/mattsb42-meta/pipx-in-pipx) (I recommend pipx-in-pipx, if you want to reinstall all other packages `pipx reinstall-all --skip pipx`):

    ```bash
    sudo apt install python3-venv python3-pip
    # pipx-in-pipx
    python3 -m pip install pipx-in-pipx --user

    # pipx
    python3 -m pip install --user pipx
    python3 -m pipx ensurepath
    # Upgrade pipx:
    # python3 -m pip install --user -U pipx
    ```

1. Install tox via pipx:

    ```bash
    pipx install tox
    ```

### Resources

- Why use pipx-in-pipx? ([comment from Hackers News](https://news.ycombinator.com/item?id=28245482))

    > Thanks to the way Python works when installed via homebrew on Mac this is actually very necessary if you want to install system-wide python utilities that do not depend on the system python. Without it you can have these utilities broken by a system update or by a homebrew update that bumps the Python version.
