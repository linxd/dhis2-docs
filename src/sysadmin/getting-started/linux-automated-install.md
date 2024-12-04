# 介绍 { #getting_started_linux_automated_install }

DHIS2 是一个免费访问、开源且灵活的软件平台，旨在收集、管理、可视化和分析来自不同来源和项目的健康数据。DHIS2 的组件包括代理服务器（如 Nginx 或 Apache2）、Tomcat 服务器、PostgreSQL 数据库以及可选的应用性能管理和服务器监控工具。本快速入门指南将向您展示如何使用 dhis2-server 工具在单个服务器上安装 DHIS2 及其组件。

#准备工作 {#getting_started_prerequisites }
--运行 Ubuntu 22.04 或 24.04 的服务器
--具有 sudo 权限的非 root 用户的 SSH 访问权限
##安装 DHIS2

1. Make sure your server’s firewall is active and the SSH port is allowed. Replace `{ssh_port}` with your actual SSH port number.
   ```
   sudo ufw limit {ssh_port}/tcp
   sudo ufw enable
   ```
2. Connect to your server via SSH and clone the repository from "https://github.com/dhis2/dhis2-server-tools".
   ```
   git clone https://github.com/dhis2/dhis2-server-tools.git
   ```
3. Run the installation
   ```
   cd dhis2-server-tools/deploy
   cp inventory/hosts.template inventory/hosts
   sudo ./deploy.sh
   ```
4. Open the DHIS2 web interface at `https://{server_ip}/dhis,` replacing `{server_ip}` with your actual IP address. Use the default credentials: `admin` for the username and `district` for the password.
   ```
   https://{server_ip}/dhis
   ```

## Next steps

### Configure fully qualified domain name

- Edit your inventory hosts file and set the `fqdn` variable, use an editor of your choice.
  ```
  vim inventory/hosts
  fqdn=dhis.example.com
  ```
- Save your changes and run the install again,

  ```
  sudo ./deploy.sh
  ```

  > **Important**
  >
  > To use Let's Encrypt, ensure the domain is mapped to your server's public IP address before setting `fqdn`. Alternatively, you can use a custom TLS certificate.

### Adding an instance

You can run multiple instances on a single server. Adding an instance will create a separate lxd container.

- Edit `dhis2-server-tools/deploy/inventory/hosts` file in
  ```
  vim dhis2-server-tools/deploy/inventory/hosts
  ```
- Add a new line line under `[instances]` section, should look like the line below,
  ```
  [instances]
  dhis    ansible_host=172.19.1.11   database_host=postgres # your first instance
  hmis    ansible_host=172.19.1.12   database_host=postgres # your second instance
  ```

> **Note**
>
> The name `hmis` and ansible_host `172.19.1.12` should be unique.

### Deploying custom TLS certificate to the reverse proxy

On some occasions, you could have your own TLS certificate and you are not using LetsEncrypt. Here is how you can instruct the tools to use your own TLS certificate.

> **Note**
>
> You'll need to have your TLS certificate file and its corresponding key.

- Copy TLS certificate and key to `dhis2-server-tools/deploy/roles/proxy/files/` They should be named `customssl.crt` and `customssl.key` respectively.

- Configure the tools to use the copied TLS certificate and key by editing your `inventory/hosts` file and setting `SSL_TYPE` parameter to `customssl` , see below,

```
SSL_TYPE=customssl
```

### PostgreSQL Optimization

The tools provide options to configure certain PostgreSQL variables for improved performance (see [PostgreSQL performance tuning](#install_postgresql_performance_tuning)). These variables can be defined in your inventory file, in-line with your host line in `key=value` format. For more details, refer to [Ansible host variables documentation](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html#assigning-a-variable-to-one-machine-host-variables). Alternatively, if your host is named `postgres` in the hosts file, you can create a file named `postgres` in the `dhis2-server-tools/deploy/inventory/host_vars/` directory and define the variables, in `key: value` format . Templates are available in the directory to help you get started.

The list of variables can be found here: [PostgreSQL Optimization Variables](#dhis2_server_tools_postgresql_variables)

### DHIS2 Instance variables

These are variables specific to the dhis2 instance, like PostgreSQL variables, you can either define them in inventory host or in the file you create in `dhis2-server-tools/deploy/inventory/host_vars/`, there is `dhis.template` that ships with the tools, create a file from the template with e.g.

```
cp dhis2-server-tools/deploy/inventory/host_vars/dhis.template dhis2-server-tools/deploy/inventory/host_vars/dhis

```

The list of variables can be found here:  [Instance Config variables](#dhis2_server_tools_instance_variables)
