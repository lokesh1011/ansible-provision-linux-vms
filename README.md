# Ansible – Provision Linux VMs

Ansible playbook that provisions 3 AWS Linux (EC2) web servers from an
`ansible-master` node.

## What the playbook does
- Runs against the **`aws`** host group (the 3 EC2 workers).
- Installs & starts **firewalld**, then opens the ports in the `ports` group
  variable (**80, 443**) with a **loop**, permanent + immediate.
- In a **block**, `stat`s `/var/www/html/www.companyplus.com` and uses `debug`
  to print an **error if missing** / a **message if it exists**.
- Calls the **`webserver`** role, which:
  - installs **nginx** via the `package` module using the role var `webserver=nginx`,
  - creates `/var/www/html/www.companyplus.com` and
    `/var/www/html/www.companypulsar.com` (owner/group **nginx**, mode **0770**)
    with a **loop**,
  - **templates** `nginx.conf.j2` (Jinja: `app_root`, `server_name`,
    `document_root`),
  - notifies a **handler** that restarts nginx.
- The role is called with `app_root: html_demo_site-main`,
  `server_name: "{{ ansible_default_ipv4.address }}"`,
  `document_root: /var/www/html`.

## Layout
```
.
├── ansible.cfg
├── inventory.ini            # fill the [aws] group with your 3 worker DNS names
├── group_vars/aws.yml       # ports: [80, 443]
├── playbook.yml
└── roles/webserver/
    ├── tasks/main.yml
    ├── vars/main.yml         # webserver: nginx
    ├── handlers/main.yml     # Restart nginx
    └── templates/nginx.conf.j2
```

## One-time AWS/EC2 setup
1. Create an EC2 instance named **`ansible-master`**, connect, then:
   ```bash
   sudo hostnamectl set-hostname ansible-master
   sudo dnf install -y ansible
   ssh-keygen -t rsa -b 4096          # accept defaults
   cat ~/.ssh/id_rsa.pub              # copy this public key
   ```
2. Create **3 more** EC2 instances (the workers). On **each**, append the
   master's public key:
   ```bash
   vi ~/.ssh/authorized_keys          # paste the id_rsa.pub line, save
   ```
3. On `ansible-master`, get this project and fill the inventory:
   ```bash
   git clone <your-repo-url> && cd <repo>
   # put each worker's Private IPv4 DNS name in the [aws] group of inventory.ini
   ```

## Run
```bash
ansible-playbook -i inventory.ini playbook.yml
```
Expect `ok`/`changed` with **0 failed** on all 3 hosts.

> Note: the playbook uses the `ansible.posix.firewalld` module. The full
> `ansible` package includes it; if you ever see "collection not found", run
> `ansible-galaxy collection install ansible.posix`.

## Submission
Push everything to your git repo and submit the repository link.
