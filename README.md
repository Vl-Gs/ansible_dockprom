# Monitoring Stack Project

This project contains a comprehensive Ansible-based deployment and configuration management system for a monitoring stack with multiple services including Grafana, Prometheus, InfluxDB, MongoDB, Mosquitto MQTT, Nginx, and Node Exporter. Below you'll find detailed information about the project structure, deployment procedures, and system maintenance.

## Project Structure

```
.
├── group_vars/          # Global variables and configurations
├── inventory/          # Inventory definitions and host groups
│   └── inventory.yml   # Main inventory file with host configurations
├── kernel/            # Kernel-related configurations
├── playbooks/         # Ansible playbooks
│   └── main.yml      # Main deployment playbook
└── roles/            # Ansible roles
    ├── grafana/      # Grafana visualization platform
    ├── influxdb/     # Time series database
    ├── mongodb/      # Document database
    ├── mosquitto/    # MQTT message broker
    ├── nginx/        # Web server and reverse proxy
    ├── node_exporter/ # System metrics collection
    └── prometheus/   # Metrics collection and storage
```

## Key Components

1. **Inventory Configuration** (`inventory/inventory.yml`):
   - Defines the host groups and their variables
   - Contains server definitions and groupings
   - Default SSH configuration:
     ```yaml
     ansible_user: your_ssh_user
     ansible_ssh_private_key_file: /path/to/your/private/key
     ```
   - To add new servers:
     1. Edit `inventory/inventory.yml`
     2. Add hosts under appropriate groups (e.g., ml_servers)
     3. Define host-specific variables if needed

2. **Global Variables** (`group_vars/all.yml`):
   - Contains project-wide settings
   - Defines common variables like base directories and ports
   - Key configurations:
     ```yaml
     monitoring_base_dir: /opt/monitoring
     docker_compose_version: "3.8"
     docker_network_name: monitoring
     ports:
       prometheus: 9090
     ```
   - To modify global settings:
     1. Edit `group_vars/all.yml`
     2. Update or add new variables as needed

3. **Main Playbook** (`playbooks/main.yml`):
   - Orchestrates the deployment process
   - Includes various role and task includes
   - To modify deployment flow:
     1. Edit `playbooks/main.yml`
     2. Add or modify task includes
     3. Update role assignments

## Service Details

### Grafana
- Configuration in `roles/grafana/`
- Uses Docker container deployment
- Requires network connectivity to Prometheus/InfluxDB
- Supports Node Exporter integration
- Data storage in mounted volume
- Network integration with monitoring network

### Prometheus
- Configuration in `roles/prometheus/`
- Data stored in `{{ prometheus_data_dir }}`
- Configuration files in `{{ prometheus_config_dir }}`
- Rules directory at `prometheus_config_dir/rules`
- Main configuration file: prometheus.yml

### MongoDB
- Configuration in `roles/mongodb/`
- Directories structure:
  - Config: `{{ mongodb_config_dir }}`
  - Data: `{{ mongodb_data_dir }}`
  - Logs: `{{ mongodb_log_dir }}`
- MongoDB exporter for metrics collection

### InfluxDB
- Configuration in `roles/influxdb/`
- Uses Docker container
- Includes InfluxDB exporter for metrics
- Network integration with monitoring stack

### Mosquitto
- Configuration in `roles/mosquitto/`
- MQTT broker service
- Includes Mosquitto exporter
- Containerized deployment

### Nginx
- Configuration in `roles/nginx/`
- Reverse proxy configuration
- Default port: 80
- Configuration mounted from `{{ nginx_config_dir }}`

### Node Exporter
- Configuration in `roles/node_exporter/`
- System metrics collection
- Deployed on monitored hosts
- Exports metrics to Prometheus

## Project Maintenance and Extension Guide

### Adding New Hosts to Inventory

1. **Edit inventory/inventory.yml**:
   
   2. **Define Host Variables**:
   - Add host-specific variables under each host
   - Common variables include:
     - ansible_host
     - ansible_port (if non-standard)
     - ansible_user (if different from global)
     - service-specific variables

3. **Group Configuration**:
   - Create new groups under `children:`
   - Group hosts by role or function
   - Define group_vars if needed

### Adding New Services

To add a new service to the monitoring stack:

1. **Create a New Role Structure**:
   ```bash
   mkdir -p roles/new_service/{tasks,templates,defaults,handlers,vars}
   touch roles/new_service/tasks/main.yml
   touch roles/new_service/defaults/main.yml
   touch roles/new_service/handlers/main.yml
   ```
  
2. **Define Role Components**:
   - Create `roles/new_service/tasks/main.yml` for service tasks
   - Add templates in `roles/new_service/templates/`
   - Define default variables in `roles/new_service/defaults/main.yml`
   - Add handlers in `roles/new_service/handlers/main.yml`

3. **Update Inventory**:
   - Add new host group in `inventory/inventory.yml` if needed
   - Define service-specific variables

4. **Update Global Variables**:
   - Add service configuration in `group_vars/all.yml`
   - Define ports, paths, and other settings

5. **Include in Main Playbook**:
   - Edit `playbooks/main.yml`
   - Add new role or include_tasks as needed

## Required Files for New Services

When adding a new service, you need to create or modify these files:

1. **Role Structure** (`roles/new_service/`):
   - `tasks/main.yml`: Service deployment tasks
   - `defaults/main.yml`: Default variables
   - `handlers/main.yml`: Service restart handlers
   - `templates/`: Configuration templates
   - `vars/main.yml`: Service-specific variables

2. **Inventory Updates** (`inventory/inventory.yml`):
   - Add new host group if needed
   - Define service-specific hosts
   - Set host variables

3. **Global Variables** (`group_vars/all.yml`):
   - Add service ports
   - Define base directories
   - Set global configurations

4. **Playbook Integration** (`playbooks/main.yml`):
   - Add new role inclusion
   - Define host targeting
   - Set role tags

## Maintenance Procedures

1. **Adding New Hosts**:
   - Edit `inventory/inventory.yml`
   - Add host under appropriate group
   - Define host variables
   - Run playbook with `--limit` to target new hosts

2. **Updating Configurations**:
   - Modify files in `group_vars/`
   - Update role defaults or templates
   - Re-run playbook to apply changes

3. **Service Management**:
   - Use role-specific tasks for service operations
   - Update service configurations in respective role directories
   - Use tags to run specific parts of playbook:
     ```bash
     ansible-playbook -i inventory/inventory.yml playbooks/main.yml --tags "service_name"
     ```

## Configuration Management

### Directory Structure Management
1. Each service has its own role directory under `roles/`
2. Service configurations are stored in their respective directories
3. Templates are used for dynamic configuration files
4. Handlers manage service restarts and reloads

### Variable Management
1. Global variables in `group_vars/all.yml`
2. Role-specific variables in `roles/<role>/defaults/main.yml`
3. Host-specific variables in inventory files
4. Sensitive data should be encrypted with ansible-vault

### Network Configuration
1. Services communicate through Docker network
2. Default network name: monitoring
3. Exposed ports are defined in global variables
4. Nginx handles external access

## Deployment

To deploy the entire stack:
```bash
ansible-playbook -i inventory/inventory.yml playbooks/main.yml
```

To deploy specific components:
```bash
ansible-playbook -i inventory/inventory.yml playbooks/main.yml --tags "component_name"
```

## Best Practices

1. Always test changes in a development environment first
2. Use version control for all configuration changes
3. Document any custom modifications
4. Keep service configurations in their respective roles
5. Use meaningful tags for selective execution
6. Maintain consistent naming conventions

## Troubleshooting

1. Check logs in `/opt/monitoring/` directory
2. Verify service configurations in respective role directories
3. Ensure all required variables are properly defined
4. Check connectivity and permissions on target hosts