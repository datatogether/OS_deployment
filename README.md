# Kubernetes Cluster on Openstack (Kilo)

[![GitHub](https://img.shields.io/badge/project-Data_Together-487b57.svg?style=flat-square)](http://github.com/datatogether)
[![Slack](https://img.shields.io/badge/slack-Archivers-b44e88.svg?style=flat-square)](https://archivers-slack.herokuapp.com/)
[![License](https://img.shields.io/github/license/mashape/apistatus.svg?style=flat-square)](./LiCENSE)

This **Openstack Heat Template** creates up a Kubernetes cluster on Openstack for[ Data Together](https://datatogether.org) resources. A HEAT Orchestration Template are used for both provisioning and configuration, however, this can be modified, to allow for tools like Ainsible to perform the configuration and/or the provisioning as well. 

New features of the Data Together platform are coordinated and developed in one or more of the other repositories listed in our [Roadmap](https://github.com/datatogether/roadmap). 

## License & Copyright

Copyright (C) 2018 Data Together

This program is free software: you can redistribute it and/or modify it under
the terms of the GNU Affero General Public License as published by the Free Software
Foundation, version 3.0.

This program is distributed in the hope that it will be useful, but WITHOUT ANY
WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A
PARTICULAR PURPOSE.

See the [`LICENSE`](./LICENSE) file for details.

## Getting Involved

We would love involvement from more people! If you notice any errors or would like to submit changes, please see our [Contributing Guidelines](./.github/CONTRIBUTING.md).

We use GitHub issues for [tracking bugs and feature requests](https://github.com/datatogether/datatogether_deployment/issues) and Pull Requests (PRs) for [submitting changes](https://github.com/datatogether/datatogether_deployment/pulls).

## Usage
### 1. Create a Keypair
_Key pairs are ssh credentials which are injected into images when they are launched. Creating a new key pair registers the public key and downloads the private key (a .pem file). Protect and use the key as you would any normal ssh private key._

* Navigate to  Compute > Access & Security > Key Pairs > Create Key Pair

* Provide a name for the key and download and save the .pem file to the .ssh folder on your local machine. Change the permission of the file to 600. (`chmod 600 ____.pem`)


### 2. Get the parameters from your Openstack Project
* **Openstack Auth URL** : Obtain from Compute > Access & Security > API Access > Download Openstack RC
* **Tenant ID** : Obtain from Compute > Access & Security > API Access > Download Openstack RC
* **External Network** : See Routers tab for name of the External Network.

### 3. Deploy the Heat Orchestration Template
* Navigate to **Orchestration** > **Stacks** > **Launch Stack**

* Select URL as the Template Source, and provide the RAW URL of the heat template. Alternatively, you may copy/paste the contents or upload the file.

* Fill out **Launch Stack** form:
	- Use the **Openstack Auth URL**, **Tenant ID**, **External Network** and **Key Pair** from above.
	- You have the option to **Use Kubeadm** to bootstrap the deployment. If used, the latest version of Kubernetes is installed. If it is not used, the latest stable version of the CentOS repository Kubernetes (current at 1.5) is installed. 
	- You may provide an additional url to a file to run **Custom Scripts** on the master node if needed.
	- The selected **Image** should be `Centos 7 x64-2017-02`. [See Customisation section for details]
	- The **Flavor** selected should ideally at least 2 processors with 6 GB of RAM.
	- You can also choose the number of **Minions** and the **Minion Volume Size**. The Master node's volume is 8GB by default.
* See the **Outputs** in the Stack Overview for Master node log in instructions post-deployment. (Orchestration > Stakcs > (Select your stack) > Overview)

[See Screenshots](./screenshots)

## Heat Resources
Heat template Version: 2015-04-30

**Policies and IPs**
* `k8s_master_floating` - Creates a floating IP address for the master node
* `k8s_master_floating_association` - Attaches it to the master node
* `k8s_master_access` - Creates a security group for the master node with the necessary open ports

**Servers and Volumes**
* `k8s_master` - Creates the master node
* `k8s_master_vol` - Creates the master volume
* `k8s_master_vol_attach` - Attaches the volume to the master node
* `k8s_minion_group` - Creates a group of minions with volumes. 


## Customisation
If switching to a different image, change the references to `centos` to the default login for that image, and adjust `yum` to the default package installer. See the functions `os_inits`, `packages_install()` and `packages_install_ka()` which are image specific in both the master and minion templates, and `kubeadm_script` in the minion template.
