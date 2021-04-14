## Ansible playbook for formatting an usb disk device and to mount it

Make sure that each pi systems has an usb block device added in the same USB port and preferably also with the same size.

Before running our `site.yml` playbook install the following ansible functions:

```bash
ansible-galaxy collection install community.general
ansible-galaxy collection install ansible.posix
```

Update the `hosts.ini` file with your pi system names.

Then run: `ansible-playbook site.yml`

This will also mount the usb block device on file system `/app/longhorn`
