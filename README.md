# jlejeune.home-ansible

These ansible playbooks configure a fresh deployment of a k3s cluster on 3 rapsberry pi 4.

## Prerequisites
Hardware:
* a workstation to run ansible commands
* 3 x raspberry pi 4 (4GB ram)

Configure the workstation:
* Install ansible
* Install collections/roles from Ansible Galaxy by running `ansible-galaxy install -r requirements.yml`
* Install sshpass and python-apt by running `sudo apt-get install sshpass python-apt -y`
* Install age and age-keygen binaries from https://github.com/FiloSottile/age/releases
* Install Mozilla sops binary from https://github.com/mozilla/sops/releases

Configure the raspberry pi:
* Flash SD-card with last Raspbian lite release (64bit): enable ssh and configure password for pi user: raspberry

## Setting up Age
Here we will create a Age Private and Public key. Using [SOPS](https://github.com/mozilla/sops) with [Age](https://github.com/FiloSottile/age) allows us to encrypt secrets and use them in Ansible and Flux.

1. Create a Age Private / Public Key

    ```sh
    age-keygen -o age.agekey
    ```

2. Set up the directory for the Age key and move the Age file to it

    ```sh
    mkdir -p ~/.config/sops/age
    mv age.agekey ~/.config/sops/age/keys.txt
    ```

## Secrets
All secrets are encrypted by Mozilla sops tool and are defined in <filename>.sops.yaml files
in inventory folder.

## Bootstrap your raspberry pis
You need to find the dynamic IPs given to your raspberry pis, you can use
nmap command to do so and fill your inventory/hosts.yml file with the temporary IPs:

```yaml
---
all:
  children:
    k3s_master:
      hosts:
        <MASTER_NAME>:
          ansible_host: <TEMPORARY_MASTER_IP>
    k3s_workers:
      hosts:
        <WORKER1_NAME>:
          ansible_host: <TEMPORARY_WORKER1_IP>
        <WORKER2_NAME>:
          ansible_host: <TEMPORARY_WORKER2_IP>
    k3s_cluster:
      children:
        k3s_master:
          hosts:
        k3s_workers:
          hosts:
```

Run the bootstrap ansible playbook on this inventory.
```sh
ansible-playbook playbooks/bootstrap.yml -u pi --ask-pass
```

You can now edit your inventory file to put the correct IPs you wanted
(defined in inventory/host_vars/ folder) and run the install-k3s.yml playbook.
```sh
ansible-playbook playbooks/install-k3s.yml
```

## Ansible roles

### common

 * Configure hostname
 * Set static IP
 * Configure locale
 * Create users and delete default pi user
 * Set the default editor
 * Setup a secure SSH configuration
 * Configure /boot/config.txt
 * Run raspi-config
 * Install and configure oh-my-zsh

### k3s

 * Download k3s binary
 * Configure k3s service on both nodes
 * Install and configure flux v2 to deploy other kubernetes components from a repository

## Ansible playbooks

### Bootstrap raspberry pi
```sh
ansible-playbook playbooks/bootstrap.yml
```

### Install k3s
```sh
ansible-playbook playbooks/install-k3s.yml
```

#### Uninstall k3s
```sh
ansible-playbook playbooks/uninstall-k3s.yml
```
