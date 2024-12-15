
# Olyn - An Automated DevOps Framework for Chef

Olyn is a bundle of custom Chef cookbooks to build and deploy functional clustered Debian web servers with globally replicated multi-write MySQL databases. All of this is done using infrastructure-as-code and configuration files.

## Features

- Configures a Percona cluster to securely replicate databases asynchronously to other nodes using their public IPs
- Builds a virtual web root from multiple Git repos
- Installs and configures openlitespeed web server with HTTPS
- Uses HAProxy to enable a reverse proxy that handles internal load balancing and SSL offloading
- Enables dynamic web page caching using Varnish and a custom warmer routine based on sitemap URLs
- Handles secrets and certificates securely using a data bag
- Sets up users, SSH keys, sudo, disables root login, and enforces security best practices
- Configures UFW to allow installed services and communications from other nodes in the cluster and blocks all other traffic
- Installs fail2ban to lock out intrusion attempts
- Sets up logwatch to email important alerts and summaries of log events
- Does your homework and the dishes

## Tech

Olyn uses a number of open source projects to work properly. You can mix and match packages as needed using Berkshelf to build your ideal IaC setup.

- [Debian Linux](https://www.debian.org/) - Reliable Linux distribution
- [Percona](https://www.percona.com/) - Deploys replicated MySQL database reliably on any IaaS provider
- [openlitespeed](https://openlitespeed.org/) - Open-source web server that boosts performance and supports Apache rewrites
- [HAProxy](https://www.haproxy.com/) - Load balancing, reverse proxy, and SSL offloading services
- [Fail2Ban](https://www.fail2ban.org/wiki/index.php/Main_Page) - Scans log files for malicious activity
- [Logwatch](https://ubuntu.com/server/docs/logwatch) - Log monitoring and alerts
- [openSSH](https://www.openssh.com/) - Secure SSH tunneling
- [UFW](https://help.ubuntu.com/community/UFW) - Software firewall
- [Varnish](https://varnish-cache.org/) - Highly configurable HTTP full page cache

And of course Olyn itself runs on [Chef](https://www.chef.io/).

## Cookbooks

Each cookbook has its own repo and individual instructions if needed.

| Cookbook | Description |
| -- | -- |
| [olyn_init](https://github.com/scottyrichardson/olyn_init) | Initializes and runs all installed Olyn cookbooks. |
| [olyn_apt](https://github.com/scottyrichardson/olyn_apt) | Configures APT. Sets up custom repos. |
| [olyn_database](https://github.com/scottyrichardson/olyn_database) | Initializes databases. Creates database users. Configures user permissions. Imports SQL files. |
| [olyn_fail2ban](https://github.com/scottyrichardson/olyn_fail2ban) | Installs Fail2Ban. Configures jails and ban durations. |
| [olyn_git](https://github.com/scottyrichardson/olyn_git) | Installs Git. Sets up Git users. Maps CI/CD repos to file directory locations. Builds virtual WWW roots. Configures folder and file permissions. Syncs and deploys new commits. |
| [olyn_haproxy](https://github.com/scottyrichardson/olyn_haproxy) | Installs HAProxy. Configures front-end and back-end listeners. Sets up SSL offloading. |
| [olyn_litespeed](https://github.com/scottyrichardson/olyn_litespeed) | Configures Openlitespeed apt repos. Installs Openlitespeed package. Configures VHOSTS, TLS certificates, PHP, admin portal, and security. |
| [olyn_logwatch](https://github.com/scottyrichardson/olyn_logwatch) | Installs LogWatch. Sets up log monitoring and email alerts. |
| [olyn_openssh](https://github.com/scottyrichardson/olyn_openssh) | Installs openSSH. Configures SSH ports and security. |
| [olyn_percona](https://github.com/scottyrichardson/olyn_percona) | Configures Percona apt repos. Installs Percona. Configures replication settings, TLS encryption, and node member list. Bootstraps and/or joins the replicated MySQL database cluster. |
| [olyn_postfix](https://github.com/scottyrichardson/olyn_postfix) | Installs Postfix. Configures relay host. |
| [olyn_sendmail](https://github.com/scottyrichardson/olyn_sendmail) | Uninstalls sendmail. |
| [olyn_sudo](https://github.com/scottyrichardson/olyn_sudo) | Installs sudo. Configures sudo group membership for users. |
| [olyn_system](https://github.com/scottyrichardson/olyn_system) | Configures Debian OS. Adds cluster nodes to hosts file. Installs base apt packages. Securely installs public and private keys from TLS certificates in data bag. Sets timezone. Creates users and configures permissions. |
| [olyn_ufw](https://github.com/scottyrichardson/olyn_ufw) | Installs UFW. Configures rules for ports, hosts, and protocols. Adds default deny rule. Reloads configuration. |
| [olyn_varnish](https://github.com/scottyrichardson/olyn_varnish) | Installs Varnish. Creates front-end and back-end listeners. Compiles VCL rules for content expiration. |
| [olyn_warmer](https://github.com/scottyrichardson/olyn_warmer) | Installs Nokogiri Ruby gem. Imports sitemaps URLs. Rewarms URLs from sitemaps in Varnish HTTP cache. |

## Local Dev Setup

#### Chef Secret Key File
Before deploying to a new environment a secret key file must be generated and saved at `[CHEF_ROOT]/provision/chef_configs/encrypted_data_bag_secret`.

To generate a new secret key file run the following in a Linux server:

    openssl rand -base64 4096 | tr -d '\r\n' > encrypted_data_bag_secret

#### Berks Install Script
This script calls Berks to install all cookbooks and their dependencies into `[CHEF_ROOT]/cookbooks` from specified sources.
Call this script from `[CHEF_ROOT]` during development.
A `Berksfile` needs to be present in `[CHEF_ROOT]` with all of the required cookbooks listed.
If a `Berksfile.lock` file already exists and the dependency versions are still valid, the existing cookbook version will be used.

To execute this script run the following in a Windows Terminal at `[CHEF_ROOT]`:

    .\cookbooks\olyn_init\scripts\dev\berks\install.bat

#### Berks Update Cookbooks Script
This script calls Berks to update all cookbooks and their dependencies into `[CHEF_ROOT]/cookbooks` from specified sources.
Call this script from `[CHEF_ROOT]` during development.
A `Berksfile` needs to be present in `[CHEF_ROOT]` with all of the required cookbooks listed.
Unlike the `install.bat` script, this will attempt to download the latest acceptable versions of all cookbooks and their dependencies.

To execute this script run the following in a Windows Terminal at `[CHEF_ROOT]`:

    .\cookbooks\olyn_init\scripts\dev\berks\update.bat

#### Data Bags Encryption Script
This script encrypts any raw data bags stored under `[CHEF_ROOT]/.unencrypted`.
Call it during development only.
Unencrypted databag contents should never hit a live server or a final git repo.
Encrypted databags are saved to `[CHEF_ROOT]/data_bags`  using the secret key installed to Chef.

To encrypt all data bags run the following in a Windows Terminal at `[CHEF_ROOT]`:

    .\cookbooks\olyn_init\scripts\dev\encrypt\data_bag.bat

To encrypt only specific data bags run the following in a Windows Terminal at `[CHEF_ROOT]`:

    .\cookbooks\olyn_init\scripts\dev\encrypt\data_bag.bat [folder_1] [folder_2]

## Initial Server Setup

#### Create The Chef Root Folder
As root, create the folder where the bootstrap Chef will reside:

    mkdir ~/chef

#### Upload Chef Files
Connect via SFTP and upload the root of Chef into `~/chef`.

#### Run the Bootstrap script
From the server execute the bootstrap script.
It will update and install all required system packages, install Chef itself, and copy the secret key file into place:

    sudo chmod +x ~/chef/cookbooks/olyn_init/scripts/bootstrap/boot.sh && sudo bash ~/chef/cookbooks/olyn_init/scripts/bootstrap/boot.sh && sudo chef-solo -c ~/chef/solo.rb -o "olyn_init"

After the Bootstrap script finishes, the first Chef run can be started:

    sudo chef-solo -c ~/chef/solo.rb

You can now run any of the standard Chef commands below. After the first run is complete, remove the uploaded Chef files:

    sudo rm ~/chef -R

## Chef Commands

#### Standard Chef Run
Runs the default runlist of Chef recipes as specified in the `[CHEF_ROOT]/node.json` file.

    sudo chef-solo -c ./solo.rb

#### Run A Single Chef Recipe
Overrides the default run list to run a single specified recipe.

    sudo chef-solo -c ./solo.rb -o "[RECIPE_NAME]"

## Written by

[Scott Richardson](https://github.com/scottyrichardson)

## License

MIT
