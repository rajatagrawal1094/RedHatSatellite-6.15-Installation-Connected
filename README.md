# Red Hat Satellite 6.15 Installation in a Connected Network Environment

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

- SELinux must be enabled, either in enforcing or permissive mode.

> [!WARNING] 
> Installation with disabled SELinux is not supported.

#### Synchronized System Clock

- The system clock on the base operating system where you are installing your Satellite Server must be synchronized across the network. 

> [!WARNING]
> If the system clock is not synchronized, SSL certificate verification might fail.

#### FIPS Mode

- You can install Satellite on a Red Hat Enterprise Linux system that is operating in FIPS mode.

> [!WARNING]
> You cannot enable FIPS mode after the installation of Satellite.

> [!CAUTION]
> The FUTURE crypto-policy is not supported for Satellite and Capsule installations.

### Storage Requirements

The following table details storage requirements for specific directories. These values are based on expected use case scenarios and can vary according to individual environments.

The runtime size was measured with Red Hat Enterprise Linux 6, 7, and 8 repositories synchronized.

| Directory        | Installation Size | Runtime Size   |
| :-------------:  | :-------------:   | :-----:        |
| /var/log         | 10 MB             | 10 GB          |
| /var/lib/pgsql   | 100 MB            | 20 GB          |
| /usr             | 10 GB             | Not Applicable |
| /opt/puppetlabs  | 500 MB            | Not Applicable |
| /var/lib/pulp    | 1 MB              | 300 GB         |

For external database servers: /var/lib/pgsql with installation size of 100 MB and runtime size of 20 GB.

### Storage Guidelines

- If you mount the /tmp directory as a separate file system, you must use the exec mount option in the /etc/fstab file. If /tmp is already mounted with the noexec option, you must change the option to exec and re-mount the file system. This is a requirement for the puppetserver service to work.
- Because most Satellite Server data is stored in the /var directory, mounting /var on LVM storage can help the system to scale.
- Use high-bandwidth, low-latency storage for the /var/lib/pulp/ directories. As Red Hat Satellite has many operations that are I/O intensive, using high latency, low-bandwidth storage causes performance degradation. Ensure your installation has a speed in the range 60 – 80 Megabytes per second.

### File System Guidelines

- Do not use the GFS2 file system as the input-output latency is too high.

### Log file storage

- Log files are written to /var/log/messages/, /var/log/httpd/, and /var/lib/foreman-proxy/openscap/content/. You can manage the size of these files using logrotate.

### Duplicated packages

- Packages that are duplicated in different repositories are only stored once on the disk. Additional repositories containing duplicate packages require less additional storage.

### Symbolic links

- You cannot use symbolic links for /var/lib/pulp/.

### Supported Operating Systems

- Red Hat Enterprise Linux 8 - x86_64 only.

> [!NOTE]
> Previous versions of Red Hat Enterprise Linux including EUS or z-stream are not supported.

> [!WARNING]  
> Red Hat advises against using an existing system because the Satellite installer will affect the configuration of several components.
> Red Hat does not support using the system for anything other than running Satellite Server.

### Supported Browsers

Satellite supports recent versions of Firefox and Google Chrome browsers.

The Satellite web UI and command-line interface support English, Simplified Chinese, Japanese, French.

### Ports and Firewall Requirements


