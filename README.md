# Ansible `known_hosts` Caching Role

Ansible role to cache (or re-cache/update) `ssh` host key(s) into `known_hosts` file.  All actions are taken locally (delegated to `localhost`), except that the host must be reachable (from localhost) to query for it's ssh key (via `ssh-keyscan`).

## Requirements

This role requires (on your control node)
 - a minimal version (> 1.2) of Ansible
 - `dnspython` python library

## Dependencies

None.

## Role Variables

* `known_hosts_ssh_key_auto_cache` (default: `true`)

  Should this role automatically cache the `ssh` host keys of new hosts?
  This is executed on `localhost` using the `ssh-keyscan` command to fetch the key,
  and the `known_hosts` module to cache it.

* `known_hosts_ssh_key_hash_hostname` (default: `false`)

  If true, hostname will be hashed in the `known_hosts` file.

* `known_hosts_ssh_key_cache_ipv4` (default: `true`)

  If true, cache the IP[V4] address(es) of the host as well.

* `known_hosts_ssh_key_cache_blindly` (default: `false`)

  If true, update cached keys on EVERY invocation.
  **SECURITY NOTE**: this runs a higher risk of MITM attacks,
  as the any cached host keys are replaced blindly every time.
  However, you might want to turn this on in certain situations,
  for example if you know the host keys have changed recently.
  Can be turned on either by changing this variable, or temporarily
  at invocation by providing the tag/option `-t cache_blindly`.

* `known_hosts_ssh_key_type` (default: `"ecdsa"`)

  What ssh key type should we fetch?  Typical options include:
    - `dsa`  (**NOTE: this is deprecated by most SSH installations as insecure**)
    - `rsa`
    - `ecdsa` (default)
    - `ed25519`

* `known_hosts_ssh_key_hosts_file` (default: `"~/.ssh/known_hosts"`)

  Location of the file into which the host key will be stored.  If unset (the default), the location is the default of the ansible `known_hosts` module, which is `~/.ssh/known_hosts`.

## Example Playbook

There is very little to this role, however, because it's primary task is to cache `ssh` host keys, and this generally should happen before you connect to the remote machines, any playbook that makes use of it must disable the gathering of facts before it runs; for example:

    - hosts: all
      gather_facts: false
      roles:
        - eengstrom.known_hosts

## Key Caching Caveats

Because this playbook is most likely to be run against a fresh system, typically it will be run with `gather_facts: false`; see above.  Because of this expected usage scenario, the caching of SSH host keys (see `known_hosts_ssh_key_auto_cache`) will use the value of `inventory_hostname` rather than `ansible_hostname`, since the latter requires facts to have been gathered, which will not have yet occurred.  This has the unfortunate *requirement* and assumption that your systems' hostnames and inventory names match.  If this is not the case, you may have issues.  If someone has a better way to do "the right thing", please submit a PR.

<!-- ## Testing

Testing of this role uses [`molecule`](https://molecule.readthedocs.io/en/latest/index.html) and it's docker driver.  To test, I use [`pipenv`](https://pipenv.readthedocs.io/en/latest/) for a local python environment, something like this:

    pipenv install docker molecule
    pipenv shell
    molecule test
    exit

CAVEAT: I do not have testing setup for FreeBSD, largely because using docker on Linux to spin up FreeBSD containers is not really easy.  I'll have to move to VirtualBox testing at some point. -->

## Author Information

- Eric Engstrom

## License

[BSD 3-Clause "New" or "Revised" License](https://spdx.org/licenses/BSD-3-Clause.html)
