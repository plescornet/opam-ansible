opam-ansible
=========

Manages the installation, switch creation, and configuration of [opam](https://opam.ocaml.org/), the OCaml package manager.

Requirements
------------

Executing this role **does not require opam installed**. The role will install opam for you.

If you want to install opam through the system package manager, you need to know the opam package manager name for your system (default is `opam`). You can find it [here](https://opam.ocaml.org/doc/Install.html).

Direct installation supports Linux only and downloads a verified, pinned OPAM release binary. Set `opam_direct_url` to the exact binary URL and `opam_direct_checksum` to its SHA-256 in Ansible format (`sha256:<64-hex-character checksum>`). Find official release binaries at [OPAM releases](https://github.com/ocaml/opam/releases). Set `opam_replace_package_managed: true` only when the role should remove an existing package-manager OPAM installation.

If you specify an opam package with external dependencies, they are installed through the system package manager.

Role Variables
--------------

The required variables are :

- `opam_install_method` : The method used to install opam. Can be `package_manager` (using, `apt`, `dnf` or your favorite distribution package manager) or `direct` (downloads a verified, pinned Linux release binary).
- `opam_install_location` : either `system_wide` for a system-wide installation or `user` for a user installation. Default is `system_wide`. For a user installation, it will install opam in the user's home directory `~/.opam`. For a system-wide installation, it will default to installing opam in `/opt/opam` (you can alter this with the `opam_root_location_system_wide` variable).
- `opam_direct_url` and `opam_direct_checksum`: required when `opam_install_method: direct`. They identify and verify a pinned Linux OPAM release binary.

You probably also want to set the following variables :

- `opam_switches` : a list of switch to create and configure, each one should have a `name`, a `compiler` and a list of `packages` to install. The compiler should be a version string (e.g. "4.14.2") or `latest` for the newest compiler available from OPAM's configured repositories. `latest` is resolved when creating a new switch; for an existing switch, the `latest_mismatch` policy decides what happens when the installed compiler is older than the newest available one: `skip` (default, keeps the switch untouched), `fail` (aborts the play) or `recreate` (removes and rebuilds the switch with the newest compiler). For an explicit compiler version mismatch, `compiler_mismatch` defaults to `fail`; set it to `skip` to retain the switch or `recreate` to remove and rebuild it. The global default for `latest_mismatch` is the `opam_latest_mismatch` variable (default `skip`). The packages should be a list of strings with the package name. The default is an empty list (no switch created). For example :
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
- `default_switch` : the name of the switch to set as the default. Default is `""`. If you don't set it, the role leaves opam's default switch unchanged.
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

Resolving `latest` uses GNU `sort -V`, so targets must be GNU/Linux (already the case for direct installations).

With `latest_mismatch: recreate` (or `compiler_mismatch: recreate`), the existing switch is removed and rebuilt **before** the requested packages are resolved and installed. If one of the packages is not compatible with the new compiler (for instance not yet ported to a fresh OCaml release), the play fails at dependency resolution (`opam` exits with code 20 and a "no solution" diagnostic) and the machine is left with an empty switch and no packages. Recovery requires fixing the configuration (pin an explicit compiler version, or remove/update the incompatible package) and running the play again. For reproducible classroom machines, an explicit pinned compiler is therefore safer than `latest` with `recreate`.

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
Troubleshooting
---------------

### `Recursive loop detected in template: maximum recursion depth exceeded`

This error occurs when a wrapper role re-declares a role variable with the same name, for example:

```yaml
- name: Install opam
  ansible.builtin.include_role:
    name: plescornet.opam-ansible
  vars:
    opam_install_method: "{{ opam_install_method }}"   # recursion!
```

The `vars` entry shadows the role parameter, so the template resolves to itself indefinitely. If your wrapper role receives the value from its caller, simply omit the variable from `vars` (role parameters are inherited by included roles), or rename your wrapper's own parameter and pass the value explicitly:

```yaml
- name: Install opam
  ansible.builtin.include_role:
    name: plescornet.opam-ansible
  vars:
    opam_install_method: "{{ my_wrapper_opam_install_method }}"
```

### `ansible_env` is undefined or deprecated

The role uses `ansible_facts['env']` and `ansible_facts['user_id']` instead of the deprecated `ansible_env` and `ansible_user_id` shorthands. Make sure the setup/gathering step has run so `ansible_facts` is populated.

License
-------

CC-BY-4.0

Author Information
------------------

Pierre LE SCORNET (pierre.le-scornet@ac-rennes.fr)
