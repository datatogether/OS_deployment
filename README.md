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


### 2. Get the Parameters from your Openstack Project
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

[See Screenshots](./screenshots)

### 4. Accessing the Kubernetes Cluster Remotely
* You will know the system is ready when the **k8s_master log** shows `"**** COMPLETE ****"`.
* Log in using this command: `ssh -i .ssh/<ssh_key_name>.pem centos@<k8s_masterip>`. **Outputs** in the Stack Overview for Master node log in instructions post-deployment. (Orchestration > Stakcs > (Select your stack) > Overview)

**Checks & Troubloshooting:**

| Test| Expected Outcome | Troubleshoot |
| -------- | -------- | -------- |
| `kubectl get nodes --all-namespaces=true` | Should list all nodes in Ready state.|If all nodes are visible and in Not Ready, check whether `.kube/config` has the same contents as `/etc/kubernetes/admin.conf`, and whether the networking addon (weave or flannel) is properly set up. 
||| If one or more nodes are missing, check the logs of the missing nodes through the Openstack dashboard.|
| `systemctl is-system-running` | `running` | run `systemctl --state=failed` to determine failed services|
|||
|-| **Issues** | **Troubleshoot** |
|-| vm set up stalled | Delete stack. Try rebooting the Network (Network > Networks > Edit Network. Set Admin State as DOWN. Refresh and set again as UP.) |


### 5. Optional Installation Instructions

## Install Helm 
```
curl -s https://raw.githubusercontent.com/helm/helm/master/scripts/get > get_helm.sh
chmod 700 get_helm.sh
./get_helm.sh
kubectl create serviceaccount --namespace kube-system tiller
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'      
helm init --service-account tiller --upgrade
helm repo update  
```

## Install GlusterFS 
```
yum -y -q install centos-release-gluster41.x86_64 glusterfs gluster-cli glusterfs-libs glusterfs-server
systemctl enable glusterd.service
systemctl start glusterd.service 
```

## Related Links
* [Install IPFS](https://github.com/helm/charts/tree/master/stable/ipfs)


### Documentation

## Heat Orchestration Template Resources
Heat template Version: 2015-04-30

**Policies and IPs**
* `k8s_master_floating` - Creates a floating IP address for the master node
* `k8s_master_floating_association` - Attaches it to the master node
* `k8s_master_access` - Creates a security group for the master node with the necessary open ports

**Servers and Volumes**
* `k8s_master` - Creates the master node
* `k8s_master_vol` - Creates the master volume
* `k8s_master_vol_attach` - Attaches the volume to the master node
* `k8s_minion_group` - Creates a group of minions with volumes
	* `k8s_minion` - Creates a minion node
	* `k8s_mimnion_vol` - Creates the minion node volumes
	* `k8s_minion_vol_attach` - Attaches the volume to the minion node

**Outputs**
* `ssh_key_name` - key pair used for remote access
* `k8s_masterip` - External IP of the master node
* `login_instructions` - Guideline on how to log into the master node

## Customisation
If switching to a different image, change the references to `centos` to the default login for that image, and adjust `yum` to the default package installer. See the functions `os_inits`, `packages_install()` and `packages_install_ka()` which are image specific in both the master and minion templates, and `kubeadm_script` in the minion template.
