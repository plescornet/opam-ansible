opam
=========

Manages the installation, switch creation, and configuration of [opam](https://opam.ocaml.org/), the OCaml package manager.

Requirements
------------

Executing this role **does not require opam installed**. The role will install opam for you.

This role assumes you are using `bash`. If you want to use another shell, you should add the appropriate command to your shell configuration file if necessary. For example, you can add in your `.zshrc` file :
```bash
eval $(opam env --root=/path/to/opam --set-root)
# by default on a system-wide installation, the root is /opt/opam :
eval $(opam env --root=/opt/opam --set-root)
```

If you want to install opam through the system package manager, you need to know the opam package manager name for your system (default is `opam`). You can find it [here](https://opam.ocaml.org/doc/Install.html). If you want a direct installation from the official opam install script, you may need to change the `opam_external_dependencies` variable to list the dependencies needed to run the script, or run it yourself before running the role.

If you specify an opam package with external dependencies, they are installed through the system package manager during the `opam install` command (using `--confirm-level=unsafe-yes`).

Role Variables
--------------

Most important variables are:
- `opam_install_method` : The method used to install opam. Can be `package_manager` (using, `apt`, `dnf` or your favorite distribution package manager) or `direct` (uses the install script from the opam website). Default is `direct`.
- `opam_install_location` : either `system_wide` for a system-wide installation or `user` for a user installation. Default is `system_wide`. For a user installation, it will install opam in the user's home directory `~/.opam`. For a system-wide installation, it will default to installing opam in `/opt/opam` (you can alter this with the `opam_root_location_system_wide` variable).
- `opam_switches` : a list of switches to create. For each switch, you must specify its name, the compiler version and the packages to install.

Dependencies
------------

Depends only on the `ansible.builtin` collection.

Example Playbook
----------------

```yaml
- hosts: localhost
  roles:
    - role: opam
      vars:
        opam_install_method: package_manager
        opam_install_location: user
        opam_root_location_user: "/home/{{ ansible_user_id }}/.opam"
        opam_switches:
          - name: "default"
            compiler: "4.14.2"
            packages:
              - dune
              - tuareg
              - ocaml-lsp-server
          - name: "bleeding-edge"
            compiler: "5.2.0"
            packages:
              - dune
              - ocaml-lsp-server
              - qcheck
```
License
-------

CC-BY-4.0

Author Information
------------------

Pierre LE SCORNET (pierre.le-scornet@ac-rennes.fr)