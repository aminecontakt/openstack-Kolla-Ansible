<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    
</head>
<body>

<h1>Déployer OpenStack All-in-One avec Kolla-Ansible</h1>

<h2>Introduction</h2>
<p>Dans cet article, nous allons explorer comment déployer une instance d'OpenStack All-in-One en utilisant Kolla-Ansible. Ce guide est destiné aux développeurs et aux administrateurs système qui souhaitent expérimenter avec OpenStack dans un environnement local.</p>

<h2>Prérequis</h2>
<p>Avant de commencer, assurez-vous d'avoir :</p>
<ul>
    <li><strong>UBUNTU 24.04 Server LTS </strong> installé sur votre machine.</li>
    <li>Minimum system requirements for running OpenStack</li>
    <li>-- CPU : 2 (Recommended 4)</li>
    <li>-- RAM : 10GB (Recommended 16 GB)</li>
    <li>-- Disk space : 60 GB (Recommended 100 GB)</li>
    <li>-- 2 Network interface</li>
    <li><strong>Client SSH</strong> pour faciliter la gestion de la machine virtuelle.</li>
    <li><strong>Une connexion Internet </strong> pour télécharger les images et les dépendances nécessaires.</li>
</ul>

<h2>Step 1 : Configuration Environment</h2>

<h3>1.1 Install Ubuntu and update system</h3>
<pre><code>sudo su
 apt update && apt -y upgrade</code></pre>

<h2>Step 2 : Install Dependencies on the VM</h2>
<h3>2.1 Configure static IP in netplan configuration file located at /etc/netplan/ and deploy the new configuration:</h3>
<li>Use nano for editing 50-cloud-init.yaml file.</li>

<pre><code>nano /etc/netplan/50-cloud-init.yaml</code></pre>

<li>Modify the file like below. Replace the IP with your actual IP.</li>

<li>enp0s3 : Internal Network</li>
<li>enp0s8 : External Network</li>

<pre><code>network:
    version: 2
    renderer: networkd
    ethernets:
        enp0s3:
            addresses:
              - 10.30.48.10/22
            nameservers:
              addresses: [8.8.8.8]
            routes:
              - to: default
                via:  10.30.51.254
        enp0s8:
            dhcp4: true</code></pre>

<li>Apply new configuration and check IP address.</li>

<pre><code>netplan apply
ip addr</code></pre>

<h3>2.2 Update and upgrade the repo. Remove unused software packages:</h3>

<pre><code>apt update && sudo apt -y upgrade
apt autoremove</code></pre>

<h3>2.3 Add docker repository and Install:</h3>

<li>Docker requires some additional dependencies to be installed on your system.</li>

<pre><code>apt install apt-transport-https ca-certificates curl software-properties-common
</code></pre>


<li>Add Docker’s Official GPG Key</li>

<pre><code>curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
</code></pre>

<li>Add Docker’s official APT repository</li>

<pre><code>echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
</code></pre>

<li>Update the Package Index Again</li>

<pre><code>apt update
</code></pre>

<li>Install Docker Engine and Docker CLI</li>

<pre><code>apt install docker-ce docker-ce-cli containerd.io
</code></pre>

<li>Start, enable and check docker</li>

<pre><code>systemctl start docker
systemctl enable docker
docker --version
</code></pre>


<h3>2.4 Install system dependencies for Kolla-Ansible openstack:</h3>

<pre><code>apt-get install -y git python3-dev libffi-dev gcc libssl-dev pkg-config libdbus-1-dev build-essential cmake libglib2.0-dev mariadb-server
</code></pre>


<h3>2.5 Python3 virtual environment will be used to install Kolla Ansible. Install virtual environment:</h3>

<pre><code>apt install -y python3-venv
</code></pre>


<h2>Step 3 : Create Directories, Enable Python3 Environment and Install Required Python Dependencies</h2>

<h3>3.1 Make a directory and enable python3 virtual environment on that directory:</h3>

<li>Make a directory using below command</li>

<pre><code>mkdir openstack
</code></pre>

<li>Switch to the directory</li>

<pre><code>cd openstack
</code></pre>

<li>Enable python3 virtual environment into current directory</li>

<pre><code>python3 -m venv .
</code></pre>

<li>Activate python3 virtual environment</li>

<pre><code>source bin/activate
</code></pre>

<h3>3.2 Upgrade pip and install setuptools docker and dbus-python:</h3>

<li>Run below command to upgrade pip</li>

<pre><code>pip install --upgrade pip
</code></pre>

<li>Install setuptools, docker and dbus-python</li>

<pre><code>pip install setuptools docker dbus-python
</code></pre>

<h3>3.3 Install targeted ansible version:</h3>

<pre><code>pip install 'ansible-core>=2.16,<2.17.99'
</code></pre>

<h3>3.4 Use pip to git clone kolla-ansible master repository:</h3>

<pre><code>pip install git+https://opendev.org/openstack/kolla-ansible@master
</code></pre>

<h3>3.5 Make a directory /etc/kolla and change the ownership to current user.</h3>

<pre><code>mkdir -p /etc/kolla
chown $USER:$USER /etc/kolla
</code></pre>

<h3>3.6 Copy all-in-one file to openstack directory:</h3>

<pre><code>cp share/kolla-ansible/ansible/inventory/all-in-one .
</code></pre>

<h3>3.7 Copy configuration file globals.yml & password.yml to /etc/kolla</h3>

<pre><code>cp -r  share/kolla-ansible/etc_examples/kolla/* /etc/kolla
</code></pre>


<h2>Step 4 : Kolla Ansible Dependencies Installation and Configuration File Modification.</h2>

<h3>4.1 Install kolla-ansible dependencies:</h3>

<pre><code>kolla-ansible install-deps
</code></pre>

<h3>4.2 Generate necessary password for openstack communication among different component:</h3>

<pre><code>kolla-genpwd
</code></pre>

<h3>4.3 Modify globals.yml file as below:</h3>

<li>Access glolobals.yml file</li>

<pre><code>nano /etc/kolla/globals.yml
</code></pre>

<li>Enable and modify below parameters</li>

<pre><code>kolla_base_distro: "ubuntu"
kolla_internal_vip_address: "< unused IP from network_interface IP range>"
network_interface: "enp0s3"
neutron_external_interface: "enp0s8"
nova_compute_virt_type: "qemu"
</code></pre>


<h2>Step 5 : Deploy Kolla Ansible Openstack</h2>

<h3>5.1 Run bootstrap-servers using all-in-one inventory file:</h3>

<pre><code>kolla-ansible  bootstrap-servers -i ./all-in-one
</code></pre>

<h3>5.2 Run prechecks using all-in-one inventory file:</h3>

<pre><code>kolla-ansible  prechecks -i ./all-in-one
</code></pre>

<h3>5.3 Deploy kolla-ansible openstack. it will take sometime (approximately 60 min):</h3>

<pre><code>kolla-ansible  deploy -i ./all-in-one
</code></pre>

<h3>5.4 OpenStack requires a clouds.yaml file where credentials for the admin user are set. To generate this file: run below command</h3>

<pre><code>kolla-ansible post-deploy -i ./all-in-one
</code></pre>


<h2>Step 6 : Install Openstack Client CLI and Create Demo Networks, Images, Flavors</h2>

<h3>6.1 Install the OpenStack CLI client:</h3>

<pre><code>pip install python-openstackclient -c https://releases.openstack.org/constraints/upper/master
</code></pre>

<h3>6.2 Depending on how you installed Kolla Ansible, there is a script that will create example networks, images, and so on :</h3>

<pre><code>./share/kolla-ansible/init-runonce
</code></pre>

<h2>Step 7 : Accessing Openstack Dashboard</h2>

<h3>7.1 Use browser and type configured IP for kolla_internal_vip_address to access dashboard:</h3>

<h3>7.2 Access /etc/kolla/clouds.yaml file to get admin user password:</h3>

<pre><code>cat /etc/kolla/clouds.yaml
</code></pre>

<h2>Step 8 : Generate SSH Key </h2>

<h3>8.1 Generate SSH Key for adding other componant node without Passphrase </h3>

<pre>ssh-keygen -t rsa -b 4096 -f ~/.ssh/key-openstack -C "key openstack" <code>
</code></pre>

<h3>8.2 Saving Public Key </h3>

<pre> cat  ~/.ssh/key-openstack.pub<code>
</code></pre>

<h2>Troubleshooting:</h2>

<h2>Installation failed somehow</h2>

<li>Run below command to destroy kolla ansible deployment</li>

<pre><code>kolla-ansible destroy --yes-i-really-really-mean-it -i ./all-in-one
</code></pre>

<h2>Redeploy Kolla Ansible Openstack</h2>

<h3>1. Follow Step 5 for Kolla Ansible Openstack.</h3>

<h3>2. Follow Step 6 for enabling openstack CLI and create demo networks, image.</h3>


<h2>Conclusion</h2>
<p>Vous avez maintenant un environnement OpenStack All-in-One fonctionnel grâce à Kolla-Ansible. Vous pouvez commencer à explorer les fonctionnalités d'OpenStack et à développer des applications cloud.</p>

<h2>Ressources supplémentaires</h2>
<ul>
    <li><a href="https://docs.openstack.org/">Documentation officielle d'OpenStack</a></li>
    <li><a href="https://github.com/openstack/kolla-ansible">Kolla-Ansible GitHub</a></li>
</ul>

<div class="footer">
    <p>Publié par HAMDI Mohamed Amine</p>
    <p>Date : 18/05/2025</p>
</div>

</body>
</html>




