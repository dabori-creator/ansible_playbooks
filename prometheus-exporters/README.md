# Ansible Playbook for Prometheus Exporters

This playbook provides a streamlined way to deploy multiple Prometheus exporters across your infrastructure. Each exporter is installed and configured with secure defaults and can be selectively deployed using Ansible groups and tags.

## Prerequisites

- **Ansible** 2.9 or higher

- **Target hosts** with:
  
  - Linux operating system
  
  - `systemd` for service management
  
  - `curl` or `wget` for downloading binaries
  
  - Docker (for container-based exporters)
  
  - PostgreSQL (for PostgreSQL exporter)
    
    ### Network Requirements

- Access to Nexus repository for binary downloads

- Port availability for each exporter (see port mapping below)
  
  ## Supported Exporters
  
  | Exporter                   | Version | Port | Type   | Description                                       |
  | -------------------------- | ------- | ---- | ------ | ------------------------------------------------- |
  | Node Exporter              | 1.10.2  | 8501 | Binary | System metrics (CPU, memory, disk, network, etc.) |
  | cAdvisor                   | v0.57.0 | 8506 | Docker | Container resource usage and performance metrics  |
  | PostgreSQL Exporter        | 0.18.1  | 8503 | Binary | PostgreSQL database performance metrics           |
  | Blackbox Exporter          | v0.28.0 | 8510 | Docker | Endpoint monitoring (HTTP/HTTPS)                  |
  | X.509 Certificate Exporter | 3.19.1  | 8511 | Docker | SSL/TLS certificate expiration monitoring         |
  
  ## Installation
  
  ### 1. Clone the Repository
  
  ```bash
  git clone https://github.com/yourusername/grafana-exporters-ansible.git
  cd grafana-exporters-ansible
  ```
  
  ### 2. Configure Variables
  
  Update the `vars` section in the playbook with your environment details:
  
  ```yaml
  vars:
  nexus_base_url: "https://nexus.your-domain.com/repository/your-repo/devops"
  nexus_docker_registry: "nexus.your-domain.com:443"
  ```
  
  ### 3. Create Inventory File
  
  Create an inventory file (`inventory.ini`) with your hosts grouped by exporter type:
  
  ```ini
  [all]
  server1 ansible_host=192.168.1.10
  server2 ansible_host=192.168.1.11
  [node_exporter]
  server1
  server2
  [docker_exporter]
  server1
  [postgres_exporter]
  server2
  [blackbox_exporter]
  server1
  [x509_exporter]
  server1
  ```
  
  ### 4. Run the Playbook
  
  ```bash
  # Install all exporters
  ansible-playbook -i inventory.ini install-exporters.yml
  ```
  
  ## Usage
  
  ### Basic Commands
  
  ```bash
  ansible-playbook -i inventory.ini exporters.yml
  ```
  
  ### Managing Services
  
  Each exporter runs as a systemd service or Docker container:
  
  ```bash
  # Systemd services (Node, PostgreSQL exporters)
  systemctl status node_exporter
  systemctl restart postgres-exporter
  systemctl enable node_exporter
  # Docker containers (cAdvisor, Blackbox, X.509 exporters)
  docker ps | grep cadvisor
  docker logs blackbox
  docker restart x509-cert
  ```
  
  ## Tags
  
  Use these tags to control which exporters to install:
  
  | Tag                 | Description                                         |
  | ------------------- | --------------------------------------------------- |
  | `node_exporter`     | Install Node Exporter                               |
  | `docker_exporter`   | Install cAdvisor                                    |
  | `postgres_exporter` | Install PostgreSQL Exporter                         |
  | `blackbox_exporter` | Install Blackbox Exporter                           |
  | `x509_exporter`     | Install X.509 Certificate Exporter                  |
  | `verify`            | Run verification checks for all installed exporters |
  
  ## Exporter Details
  
  ### Node Exporter (Port: 8501)
  
  Installs as a systemd service with custom collection flags:

- **Collectors enabled**: systemd, cpu, conntrack, cpufreq, filefd, netclass, netstat, nfs, time, meminfo, uname, filesystem, netdev, vmstat, diskstats, stat, softnet, sockstat, pressure, powersupplyclass, schedstat, entropy, loadavg, arp, os
  
  ### cAdvisor (Port: 8506)
  
  Runs as a Docker container with mounted volumes for container and host metrics.
  
  ### PostgreSQL Exporter (Port: 8503)
  
  Installs as a systemd service with:

- Default connection string: `postgresql://postgres_exporter:password@localhost/postgres?sslmode=disable`

- **Important**: Update the password in the service file for production use
  
  ### Blackbox Exporter (Port: 8510)
  
  Runs as a Docker container with configuration file at `/etc/blackbox/blackbox.yml`.
  Pre-configured modules:

- `http_2xx`: HTTP endpoint monitoring

- `ecp_cert`: SSL certificate monitoring with 35-minute timeout
  
  ### X.509 Certificate Exporter (Port: 8511)
  
  Monitors SSL/TLS certificates in `/etc/pki/ca-trust/source/anchors/` directory.
  
  ## Verification
  
  The playbook includes automatic verification tasks that check if each exporter is running and accessible:
  
  ```bash
  # Run verification separately
  ansible-playbook -i inventory.ini install-exporters.yml --tags verify
  # Check individual exporters
  curl http://localhost:8501/metrics
  curl http://localhost:8506/metrics
  ```
  
  ## Configuration Customization
  
  ### Modifying Node Exporter Collectors
  
  Edit the `ExecStart` line in the Node Exporter systemd service definition:
  
  ```yaml
  ExecStart=/usr/bin/node_exporter-{{ node_exporter_version }}.linux-amd64/node_exporter \
  --web.listen-address=:8501 \
  --collector.disable-defaults \
  --collector.systemd \
  --collector.cpu
  ```
  
  ### Changing PostgreSQL Connection
  
  Update the `DATA_SOURCE_NAME` environment variable in the PostgreSQL exporter service:
  
  ```yaml
  Environment=DATA_SOURCE_NAME="postgresql://exporter_user:new_password@localhost/dbname?sslmode=disable"
  ```
  
  ### Adding Blackbox Modules
  
  Edit `/etc/blackbox/blackbox.yml` to add custom monitoring modules:
  
  ```yaml
  modules:
  custom_https:
    prober: http
    timeout: 10s
    http:
      valid_status_codes: [200, 302]
      method: GET
  ```
  
  ## Troubleshooting
  
  ### Common Issues
1. **Permission Denied**
   
   ```bash
   # Check file ownership and permissions
   ls -la /usr/bin/node_exporter-1.10.2.linux-amd64/
   ```

2. **Port Already in Use**
   
   ```bash
   # Find process using the port
   lsof -i :8501
   # Stop the conflicting service
   systemctl stop node_exporter
   ```

3. **Docker Container Fails to Start**
   
   ```bash
   # Check container logs
   docker logs cadvisor
   # Verify Docker is running
   systemctl status docker
   ```

4. **PostgreSQL Connection Issues**
   
   ```bash
   # Test connection as postgres user
   sudo -u postgres psql -c "SELECT 1;"
   # Check exporter logs
   journalctl -u postgres-exporter -f
   ```
   
   ### Debug Mode
   
   Run with increased verbosity:
   
   ```bash
   ansible-playbook -i inventory.ini install-exporters.yml -vvv
   ```
   
   # 
