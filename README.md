# Infrastructure Monitoring Stack Setup with Ansible

Automate the deployment of a full monitoring stack — **Nginx**, **Grafana**, **Prometheus**, and **Node Exporter** — across multiple servers using Ansible.

---

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Step 1: Install Ansible on the Control Machine](#step-1-install-ansible-on-the-control-machine)
- [Step 2: Set Up SSH Key-Based Authentication](#step-2-set-up-ssh-key-based-authentication)
- [Step 3: Create the Ansible Inventory File](#step-3-create-the-ansible-inventory-file)
- [Step 4: Write the Ansible Playbook](#step-4-write-the-ansible-playbook)
- [Step 5: Deploy a Custom HTML Page](#step-5-deploy-a-custom-html-page)
- [Step 6: Execute the Playbook](#step-6-execute-the-playbook)
- [Step 7: Result](#step-7-result)
- [Project Structure](#project-structure)
- [References](#references)

---

## Overview

This project uses **Ansible** to automate setting up a complete infrastructure monitoring stack on remote servers. The control machine (master node) communicates with managed servers (slave nodes) over SSH and executes a YAML playbook that installs and configures all required services.

**Stack components:**

| Component | Purpose |
|---|---|
| Nginx | Web server — serves the custom HTML dashboard |
| Grafana | Monitoring visualization (dashboards & alerts) |
| Prometheus | Metrics collection and storage |
| Node Exporter | Exposes system-level metrics to Prometheus |

**Repo:** [ansible-monitoring-stack](https://github.com/kasturi-thanusha/ansible-monitoring-stack)

---

## Prerequisites

- One **control machine** (Ubuntu) with internet access
- One or more **managed/slave servers** (Ubuntu) reachable over the network
- A user with `sudo` privileges on the control machine
- SSH access to the managed servers

---

## Step 1: Install Ansible on the Control Machine

Ansible is an agentless automation tool installed only on the **controller node**. Install it using `apt`:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install software-properties-common -y
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible -y
ansible --version   # Verify the installation
cd ansible
```

> **Note:** Ansible does not need to be installed on the managed (slave) servers — only Python is required there, which is typically pre-installed on Ubuntu.

---

## Step 2: Set Up SSH Key-Based Authentication

Ansible communicates with managed servers over SSH. Configure passwordless SSH so Ansible can connect without manual intervention.

### 2.1 Generate an SSH Key Pair on the Control Machine

```bash
ssh-keygen
```

### 2.2 Retrieve the Public Key

```bash
ls /root/.ssh
cat /root/.ssh/id_ed25519.pub
```

Copy the output — this is your public key.

### 2.3 Add the Public Key to Each Slave Server

On each managed/slave server, open (or create) the `authorized_keys` file and paste the public key:

```bash
vim /root/.ssh/authorized_keys
```

### 2.4 Verify Passwordless Connection

From the control machine, SSH into a slave server using its public IP:

```bash
ssh <slave-public-ip>
```

If you connect without being prompted for a password, the setup is successful.

---

## Step 3: Create the Ansible Inventory File

An inventory file tells Ansible which servers to manage. Create one at `/root/ansible/inventory.ini`:

```bash
vim /root/ansible/inventory.ini
```

Add the **private IP addresses** of your slave servers, grouped logically:

```ini
[webservers]
172.31.34.228   # slave1
172.31.11.48    # slave2
```

> **Tip:** You can test connectivity to all listed hosts with:
> ```bash
> ansible -i inventory.ini all -m ping
> ```

---

## Step 4: Write the Ansible Playbook

A playbook is a YAML file containing ordered tasks that Ansible executes on the remote servers.

Create the playbook file:

```bash
vim /root/ansible/firstplaybook.yml
```

This playbook automates the following:

- ✅ Installing **Nginx** web server
- ✅ Adding the Grafana APT repository and installing **Grafana**
- ✅ Downloading and installing **Prometheus**
- ✅ Installing **Node Exporter** for system-level metrics
- ✅ Deploying a custom **HTML file** to the Nginx web directory

For the full playbook source, see the repository:
🔗 [https://github.com/kasturi-thanusha/ansible-monitoring-stack](https://github.com/kasturi-thanusha/ansible-monitoring-stack)

---

## Step 5: Deploy a Custom HTML Page

To serve a custom HTML dashboard via Nginx, place the HTML file inside a `files/` directory alongside the playbook.

### Directory Structure

```
ansible/
├── firstplaybook.yml
└── files/
    └── index.html
```

### Create the Files Directory and HTML File

```bash
mkdir -p /root/ansible/files
vim /root/ansible/files/index.html
```

Write your HTML content inside `index.html`.

### Playbook Task to Deploy the HTML File

```yaml
- name: Deploy external HTML file
  copy:
    src: files/index.html
    dest: /var/www/html/index.html
    owner: www-data
    group: www-data
    mode: '0644'
```

---

## Step 6: Execute the Playbook

Run the playbook against the inventory file using the `ansible-playbook` command:

```bash
ansible-playbook -i inventory.ini firstplaybook.yml
```

Ansible will:
1. Connect to each server in the inventory over SSH
2. Execute each task in the playbook sequentially
3. Report the status of every task (ok / changed / failed)
4. Ensure all services are running and enabled on boot

---

## Step 7: Result

Once the playbook completes successfully, all managed servers will have:

| Service | Default Port | Access |
|---|---|---|
| Nginx (HTML page) | `80` | `http://<server-ip>` |
| Grafana | `3000` | `http://<server-ip>:3000` |
| Prometheus | `9090` | `http://<server-ip>:9090` |
| Node Exporter | `9100` | `http://<server-ip>:9100/metrics` |

- All services are **started automatically** and configured to **start on boot**.
- Grafana connects to Prometheus as a data source to display live system metrics.
- The custom HTML page is served by Nginx and accessible via the server's public IP.

---

## Project Structure

```
ansible/
├── inventory.ini          # Hosts inventory (slave server IPs)
├── firstplaybook.yml      # Main Ansible playbook
└── files/
    └── index.html         # Custom HTML page deployed to Nginx
```

---

## References

- [Ansible Documentation](https://docs.ansible.com/)
- [Grafana Installation Guide](https://grafana.com/docs/grafana/latest/setup-grafana/installation/)
- [Prometheus Download Page](https://prometheus.io/download/)
- [Node Exporter GitHub](https://github.com/prometheus/node_exporter)
- [Project Repository](https://github.com/kasturi-thanusha/ansible-monitoring-stack)
