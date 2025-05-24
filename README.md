# Red Hat Satellite 6.15 Installation in a Connected Network Environment

## Preparing your environment for Installation

### System Requirements for base OS

- x86_64 architecture
- The latest version of Red Hat Enterprise Linux 8
- 4-core 2.0 GHz CPU at a minimum
- A minimum of 20 GB RAM is required for Satellite Server to function
  - In addition, a minimum of 4 GB RAM of swap space is also recommended
- A unique host name, which can contain lower-case letters, numbers, dots (.) and hyphens (-)
- A current Red Hat Satellite subscription
- Administrative user (root) access
- Full forward and reverse DNS resolution using a fully-qualified domain name

### Ensure that your environment meets the requirements for installation

- Satellite Server must be installed on a freshly provisioned system that serves no other function except to run Satellite Server.
- The freshly provisioned system must not have the following users provided by external identity providers to avoid conflicts with the local users that Satellite Server creates:
  - apache
  - foreman
  - foreman-proxy
  - postgres
  - pulp
  - puppet
  - redis
  - tomcat

#### SELinux Mode

SELinux must be enabled, either in enforcing or permissive mode.

> [!WARNING] 
> Installation with disabled SELinux is not supported.

#### Synchronized System Clock

The system clock on the base operating system where you are installing your Satellite Server must be synchronized across the network. 

> [!WARNING]
> If the system clock is not synchronized, SSL certificate verification might fail.

#### FIPS Mode

You can install Satellite on a Red Hat Enterprise Linux system that is operating in FIPS mode.

> [!WARNING]
> You cannot enable FIPS mode after the installation of Satellite.

> [!CAUTION]
> The FUTURE crypto-policy is not supported for Satellite and Capsule installations.

### Storage Requirements

The following table details storage requirements for specific directories. These values are based on expected use case scenarios and can vary according to individual environments.

The runtime size was measured with Red Hat Enterprise Linux 6, 7, and 8 repositories synchronized.

| Directory        | Installation Size | Runtime Size   |
| :-------------   | :-------------:   | :-----:        |
| /var/log         | 10 MB             | 10 GB          |
| /var/lib/pgsql   | 100 MB            | 20 GB          |
| /usr             | 10 GB             | Not Applicable |
| /opt/puppetlabs  | 500 MB            | Not Applicable |
| /var/lib/pulp    | 1 MB              | 300 GB         |

> [!NOTE]
> For external database servers: /var/lib/pgsql with installation size of 100 MB and runtime size of 20 GB.

### Storage Guidelines

- If you mount the /tmp directory as a separate file system, you must use the exec mount option in the /etc/fstab file. If /tmp is already mounted with the noexec option, you must change the option to exec and re-mount the file system. This is a requirement for the puppetserver service to work.
- Because most Satellite Server data is stored in the /var directory, mounting /var on LVM storage can help the system to scale.
- Use high-bandwidth, low-latency storage for the /var/lib/pulp/ directories. As Red Hat Satellite has many operations that are I/O intensive, using high latency, low-bandwidth storage causes performance degradation. Ensure your installation has a speed in the range 60 – 80 Megabytes per second.

### File System Guidelines

Do not use the GFS2 file system as the input-output latency is too high.

### Log file storage

Log files are written to /var/log/messages/, /var/log/httpd/, and /var/lib/foreman-proxy/openscap/content/. You can manage the size of these files using logrotate.

### Duplicated packages

Packages that are duplicated in different repositories are only stored once on the disk. Additional repositories containing duplicate packages require less additional storage.

### Symbolic links

You cannot use symbolic links for /var/lib/pulp/.

### Supported Operating Systems

Red Hat Enterprise Linux 8 - x86_64 only.

> [!NOTE]
> Previous versions of Red Hat Enterprise Linux including EUS or z-stream are not supported.

> [!WARNING]
> - Red Hat advises against using an existing system because the Satellite installer will affect the configuration of several components.  
> - Red Hat does not support using the system for anything other than running Satellite Server.

### Supported Browsers

Satellite supports recent versions of Firefox and Google Chrome browsers.

> [!NOTE]
> The Satellite web UI and command-line interface support English, Simplified Chinese, Japanese, French.

### Ports and Firewall Requirements

| Destination Port | Protocol           | Service         | Source                  | Required For                       | Description                                                            |
| :-------------:  | :-------------:    | :-------------: | :---------------------: | :--------------------------------: | :--------------------------------------------------------------------: | 
| 53               | TCP and UDP        | DNS             | DNS Servers and clients | Name resolution                    | DNS (optional)                                                         |
| 67               | UDP                | DHCP            | Client                  | Dynamic IP                         | DHCP (optional)                                                        |
| 69               | UDP                | TFTP            | Client                  | TFTP Server (optional)             |                                                                        |
| 443              | TCP                | HTTPS           | Capsule                 | Red Hat Satellite API              | Communication from Capsule                                             |
| 443, 80          | TCP                | HTTPS, HTTP     | Client                  | Global Registration                | Registering hosts to Satellite                                         |
| 443              | TCP                | HTTPS           | Red Hat Satellite       | Content Mirroring                  | Management                                                             |
| 443              | TCP                | HTTPS           | Red Hat Satellite       | Capsule API                        | Smart Proxy functionality                                              |
| 443, 80          | TCP                | HTTPS, HTTP     | Capsule                 | Content Retrieval                  | Content                                                                |
| 443, 80          | TCP                | HTTPS, HTTP     | Client                  | Content Retrieval                  | Content                                                                |
| 1883             | TCP                | MQTT            | Client                  | Pull based REX (optional)          | Content hosts for REX job notification (optional)                      |
| 5910 – 5930      | TCP                | HTTPS           | Browsers                | Compute Resource’s virtual console |                                                                        |
| 8000             | TCP                | HTTP            | Client                  | Provisioning templates             | Template retrieval for client installers, iPXE or UEFI HTTP Boot       |
| 8000             | TCP                | HTTPS           | Client                  | PXE Boot                           | Installation                                                           |
| 8140             | TCP                | HTTPS           | Client                  | Puppet agent                       | Client updates (optional)                                              |
| 9090             | TCP                | HTTPS           | Red Hat Satellite       | Capsule API                        | Smart Proxy functionality                                              |
| 9090             | TCP                | HTTPS           | Client                  | OpenSCAP                           | Configure Client (if the OpenSCAP plugin is installed)                 |
| 9090             | TCP                | HTTPS           | Discovered Node         | Discovery                          | Host discovery and provisioning (if the discovery plugin is installed) |

## Installing Red Hat Satellite Server 6.15

### Prerequisites

A Virtual Machine with the latest version of Red Hat Enterprise Linux 8 on it.

I have created a Virtual Machine with the following configuration:
- **vCPUs** - 4
- **RAM** - 20 GB
- **Storage** - 500 GB
- **Operating System** - Red Hat Enterprise Linux 8.10

### Steps to Install

Become root user
```console
su -
```

```none
[ragrawal@localhost ~]$ su -
Password: <enter_password>
```

Set hostname and reboot 
```
[root@localhost ~]# hostnamectl set-hostname satellite.example.com
[root@localhost ~]# reboot
```

Become root user
```
[ragrawal@localhost ~]$ su -
Password: <enter_password>
```

Verify hostname
```
[root@satellite ~]# hostname
```

> [!WARNING]
> Name resolution is critical to the operation of Satellite. If Satellite cannot properly resolve its fully qualified domain name, tasks such as content management, subscription management, and provisioning will fail.

Configure SELinux in Enforcing mode
```
[root@satellite ~]# entenforce 1
```

Verify SELinux mode
```
[root@satellite ~]# getenforce
```

Verify System Clock Synchronization
```
[root@satellite ~]# chronyc sources -v
```

Open the firewall ports for clients on Satellite Server
```
[root@satellite ~]# firewall-cmd \
--add-port="8000/tcp" \
--add-port="9090/tcp"
```

Allow access to services on Satellite Server
```
[root@satellite ~]# firewall-cmd \
--add-service=dns \
--add-service=dhcp \
--add-service=tftp \
--add-service=http \
--add-service=https \
--add-service=puppetmaster
```

Make the changes persistent in firewall
```
[root@satellite ~]# firewall-cmd --runtime-to-permanent
```

Verify firewall configuration
```
[root@satellite ~]# firewall-cmd --list-all
```

![firewall_output](/images/1-firewall_output.png)

Verify DNS resolution
```
[root@satellite ~]# ping -c1 localhost
[root@satellite ~]# ping -c1 `hostname -f`
```

![ping_output](/images/2-ping_output.png)

Register the host to Red Hat Subscription Management
```
[root@satellite ~]# subscription-manager register
```

![subscription_register](/images/3-subscription_register.png)

Disable all repos
```
[root@satellite ~]# subscription-manager repos --disable "*"
```

Enable the required repos
```
[root@satellite ~]# subscription-manager repos --enable=rhel-8-for-x86_64-baseos-rpms \
--enable=rhel-8-for-x86_64-appstream-rpms \
--enable=satellite-6.15-for-rhel-8-x86_64-rpms \
--enable=satellite-maintenance-6.15-for-rhel-8-x86_64-rpms
```

![enabling_repos](/images/4-enabling_repos.png)

Verify that repositories are enabled.
```
[root@satellite ~]# subscription-manager repos --list-enabled
```

![enabled_repos](/images/5-enabled_repos.png)

Enable the DNF module for satellite
```
[root@satellite ~]# dnf module enable satellite:el8 -y
```

Install, Enable and Verify fapolicyd on Satellite Server (Optional)
```
[root@satellite ~]# dnf install fapolicyd -y
[root@satellite ~]# systemctl enable --now fapolicyd
[root@satellite ~]# systemctl status fapolicyd
```

Update all packages
```
[root@satellite ~]# dnf upgrade -y
```

Reboot System (If required)
```
[root@satellite ~]# reboot
```

Install Satellite Server packages
```
[root@satellite ~]# dnf install satellite -y
```

Install Satelllite Server

```
# satellite-installer --scenario satellite \
--foreman-initial-organization "My_Organization" \
--foreman-initial-location "My_Location" \
--foreman-initial-admin-username admin_user_name \
--foreman-initial-admin-password admin_password
```

- **Organization**: redhat
- **Location**: redhat
- **Admin Username**: admin
- **Admin Password**: redhat

```
[root@satellite ~]# satellite-installer --scenario satellite \
--foreman-initial-organization "redhat" \
--foreman-initial-location "ontario" \
--foreman-initial-admin-username admin \
--foreman-initial-admin-password redhat
```

Once the installation is completed, similar output will be shown on the screen

![installation_output](/images/6-installation_output.png)  

### Login to Red Hat Satellite Server Dashboard

Open a browser, enter the Red Hat Satellite Server URL - https://satellite.example.com

![browser](/images/7-browser.png)

Enter Credentials and click Log In button

- **Username**: admin
- **Password**: redhat 

![credentials](/images/8-credentials.png)

You can now see the dashboard of Red Hat Satellite Server 6.15

![Dashboard](/images/9-dashboard.png)

## References
[Installing Satellite Server in a connected network environment](https://docs.redhat.com/en/documentation/red_hat_satellite/6.15/html-single/installing_satellite_server_in_a_connected_network_environment/index#providing-feedback-on-red-hat-documentation_satellite)  
[Red Hat Satellite 6.x Release info](https://access.redhat.com/articles/1365633#sat6)  
[Red Hat Satellite Product Life Cycle](https://access.redhat.com/support/policy/updates/satellite)
