# Developer Station - WSL Ubuntu

# Setup the PowerShell Environment
Update/Install  the AzureRM and pre-requisiste PS1 Modules. Open up a PS Window as Administrator and run:
```
Set-PSRepository -Name PSGallery -InstallationPolicy Trusted
install-module PackageManagement -verbose -force
install-module PowerShellGet -verbose -force
install-module pester -verbose -force –SkipPublisherCheck
install-module azurerm -verbose -AllowClobber
install-module azuread -verbose –AllowClobber
```
Once completed close the PS Window.

# Windows Subsystem for Linux (WSL)

Windows Subsystem for Linux (WSL) is a compatibility layer for running Linux binary executables natively on Windows 10. (Source: [Wikipedia](https://en.wikipedia.org/wiki/Windows_Subsystem_for_Linux))

## Uninstall WSL if already present (optional)
With a **_privileged user_** run on a command window:

```
lxrun /uninstall /full /y
```

## Install WSL

First, you need to upgrade your Windows 10 to the lastest version (take 3 hours aprox). Download the Windows Update Assistant [here](https://www.microsoft.com/en-us/software-download/windows10) and execute it. When this entry was written the lastest Windows 10's build was 1809.

Second, go to *Settings* --> *Apps* --> *Programs and Features* --> *Turn Windows features on or off* and tick the feature "*Windows Subsystem for Linux*".

Third, go to Microsoft Store and install *Ubuntu 16.04 LTS* or *Ubuntu 18.04 LTS* accord to your needs.

## Specially if you are behind of proxies
## Set the Proxy Server for Shell & APT Repositories  
Edit the following files. Make sure you replace with your LDAP credentials.
```
you may create a config file containing proxy settings as follows:

## Set your new root password in Ubuntu
sudo passwd

## Login as root
sudo -su

# nano /etc/apt/apt.conf.d/99proxy.conf

Acquire::http::Proxy "http://domain\samaccount:yourpassword@proxyip:8080/";
Acquire::https::Proxy "http://domain\samaccount:yourpassword@proxyip:8443/";
Acquire::ftp::Proxy "ftp://domain\samaccount:yourpassword@proxyip:8080/";


# nano /etc/apt/apt.conf

Acquire::http::Proxy "http://domain\samaccount:yourpassword@proxyip:8080/";
Acquire::https::Proxy "http://domain\samaccount:yourpassword@proxyip:8443/";
Acquire::ftp::Proxy "ftp://domain\samaccount:yourpassword@proxyip:8080/";


# nano /etc/apt/apt.conf.d/95proxies

Acquire::http::Proxy "http://domain\samaccount:yourpassword@proxyip:8080/";
Acquire::https::Proxy "http://domain\samaccount:yourpassword@proxyip:8443/";
Acquire::ftp::Proxy "ftp://domain\samaccount:yourpassword@proxyip:8080/";


# nano ~/.bash.rc
Acquire::http::Proxy "http://domain\samaccount:yourpassword@proxyip:8080/";
Acquire::https::Proxy "http://domain\samaccount:yourpassword@proxyip:8443/";
Acquire::ftp::Proxy "ftp://domain\samaccount:yourpassword@proxyip:8080/";


# nano /etc/environment
http_proxy=http://proxyip:8080/
https_proxy=http://proxyip:8080/
ftp_proxy=http://proxyip:8080/
no_proxy="localhost,127.0.0.1,youripaddress,.local.domain"
HTTP_PROXY=http://proxyip:8080/
HTTPS_PROXY=http://proxyip:8080/
FTP_PROXY=http://proxyip:8080/
NO_PROXY="localhost,127.0.0.1,youripaddress,.local.domain"

# nano /etc/wgetrc (Search Proxy Section)
https_proxy  = http://domain\samaccount:yourpassword@proxyip:8443
http_proxy = http://domain\samaccount:yourpassword@proxyip:8080
ftp_proxy = http://domain\samaccount:yourpassword@proxyip:8080

Uncomment 'use_proxy = on'




```

Some valid proxies are:
```
domain (ip)
domain (ip)
```


## Update the WSL and set your timezone
Get your WSL to the latest and greatest within the version installed. Run as root:
```
apt-get update -y
apt-get upgrade -y
dpkg-reconfigure tzdata

```

## Install Ansible

There are 2 ways of installing Ansible. The preferred way is to install it from the OS repositories but this may end up unstalling a very old version (depends on multiple things on what version you get). If the Ansiuble version is to old then you can try installing it via Python's Package Manager (PIP).

Ansible Tower version is 3.2.3 and it runs Ansible version 2.4.2.

### Ansible version 2.4.2 (from OS repository)

To install the Ansible v2.4.2 run

```
echo 'deb http://ppa.launchpad.net/ansible/ansible/ubuntu zesty main' | \
     sudo tee /etc/apt/sources.list.d/ansible.list
wget -qO- 'https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x93C4A3FD7BB9C367' | \
     sudo apt-key add -
sudo su -c 'apt-get update && apt-get install -y ansible==2.4.2.0-1ppa~zesty && apt-mark hold ansible'
```

Keep in mind that package *ansible* is marked on hold to prevent future upgrades.

### Install Ansible  + PIP 2 & 3 version by running as root:

```
apt install python-pip -y
apt install python3-pip -y
apt install ansible -y


#ansible --version *2.4.2 for Ubuntu 16.04 LTS*
ansible 2.4.2.0
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/home/pr1v8/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.12 (default, Dec  4 2017, 14:50:18) [GCC 5.4.0 20160609]


root@x:~# ansible --version  *2.5.1 for Ubuntu 18.04 LTS*
ansible 2.5.1
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/home/pr1v8/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.15rc1 (default, Apr 15 2018, 21:51:34) [GCC 7.3.0]
root@x:~#
```

### Set-up the WLS Ansible environment
#### Ansible Global Variables
On the file **_/etc/ansible/group_vars/all_**, set the variables as shown below. **It’s important you don’t include the initial “?”/question mark character provided by Azure on the SAS Tokens variables. It's also important to preserve the identation as shown below.**

```
---
ansible_env:
  IPAMURL: "url"
  SOFTWAREREPOURL: "https://<STORAGE_ACCOUNT_1_NAME>.blob.core.windows.net"
  SOFTWARESASTOKEN: "<SAS-Token 1>"
  TEMPLATEREPOURL: "https://<STORAGE_ACCOUNT_2_NAME>.blob.core.windows.net"
  TEMPLATESASTOKEN: "<SAS-Token 2>”
  ENVIROMENT: "local"
  DSCAUTOMATIONACCOUNT: "name"
  DCSAUTOMATIONKEY: "key"
  DSCAUTOMATIONURL: "url"
  MYWORKSPACEKEY: "key"
  MYWORSKPACEID: "id"
  OBJIDGROUPFOUNDATIONL3: "id"
  OBJIDGROUPENGINEERING: "id"
  OBJIDSPNANSIBLE: "id"
  AZURE_RM_SUB_australiaeast: "id"
  AZURE_RM_SUB_canadacentral: "id"
  AZURE_RM_SUB_canadaeast: "id"
  AZURE_RM_SUB_centralindia: "id"
  AZURE_RM_SUB_koreasouth: "id"
  AZURE_RM_SUB_koreasouth_mgmt: "id"
  AZURE_RM_SUB_southeastasia: "id"
  AZURE_RM_SUB_southindia: "id"
  AZURE_RM_SUB_uksouth: "id"
  AZURE_RM_SUB_uksouth_mgmt: "id"
  AZURE_RM_SUB_ukwest: "id"
  AZURE_RM_SUB_westcentralus: "id"
  AZURE_RM_SUB_westindia: "id"
  AZURE_RM_SUB_westus: "id"
  AZURE_RM_SUB_westus2: "id"
  AZURE_RM_SUB_westus2_mgmt: "id"
  AZURE_RM_SUB_eastus: "id"
AZURE_RM_CLIENTID: "SPN_ID"
AZURE_RM_SECRET: "SPN_SECRET"
AZURE_RM_TENANTID: "id"
```
Make sure you replace any text bwtween **< >**a with your personal information.

#### Ansible Config file
On the file **_/etc/ansible/ansible.cfg_**, set the variables as shown below:
```
[defaults]
hash_behaviour = merge
```

Either add it to the file / default section, add the section if not present or even create the file with the content shown as above.

### Install and Set your default Azure CLI in *Ubuntu 16.04 LTS*
As root run:
```
AZ_REPO=$(lsb_release -cs)
echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $AZ_REPO main" | tee /etc/apt/sources.list.d/azure-cli.list
curl -L https://packages.microsoft.com/keys/microsoft.asc | apt-key add -
apt-get install apt-transport-https -y
apt-get update && apt-get install azure-cli -y






### Install and Set your default Azure CLI + VS Code + .NET Core SDK 2.1 in *Ubuntu 18.04 LTS*
As root run:

rm -r /etc/resolv.conf
sudo systemctl disable systemd-resolved.service
sudo systemctl enable systemd-resolved.service

close Ubuntu

open again Ubuntu

sudo apt-get install dirmngr apt-transport-https

curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
sudo mv microsoft.gpg /etc/apt/trusted.gpg.d/microsoft.gpg
sudo sh -c 'echo "deb [arch=amd64] https://packages.microsoft.com/repos/vscode stable main" > /etc/apt/sources.list.d/vscode.list'

sudo apt-get update -y
sudo apt-get install code -y
sudo sh -c 'echo "deb [arch=amd64] https://packages.microsoft.com/repos/microsoft-ubuntu-bionic-prod bionic main" > /etc/apt/sources.list.d/dotnetdev.list'
sudo apt-get update -y
sudo apt-get install dotnet-sdk-2.1 -y

AZ_REPO=$(lsb_release -cs)
echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $AZ_REPO main" | \
    sudo tee /etc/apt/sources.list.d/azure-cli.list
curl -L https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add -
sudo apt-get update -y
sudo apt-get install azure-cli -y
sudo apt-get update && sudo apt-get upgrade

## Common az cli commands

» For aunthenticate in Azure cli
az login

### Example for Set an subscription as default 

#az account list


#az account set --subscription "id"

» For see the guide for use the azure cli
#az --help

» For start an interactive session in azure cli, like Cloud Shell with autocomplete feature, using TAB key 
#az interactive 

```



### Install PowerShell Core on WSL
As root run:
```
curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add -
curl -o /etc/apt/sources.list.d/microsoft.list https://packages.microsoft.com/config/ubuntu/16.04/prod.list
apt-get update && apt-get install -y powershell
```

Use Power shell as a regular user by running the following command:
```
pwsh
```

### Install a Desktop graphical UI (User Interface) and RDP you need to be sure to open port 3389
As root run:
```
sudo apt-get install lxde xrdp -y

#Initialize xrdp
/etc/init.d/xrdp start
apt autoremove -y

```

You may want to also install the AzureRM's modules. Within a PowerShell prompt as root run:

```
Install-Module -Name AzureRM.Netcore
Import-Module -Name AzureRM.Netcore 
```
