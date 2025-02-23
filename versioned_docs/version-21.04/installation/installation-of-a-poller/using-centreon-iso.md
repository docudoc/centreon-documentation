---
id: using-centreon-iso
title: Using Centreon ISO
---

> If you want to install Centreon on CentOS / Oracle Linux / RHEL distribution
> in version 8, you must [use RPM packages](./using-packages.md)

## Prerequisites

Download the ISO from [Centreon's download site](https://download.centreon.com/).

## Step 1: Startup the server

To install a Centreon Poller, start up your server from the Centreon ISO image in version el7.
Start up with **Install CentOS 7**:

![image](../../assets/installation/01_bootmenu.png)

## Step 2: Choose a language

Choose the language for the installation process and click on **Continue**:

![image](../../assets/installation/02_select_install_lang.png)

## Step 3: Select the component

Click on the **Installation Type** menu:

![image](../../assets/installation/03_menu_type_install.png)

You have different options to choose from:

![image](../../assets/installation/07installpoller.png)

* **Central with database**: Install Centreon (web interface and database), monitoring engine and Broker.
* **Central without database**: Install Centreon (web interface only), monitoring engine and Broker.
* **Poller**: Install poller (monitoring engine and Broker only).
* **Database**: Install database server (if you have already installed a **Central server without a database** option).

Select **Poller** and click **Done**.

## Step 4: System configuration

### Configure disk partitioning

Click on the **Installation Destination** menu:

![image](../../assets/installation/05_menu_filesystem.png)

Select the hard disk drive and the **I will configure partitioning** option. Then click on **Done**:

![image](../../assets/installation/06_select_disk.png)

Using the **+** button, create your own partitioning file system following the instructions in
[prerequisites chapter](../prerequisites.md). Then click on **Done**:

![image](../../assets/installation/07_partitioning_filesystem.png)

> It is recommended to use LVM as the default partitioning scheme.

A confirmation window appears. Click on **Accept Changes** to validate the partitioning:

![image](../../assets/installation/08_apply_changes.png)

### Configure the timezone

Click on the **Date & Time** menu:

![image](../../assets/installation/11_menu_timezone.png)

Select the time zone and then click the gear button to configure the NTP server:

![image](../../assets/installation/12_select_timzeone.png)

Type in the name of the NTP server you wish to use and click the plus button. Or, select one from the list of
predefined NTP servers then click **OK** and then **Done**:

![image](../../assets/installation/13_enable_ntp.png)

> It is okay that you can't enable the “network time” option in this screen. It will become enabled automatically when
> you configure the network and hostname.

### Configure the network

Click on the **Network & Hostname** menu:

![image](../../assets/installation/09_menu_network.png)

Enable all network interfaces by clicking the button in the top right from **off** to **on**. Then click on **Done**:

![image](../../assets/installation/10_network_hostname.png)

## Begin the installation

Once configuration is complete, click on **Begin Installation**:

![image](../../assets/installation/14_begin_install.png)

Click on **Root Password**:

![image](../../assets/installation/15_menu_root_password.png)

Define and confirm the **root** user password. Click on **Done**:

![image](../../assets/installation/16_define_root_password.png)

Wait for the installation process to finish. You can also use this time to add additional users to the system if you
desire.

![image](../../assets/installation/17_wait_install.png)

When the installation is complete, click on **Reboot**:

![image](../../assets/installation/18_reboot_server.png)

## Server name

If you want, you can change the server's name using the following command:
```shell
hostnamectl set-hostname new-server-name
```

Replace **new-server-name** by the name you want. Example:
```shell
hostnamectl set-hostname poller1
```

## Update the system

Connect to your server using a terminal, and execute the command:
```shell
yum update
```

> Accept all GPG keys if you are prompted

Then restart your server with the following command:
```shell
reboot
```

After the machine has been rebooted, enable the services to allow centengine to be running on startup.
```shell
systemctl enable centreon centengine centreontrapd snmptrapd
```

Passive monitoring services can be started:
```shell
systemctl start centreontrapd snmptrapd
```

Restart Centreon Engine:
```shell
systemctl restart centengine
```
## Register the server

To register it to the Centreon Central server or a Remote server, execute the following command:

``` shell
/usr/share/centreon/bin/registerServerTopology.sh -u <API_ACCOUNT> \
-t poller -h <IP_TARGET_NODE> -n <POLLER_NAME>
```

Example:

``` shell
/usr/share/centreon/bin/registerServerTopology.sh -u admin -t poller -h 192.168.0.1 -n poller-1
```

> Replace **<IP_TARGET_NODE>** by the IP of the Centreon server seen by the poller or by the Remote Server if you
> want to link your server to it.

> The **<API_ACCOUNT>** must have access to configuration API. You can use default **admin** account.

> If you need to change the HTTP method or the port, you can use the following format for the **-h** option:
> HTTPS://<IP_TARGET_NODE>:PORT

Then follow instructions by
1. Entering your password:

    ``` shell
    192.168.0.1: please enter your password:
    ```

2. Select the IP adress if multiple network interfaces exist:

    ```shell
    Which IP do you want to use as CURRENT NODE IP ?
    1) 192.168.0.2
    2) 192.168.0.3
    1
    ```

3. Then validate the information:

    ``` shell
    Summary of the informations that will be send:
    
    Api Connection:
    username: admin
    password: ******
    target server: 192.168.0.1
    
    Pending Registration Server:
    name: poller-1
    type: poller
    address: 192.168.0.2
    
    Do you want to register this server with those informations ? (y/n)y
    ```
    
You will receive the validation of the Centreon central or the Remote Server server:

``` shell
2020-10-16T17:19:37+02:00 [INFO]: The CURRENT NODE 'poller': 'poller-1@192.168.0.2' linked to TARGET NODE: '192.168.0.1' has been added
```

### Main error messages

``` shell
2020-10-20T10:23:15+02:00 [ERROR]: Invalid credentials
```

> Your credentials are incorrect for the **<API_ACCOUNT>**.

``` shell
2020-10-20T10:24:59+02:00 [ERROR]: Access Denied.
```

> The **<API_ACCOUNT>** doesn't have access to configuration API.

``` shell
Failed connect to 192.168.0.1:444; Connection refused
```

> Unable to access to the API. Please check **<IP_TARGET_NODE>**, scheme and port.

``` shell
2020-10-20T10:39:30+02:00 [ERROR]: Can’t connect to the API using: https://192.168.0.1:443/centreon/api/latest/login
```

> The access url is not complete or invalide. Use the **--root** option to define the API URL Path. For example: **--root monitoring**.

``` shell
2020-10-20T10:42:23+02:00 [ERROR]: No route found for “POST /centreon/api/latest/platform/topology”
```

> Your Centreon target version is invalid. It should be greater or equal to 21.04.

## Add the Poller to configuration

Go to the [Add a Poller to configuration](../../monitoring/monitoring-servers/add-a-poller-to-configuration.md).

## Secure your platform

Don't forget to secure your Centreon platform following our
[recommendations](../../administration/secure-platform.md)
