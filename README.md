# s2i-vm
This repo allows to have s2i in machines that do not want to install docker




## Section 1. Create VMs

I recommend using kcli to create VMS for kvm. Check the [Github project](https://github.com/karmab/kcli) and the [documentation](https://buildmedia.readthedocs.org/media/pdf/kcli/latest/kcli.pdf) for more examples of configuration.

### Create the VM with kcli
The following yaml shows the configuration of one VM using kcli:

```yaml
rhel8.vm:
  template: /home/alvaro/vms/rhel-8.1-x86_64-kvm.qcow2
  numcpus: 2
  memory: 6144
  disks:
    - size: 50
  dns: 192.168.122.20
  cmds:
    - echo root | passwd --stdin root
    - useradd -G wheel user
    - echo user | passwd --stdin user
```

Modify `kcli-template-rhel.yml` to set different values of cpu, memory, disks, network configuration or even add new VMs. The kcli template also configures `root/root` and `user/user`.


These are all the commands needed to create and manage VMs using kcli:

```bash
kcli create plan -f kcli-template-rhel.yml rhel8 # Create the Kcli plan with name rhel8
kcli start plan rhel8   # Start the plan
kcli stop plan rhel8    # Stop the plan
kcli get vms            # Show VMs status
kcli delete plan rhel8  # Delete the plan
```


## Section 2. Configure files under group_vars

Modify these values inside `group_vars/all.yml`. `rh_subscription` values should be modified in the Ansible vault file.

```yaml
rhel:
  ip:   192.168.122.200
  host: rhel8.vm
  user: user

rh_subscription:
  username: "{{ vault_rh_subscription_username | mandatory }}"
  password: "{{ vault_rh_subscription_password | mandatory }}"
  poolid: "{{ vault_rh_subscription_poolid | mandatory }}"
```



## Section 3. Run the localhost configurer

```bash
ansible-playbook 01-configure-localhost.yml --become --ask-become-pass
```

If you do not have ansible, you can just use bash commands:

```bash
echo '192.168.122.200 rhel8.vm' | sudo tee -a /etc/hosts
echo "Host rhel8.vm
   User user" >> ~/.ssh/config
touch inventory
echo "[vms]
rhel8.vm" >> inventory
```



## Section 4. Run the rhel vm configurer

```bash
# Option 1
ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook 02-configure-vm.yml -i inventory --ask-vault-pass --ask-pass --become --ask-become-pass
    SSH password: user
    BECOME password[defaults to SSH password]: user
    Vault password: <your_password>

# Option 2
ansible-playbook 02-configure-vm.yml -i inventory --ask-vault-pass --extra-vars "ansible_user=user ansible_ssh_pass=user" --become
    Vault password: <your_password>
 ansible_password=yourpassword
```
