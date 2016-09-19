# Smash Ansible Role

Smash is an Ansible role which "mashes" together several small
snippets of code to create a "SMArter sSH" experience:

* `sshd` will use
  [PAM](https://en.wikipedia.org/wiki/Pluggable_authentication_module)
  to authenticate users via
  [LDAP](https://en.wikipedia.org/wiki/Lightweight_Directory_Access_Protocol).
* [`nslcd`](https://linux.die.net/man/8/nslcd) is used to query LDAP
* [`nscd`](https://linux.die.net/man/8/nscd) is used for caching
* SSH public keys in LDAP are used to authenticate logins
* `sudo` access is defined in LDAP
* Home directories are automatically created at login (via
    [`pam_mkhomedir`](https://linux.die.net/man/8/pam_mkhomedir)).
* Home directories are served from an
  [NFS](https://en.wikipedia.org/wiki/Network_File_System) share

# Requirements

This role does not set up the LDAP server.  It must already be running
and properly configured:

* It must have `posixAccount`, `ldapPublicKey`, and `posixGroup`
  entries already defined for the users who will login.
* It must have a `ou=SUDOers` entry containing `sudoRole` entries.

The `ldapPublicKey` and `sudoRole` LDAP object classes are not part of
the default schema of most LDAP installations.  You will need to
import them or create them yourself.  The necessary files are bundled
with this repository (see the `files` directory).

This role also does not set up the NFS server.  It must already be
configured with an NFS share appropriate for user home directories.

# Variables

There are a lot of variables to configure as this role is just glue.
Most of these make sense if you understand the underlying
technologies...but otherwise they can be opaque.

* Overall variables
  * `smash_ldap_host`: LDAP server host. (Default: `localhost`)
  * `smash_bind_dn`: The `dn` to bind as when looking up users. (Default: `cn=admin,dc=example,dc=com`)
  * `smash_bind_pw`: Password for above DN.
* `nslcd` variables:
  * `nslcd_bases`: List of `dn`s to search for POSIX assets. (Default: [`ou=people,dc=example,dc=com`, `ou=groups,dc=example,dc=com`])
  * `nslcd_scope`: The LDAP query scope to use when searching for POSIX assets, one of: `sub`, `one`, `base`.  (Default: `sub`)
  * `nslcd_root_pw_mod_dn` The `dn` to bind to when performing "superuser" tasks such as changing a user's password.
* `sshd` variables:
  * `sshd_ldap_base`: The `dn` to search for POSIX users. (Default: `ou=people,dc=example,dc=com`)
  * `sshd_ldap_object_class`: The LDAP object class POSIX users must have. (Default: ` ldapPublicKey`)
  * `sshd_ldap_user_attr`: The LDAP attribute defining a POSIX user's account name. (Default: `uid`)
  * `sshd_ldap_key_attr`: The LDAP attribute defining a POSIX user's SSH authorized keys.  (Default: `sshPublicKey`)
* NFS variables:
  * `home_nfs_host`: Host providing the NFS share (Default: `localhost`)
  * `home_nfs_share`: The name of the NFS share (Default: `/home`)
  * `home_nfs_dir`: The local path to mount the NFS share. (Default: `/home`)
  * `home_nfs_version`: The NFS version. (Default: `nfs4`)
  * `home_nfs_opts`: Options to mount the NFS share with. (Default: `sec=sys,noatime,nodiratime`)
* `pam_mkhomedir` variables:
  * `pam_mkhomedir_umask`: Umask for files in newly created home directory.  (Default: `0002`)
  * `pam_mkhomedir_skeleton`: Directory containing skeleton for new home directories.  (Default: `/etc/skel`)
* `sudo` variables:
  * `sudoers_ldap_base`: The `dn` to search for `sudoRole` entries.  (Default: `ou=SUDOers,dc=example,dc=com`)

# Usage

If all requirements are already met and you've set everything up
according to the defaults, you should be able to get away with
something like:

```yaml
- hosts: all
  roles:
    - role: smash
	  smash_ldap_uri: "ldap://MY_LDAP_HOST/"
      nslcd_bases:
        - "ou=people,dc=example,dc=com"
        - "ou=groups,dc=example,dc=com"
      sshd_ldap_base: "ou=people,dc=example,dc=com"
      home_nfs_host:  MY_NFS_HOST
      home_nfs_share: "/exports/home"
      sudoers_ldap_base: "ou=SUDOers,dc=example,dc=com"
```

# Note

Still some work/thinking to do about the key cache maintained by
`nscd`. One strategy (as part of removing a user account): run these
commands all sshd hosts to clear the cache:

```bash
$ nscd -i passwd
$ nscd -i group
```

# Author Information

* Brandon Hudgeons, with lots of assistance from myriad public sources.
* Dhruv Bansal
