Below is a well-formatted README file based on your provided content:

---

---

## Ansible Roles and Playbook

### Overview

This playbook is designed to automate the setup and deployment of Docker-based applications, including the installation of Docker, Portainer, NGINX, and Webmin. It also configures Docker networks and containers and deploys services with custom NGINX configurations. The playbook includes two roles for installation and deployment tasks.

### Structure

```
Ansible_roles
│   inventory.ini
│   site.yml
├───roles
│   ├───deploy
│   │   ├───defaults
│   │   │       main.yml
│   │   ├───handlers
│   │   │       main.yml
│   │   ├───meta
│   │   ├───tasks
│   │   │       main.yml
│   │   │
│   │   ├───templates
│   │   │       miniserv.conf.j2
│   │   │       nginx_config.conf.j2
│   │   │       systemd_container.service.j2
│   │   │
│   │   ├───tests
│   │   └───vars
│   │           main.yml
│   │
│   └───install
│       ├───defaults
│       │       main.yml
│       ├───handlers
│       │       main.yml
│       ├───meta
│       ├───tasks
│       │       install_docker.yml
│       │       main.yml
│       │
│       ├───tests
│       └───vars
│               main.yml
│
└───vars
        main.yml
```

### site.yml

The `site.yml` file serves as the entry point for setting up the server and deploying applications. It orchestrates the execution of different roles and tasks defined in the playbook, ensuring a consistent and automated deployment process across all targeted hosts.

In our `site.yml`, we implemented dynamic role inclusion, where the roles to be executed are passed as a variable through the `roles_list` variable, enabling flexibility in determining which roles to include in the playbook run. This allows easy adjustments to the deployment process by simply updating the `roles_list` variable, adding or removing roles without modifying the core structure of the playbook itself.

### Execution Flow

1. The playbook targets all hosts defined in the inventory file, applying the configurations and deployments universally.
2. The playbook elevates privileges using `become: true` to ensure that tasks requiring administrative access can be executed without issues.
3. The `vars/main.yml` file is loaded, providing all the necessary variables, including the `roles_list` variable and other configurations needed for the subsequent tasks and roles.
4. The `Include roles dynamically` task iterates over the `roles_list` variable, including and executing each specified role.
5. We can also execute specific tasks or sets of tasks by using tags. Tags provide a convenient way to manage and execute only certain parts of a playbook without running the entire set of tasks. This can be particularly useful for testing, debugging, or running specific configuration updates.
to run playbook :
```bash
ansible-playbook -i inventory.ini site.yml
```
To run specific tasks associated with a tag, use the `--tags` option when executing the playbook:
```bash
ansible-playbook playbook.yml --tags "docker"
```

Alternatively, use the `--skip-tags` option to exclude tasks with certain tags:
```bash
ansible-playbook playbook.yml --skip-tags "nginx"
```

### Inventory.ini

The `inventory.ini` file is a central component in Ansible, used to define the hosts and groups of hosts that the playbooks will target. It allows for organized and flexible management of multiple environments, such as development, staging, and production. Each server can be specified by its hostname, IP address, or both.

For example:
```
[webservers]
server1.example.com
server2.example.com

[dbservers] 
192.168.2.20 
192.168.2.21
```

### Variables

The variables file `Ansible_roles/vars/main.yml` defines key parameters and settings used throughout the Ansible playbook. Below are the key variables along with their descriptions and types:

- **gitlab_access_token**  
  **Description:** Token used for authenticating with the GitLab Container Registry.  
  **Type:** String

- **webmin_port**  
  **Description:** Specifies the port number on which Webmin will be accessible.  
  **Type:** Integer

- **portainer_port**  
  **Description:** Specifies the port number on which the Portainer web interface will be accessible.  
  **Type:** Integer

- **portainer_credentials**  
  **Description:** Defines the administrator credentials for accessing Portainer.  
  **Type:** Dictionary  
  **Sub-Variables:**  
  - **ADMIN_USERNAME:** The administrator username. (String)  
  - **ADMIN_PASS:** The administrator password. (String)

  **Example:**
  ```yaml
  portainer_credentials:
      ADMIN_USERNAME: "admin"
      ADMIN_PASS: "admin1234567890"
  ```

- **containers**  
  **Description:** A list of containers to be deployed, each defined by specific parameters.  
  **Type:** List of Dictionaries  
  **Container Dictionary Keys:**
  - **container_name:** The name of the Docker container. (String)
  - **container_image:** The Docker image used for the container. (String)
  - **environment_variables:** List of environment variables for the container.
  - **container_port:** Port exposed by the container.
  - **host_port:** Port of the host.
  
  **Example:**
  ```yaml
  containers:
      - container_name: "mycontainer"
        container_image: "myimage"
        container_port: 80
        host_port: 8090
        environment_variables:
          ENV_VAR1: "var1"
          ENV_VAR2: "var2"
  ```

- **nginx_configs**  
  **Description:** A list of NGINX configuration files, each defining server settings and proxy pass rules.  
  **Type:** List of Dictionaries  
  **Configuration Dictionary Keys:**
  - **domain_name:** The server name or domain. (String)
  - **proxy_pass_port:** The port to which requests will be forwarded based on localhost. (Integer)

  **Example:**
  ```yaml
   nginx_configs:
      - domain_name: "project.com"
        proxy_pass_port: 9000
  ```

---

## Roles Overview

As outlined in the playbook structure section, each role follows a standard layout initialized with Ansible Galaxy. This structure includes subdirectories such as `tasks`, `vars`, `templates`, and `handlers`, among others. These directories organize and encapsulate different components, making it easier to manage and reuse our Ansible configurations.

### 1. Install Role

The `install` role is responsible for setting up essential software and services needed for the deployment environment. This role includes the following key tasks in the `/roles/install/tasks/` directory:

- **install_docker.yml**:  
  A specific task file dedicated to adding the Docker GPG key, Docker repository, installing Docker, and setting up access to the GitLab Container Registry.

- **main.yml**:  
  The primary task file that orchestrates the overall installation process, containing the following tasks:
  1. **Docker Installation**: Installs Docker along with its dependencies to enable containerization by invoking the `install_docker.yml` task file.
  2. **Portainer Setup**: Deploys Portainer, a lightweight management UI, which allows users to easily manage Docker environments.
  3. **NGINX Installation**: Installs the NGINX web server, which can be used as a reverse proxy, load balancer, and HTTP cache.
  4. **Webmin Installation**: Installs Webmin, a web-based interface for system administration for Unix.

### 2. Deploy Role

The `deploy` role manages the deployment of services, including configuring NGINX, Webmin, and setting up Docker containers. This role includes the following key tasks in the `/roles/deploy/tasks/main.yml

` file:

1. **Copying the Webmin Configuration File:** This task utilizes the `miniserv.conf.j2` Jinja2 template to copy the Webmin configuration file from the template to the target location.
2. **Copying Systemd Configuration Files:** This task copies the systemd configuration files to the appropriate directory, ensuring that Docker containers are started and managed correctly by the system.
3. **Copying NGINX Configuration Files:** This task uses the `nginx_config.conf.j2` template to copy the NGINX configuration files to the target location, setting up the necessary proxy and server blocks for the deployed services.
4. **Deploying Docker Containers:** This task deploys Docker containers as specified in the variable files, ensuring that each container is started with the correct environment variables, ports, and settings.
5. **Restarting the Necessary Services:** This task ensures that the appropriate services, such as NGINX and Webmin, are restarted to apply the new configurations.
   


---

### Templates

The following templates in `roles/deploy/templates/` are utilized in the deployment process to generate the necessary configurations. They use Jinja2 syntax to dynamically insert variable values, ensuring flexibility and reusability across different environments and setups.

#### 1. systemd_container.service.j2

This template defines a systemd service unit for managing Docker containers. It includes configuration details for container management, such as starting, stopping, and restarting containers.

**Sections:**
- **[Unit]:** Describes the service and specifies dependencies.
- **[Service]:** Includes pre-start commands for stopping and removing existing containers, pulling the latest container image, and running the container with the specified network, volume, and port configurations.
- **[Install]:** Specifies the target for enabling the service.

**Key Variables:**
- `item.container_name`: Name of the Docker container.
- `item.container_image`: Docker image to use for the container.
- `network_name`: Docker network to which the container belongs.
- `base_directory`: Base directory for volume mappings.
- `item.container_ports`: Port mappings for the container.

#### 2. nginx_config.conf.j2

This template is used to create NGINX configuration files for setting up virtual hosts. It defines server blocks with specific proxy settings for forwarding requests to backend services.

**Server Block Details:**
- **listen:** Port on which the server listens (default is 80 for HTTP).
- **server_name:** Domain name associated with the server.
- **location /:** Proxy settings for forwarding client requests to the backend service, including necessary headers.

**Key Variables:**
- `item.domain_name`: Domain name of the server.
- `item.proxy_pass_port`: Backend service to which requests are proxied.

```nginx
server {
    listen 80;
    server_name {{ item.domain_name }};
    
    location / {
        proxy_pass http://localhost:{{ item.proxy_pass_port }};
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

#### 3. miniserv.conf.j2

This template configures Webmin, a web-based system administration interface. It specifies various parameters and settings for the Webmin service.

**Key Configurations:**
- **port:** The port on which Webmin listens.
- **ssl:** SSL settings to ensure secure connections.
- **logfile and error log:** Paths to log files for tracking service activity and errors.
- **userfile:** Path to the user file for authentication.

**Key Variables:**
- `webmin_port`: Port on which Webmin is configured to listen.

---



---



---



---