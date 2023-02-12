# Ansible Role: mikroways.restic (forked)

See the original [README.md](../README.md) for usage & other details.

This fork will implement the following features:

1. Run restic as non-root, following the [docs recommendations](https://restic.readthedocs.io/en/stable/080_examples.html#backing-up-your-system-without-running-restic-as-root). I don't plan to create the user within this role.
2. Add `ExecStartPre=` & `ExecStartPost=` to the systemd_service.unit as a list of commands that a user can pass as a list. If the user calls a bash file, then this should already be available on the system (with the correct permissions).
3. Add the ability to [set the GOMAXPROCS in the backup script](https://github.com/arillso/ansible.restic/issues/109).
