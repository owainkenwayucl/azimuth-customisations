# These customisations are for the slurm cluster

To use these you need to:

1. Create an `inventory.ini` file.

You can find the names of your nodes via slurm, e.g. 

```
[azimuth@owaink-testcluster-login-0 ansible-management]$ sinfo
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
standard*    up 60-00:00:0      3   idle owaink-testcluster-compute-standard-[0-2]
[azimuth@owaink-testcluster-login-0 ansible-management]$ 
```

Tells me that I have three nodes called `owaink-testcluster-compute-standard-0`, `owaink-testcluster-compute-standard-1` and `owaink-testcluster-compute-standard-2`, so my `inventory.ini` would look like:


```
[login]
localhost

[compute]
owaink-testcluster-compute-standard-0
owaink-testcluster-compute-standard-1
owaink-testcluster-compute-standard-2
```

2. Make sure that you have the SSH hostkeys in your known hosts file by SSHing into each of the nodes at least once, *or* if you are brave you can enable auto-accepting of new keys, adding:

```
Host *
	StrictHostKeyChecking accept-new
```

to `~/.ssh/config` on the login node. Make sure the permissions of that file are 600 (`chmod 600 ~/.ssh/config`).

3. Install ansible on your login node:

```
sudo dnf install -y ansible
```

4. Run `ansible-playbook -i inventory.ini tools.yaml` on your login node.

This will produce output like this:

```
[azimuth@owaink-testcluster-login-0 ansible-management]$ ansible-playbook -i inventory.ini tools.yaml

PLAY [all] *************************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************
ok: [localhost]
ok: [owaink-testcluster-compute-standard-0]
ok: [owaink-testcluster-compute-standard-1]
ok: [owaink-testcluster-compute-standard-2]

TASK [tools : Install quality of life tools] ***************************************************************************************************************
ok: [owaink-testcluster-compute-standard-1]
ok: [localhost]
ok: [owaink-testcluster-compute-standard-0]
ok: [owaink-testcluster-compute-standard-2]

TASK [tools : Install Python 3] ****************************************************************************************************************************
ok: [owaink-testcluster-compute-standard-1]
ok: [owaink-testcluster-compute-standard-0]
ok: [localhost]
ok: [owaink-testcluster-compute-standard-2]

TASK [tools : Install compilers] ***************************************************************************************************************************
ok: [owaink-testcluster-compute-standard-0]
ok: [owaink-testcluster-compute-standard-2]
ok: [localhost]
ok: [owaink-testcluster-compute-standard-1]

TASK [python312 : Install Python 3.12] *********************************************************************************************************************
ok: [owaink-testcluster-compute-standard-1]
ok: [owaink-testcluster-compute-standard-0]
ok: [owaink-testcluster-compute-standard-2]
ok: [localhost]

PLAY RECAP *************************************************************************************************************************************************
localhost                  : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
owaink-testcluster-compute-standard-0 : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
owaink-testcluster-compute-standard-1 : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
owaink-testcluster-compute-standard-2 : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

Although some of yours will say changed. Ansible is idempotent and I've already installed the packages so nothing changes!

The `tools.yaml` playbook runs two roles (`roles/tools` and `roles/python312`) which install some helpful tools (`vim`, `screen` etc) and compilers, as well as Python 3.12 which you can invoke by running `python3.12` on all the nodes.

If you want to add additional packages, look in `roles/tools/tasks/tools.yaml` to see how to add them to playbook and re-run `ansible-playbook -i inventory.ini tools.yaml`.

# Dangerous playbooks

Thee `update.yaml` playbook will update the OS install your nodes. This will break the OpenMPI installation. `reboot.yaml` will reboot the compute nodes only (not the login node).