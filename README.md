opam-ansible
=========

Manages the installation, switch creation, and configuration of [opam](https://opam.ocaml.org/), the OCaml package manager.

Requirements
------------

Executing this role **does not require opam installed**. The role will install opam for you.

If you want to install opam through the system package manager, you need to know the opam package manager name for your system (default is `opam`). You can find it [here](https://opam.ocaml.org/doc/Install.html). If you want a direct installation from the official opam install script, you may need to change the `opam_external_dependencies` variable to list the dependencies needed to run the script, or run it yourself before running the role.

If you specify an opam package with external dependencies, they are installed through the system package manager.

Role Variables
--------------

The required variables are :

- `opam_install_method` : The method used to install opam. Can be `package_manager` (using, `apt`, `dnf` or your favorite distribution package manager) or `direct` (uses the install script from the opam website).
- `opam_install_location` : either `system_wide` for a system-wide installation or `user` for a user installation. Default is `system_wide`. For a user installation, it will install opam in the user's home directory `~/.opam`. For a system-wide installation, it will default to installing opam in `/opt/opam` (you can alter this with the `opam_root_location_system_wide` variable).

You probably also want to set the following variables :

- `opam_switches` : a list of switch to create and configure, each one should have a `name`, a `compiler` and a list of `packages` to install. The compiler should be a string with the version number (e.g. "4.14.2" or ). The packages should be a list of strings with the package name. The default is an empty list (no switch created). For example :
  ```yaml
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
- `default_switch` : the name of the switch to set as the default. Default is `""`. If you don't set it, the ansible script will leave it .
- `configure_vscode`, `configure_jupyter` and `configure_user_setup` are boolean variables that will configure the corresponding tools for the user. Default is `false` (no postinstall configuration, except eventually configuring the default switch).

Dependencies
------------

Depends only on the `ansible.builtin` collection.

Remarks
-------

For a system-wide installation role assumes you are using `bash` and all users should be able to access ocaml without modifying their configuration files. If you want some user to use another shell, you should add the appropriate command to your shell configuration file if necessary, either manually or through the `opam init` command. For example, to configure `zsh` for the current user, you can run the following command :

```shell
opam init --shell=zsh
```

Example Playbook
----------------

```yaml
- hosts: localhost
  roles:
    - role: opam-ansible
      vars:
        opam_install_method: package_manager
        opam_install_location: user
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
