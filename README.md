opam
=========

Manages the installation, switch creation, and configuration of [opam](https://opam.ocaml.org/), the OCaml package manager.

Requirements
------------

Executing this role **does not require opam installed**. The role will install opam for you.

This role assumes you are using `bash`.

If you want to install opam through the system package manager, you need to know the opam package manager name for your system. You can find it [here](https://opam.ocaml.org/doc/Install.html). If you want a direct installation from the opam install script, you need curl available.

If you specify an opam package with external dependencies, you need to have them installed on your system.

Role Variables
--------------

Most important variables are:
- `opam_install_method` : The method used to install opam. Can be `package_manager` or `direct`. Default is `direct`.
- `opam_install_location` : either `system_wide` for a system-wide installation or `user` for a user installation. Default is `system_wide`. For a user installation, it will install opam in the user's home directory `~/.opam`. For a system-wide installation, it will default to installing opam in `/opt/opam` (you can alter this with the `opam_root_system_wide` variable).
- `opam_switches` : a list of switches to create. For each switch, you must specify its name, the compiler version and the packages to install.

Dependencies
------------

Depends only on the `ansible.builtin` collection.

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

<!-- TODO -->

License
-------

?

Author Information
------------------

Pierre LE SCORNET (pierre.le-scornet@ac-rennes.fr)