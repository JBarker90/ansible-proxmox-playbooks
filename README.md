# Description
This is a general repository for my Ansible playbooks for ProxMox VM/LXC setups.

# Pre-Requirement
You will need an SSH key configured on your Ansible controller. If you need to generate one, you can follow the instructions here:

- https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent

# Initial Setup Instructions
### 1. Add the file path for your SSH Ansible Key (from your Ansible controller node) in `ansible.cfg`

```
private_key_file = "~/.ssh/example_key"
```

### 2. Add the target IPs to the `~/ansible-proxmox-playbooks/inventory` file that you want Ansible to change.

### 3. Copy the example `example.yml` file located in `~/ansible-proxmox-playbooks/host_vars` and add your specific values for your target node you want to update.

```
cp -av ~/ansible-proxmox-playbooks/host_vars/example.yml <destination_IP>.yml
```

- Then, fill in the values in the host_vars file

Example:

```
target_user: admin
local_pub_key: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJuRgmQFyxO6bHyP4Bvma9MFKZGmqcj5f4LjF8QNOjai Example Key"
sudoers_template_file: sudoers_ubuntu.j2
bashrc_template_file: .bashrc_ubuntu.j2
```

NOTE: Make sure you use the template files specific to your distro (either AlmaLinux or Ubuntu). These can be found in `~/ansible-proxmox-playbooks/playbooks/roles/base/templates`.

- Additionally, the `local_pub_key: ` is for a Public SSH key that you might use on your local machine (NOT related to Ansible) to log into servers.  

### 4. Run `ping.yml` playbook to test connection.

```
ansible-playbook playbooks/ping.yml
```

### 5. Run `create_user.yml` playbook to create your new user that was configured in as `target_user: <username>` in `~/ansible-proxmox-playbooks/host_vars/<destination_IP>.yml`

```
ansible-playbook playbooks/create_user.yml
```

### 6. Run `initial_setup.yml` to configure `.bashrc` and install packages

```
ansible-playbook playbooks/initial_setup.yml
```

### 7. (Optional) Run `remove_user.yml` if you would like to remove the user you configured.

```
ansible-playbook playbooks/remove_user.yml
```

# Usage Example

- All playbooks will be located in `./ansible-proxmox-playbooks/playbooks` and will need to be run from the root project directory

```
ansible-playbook [playbooks/<file.yml>]
```

Example:

```
jonathan@dockerhost-01:~/ansible-proxmox-playbooks$ ansible-playbook playbooks/initial_setup.yml

PLAY [all] ***************************************************************************************

TASK [Gathering Facts] ***************************************************************************
ok: [192.168.30.24]

TASK [Update repo cache (AlmaLinux)] *************************************************************
skipping: [192.168.30.24]

TASK [Update repo cache (Ubuntu)] ****************************************************************
ok: [192.168.30.24]

PLAY [all] ***************************************************************************************

TASK [Gathering Facts] ***************************************************************************
ok: [192.168.30.24]

TASK [base : Add sudoers files for users] ********************************************************
changed: [192.168.30.24]

TASK [base : Add custom .bashrc for new user] ****************************************************
changed: [192.168.30.24]

TASK [base : Add SSH Public Key for new user] ****************************************************
changed: [192.168.30.24]

TASK [packages : Install various packages based on preference] ***********************************
changed: [192.168.30.24]

RUNNING HANDLER [base : restart_sshd] ************************************************************
changed: [192.168.30.24]

PLAY RECAP ***************************************************************************************
192.168.30.24              : ok=8    changed=5    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
```

# Notes

- This will run the playbooks as root for the remote user. So you will need to verify that your Ansible SSH key is present in `~/.ssh/authorized_keys` for root. 

- For security reasons, the `create_user.yml` playbook does NOT set a password. It will only configure an SSH key for your local machine to authenticate to your sudo user you create. If you want to set a sudo password, you will need to do it manually from your new server.

```
passwd <username>
```

- The `initial_setup.yml` will install specific packages. So if you would like to add or remove packages to install this will need to modified in `~/ansible-proxmox-playbooks/playbooks/roles/packages/tasks/main.yml`. These are the packages that are set:
    - vim
    - atop
    - htop
    - ncdu
    - screen
    - nmap

- These playbooks will only run on Ubuntu (or apt based distros) and AlmaLinux (or dnf based distros). If you would like to configure a different type of distro, then they will need to be configured in the project.