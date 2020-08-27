Anarcho-Tech NYC: Disk Quotas
=========

Configure Linux ext3 or ext4 filesystem quotas per user or per user group, and optionally auto-enable the quota on the root (`/`) filesystem.

Requirements
------------

Users or groups on which you set a quota must already exist by the time this role runs.

The host on which this role runs must have a kernel with filesystem quota support. To confirm this:

```sh
lsmod | grep quota
```

If you don't see any output, quota support is either not supported by or not loaded in your kernel. You may already have the required kernel modules on your system, in which case you can simply attempt to load them into the running kernel:

```sh
sudo modprobe quota_v2
```

Otherwise, you'll need to get or build your own filesystem quota-supporting kernel. In the simplest cases:

```sh
sudo apt install linux-generic # Generic Linux kernel images support quotas.
sudo shutdown -r now           # Reboot is required for new kernel.
```

Then you can use this role to set and modify quota limits using the [role variables](#role-variables) described below.

Once you configure quotas, remember to actually activate them:

```sh
sudo quotaon /path/to/quota-enabled/filesystem/mountpoint
```

Role Variables
--------------

* `disk_quotas_users`: List of disk utilization limits imposed on a per-filesystem basis for a given Operating System user account. Each list item is a dictionary with the following structure:
    * `name`: The name of the user account to set a quota for.
    * `block_soft`: The soft limit for the amount of disk space that the given user can take up. Setting this to `0` means "no limit." The suffixes `K`, `M`, `G`, and `T` can be used to express kibibytes, mebibytes, gibibytes, and tebibytes, respectively. See the manual page for `setquota(8)` for more details.
    * `block_hard`: The hard limit for disk space used. The same semantics apply as for `block_soft`.
    * `inode_soft`: The soft limit for the number of files and directories that the given user can create. The same semantics apply as for `block_soft`.
    * `inode_hard`: The hard limit for number of files and directories created. The same semantics as above apply.
    * `filesystem`: The mount point of the filesystem to apply the quota on.
* `disk_quotas_groups`: List of disk utilization limits imposed on a per-filesystem basis for a given Operating System user group. Each list item is a dictionary whose structure is identical to the `disk_quotas_users` list items.
* `enable_root_fs_disk_quotas`: Whether or not to turn on disk quotas for the root filesystem. Defaults to `true`.


You can also have the role automatically enable quotas on the root filesystem using the `enable_root_fs_disk_quotas` variable, which defaults to `true`.

Dependencies
------------

A list of other roles hosted on Galaxy should go here, plus any details in regards to parameters that may need to be set for other roles, or variables that are used from other roles.

Example Playbook
----------------

It is often important to [impose disk utilization limits](https://wiki.archlinux.org/index.php/Disk_quota) on a certain [user account or user group](https://wiki.archlinux.org/index.php/Users_and_groups) to ensure that a rogue process or compromised service can not eat up all the available space on a given [filesystem](https://wiki.archlinux.org/index.php/File_systems). This can be trivially configured with the `disk_quotas_users` and `disk_quotas_groups` lists. For example, this snippet will configure the server to allow the `www-data` user (the account under which a typical Web server runs) access to no more than 50 gibibytes of space on the default filesystem:

```yaml
- name: Enforce disk quotas.
  hosts: webservers
  tasks:
    - name: Set disk quota for `www-data` user.
      import_role: "anarchotechnyc.disk-quotas"
      vars:
        disk_quotas_users:
          - name: www-data
            block_hard: 50G
```

License
-------

AGPL-3.0-or-later

Testing
-------

Use [Molecule](https://molecule.readthedocs.io/en/latest/) to run the tests. (You'll also need to install [VirtualBox](https://virtualbox.org/) and [Vagrant](https://vagrantup.com/), as tests are run in Vagrant-managed, VirtualBox-backed virtual machines.) Here's how to install Molecule into a virtual environment.

```sh
# Molecule is written in Python, so you'll also need Python.
python -m venv venv                                # Create your virtual environment.
source venv/bin/activate                           # Activate it.
pip install molecule ansible-lint molecule-vagrant # Install testing tools.

# Then, you can run the tests:
molecule test

# When you're done, deactivate your virtual environment.
deactivate
```
