<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    
</head>
<body>

<h1>Ajouter Compute Nova avec Kolla-Ansible</h1>


<h2>Prérequis</h2>
<p>Avant de commencer, assurez-vous d'avoir :</p>
<ul>
    <li><strong>UBUNTU 24.04 Server LTS </strong> installé sur votre machine.</li>
    <li>Minimum system requirements for running OpenStack</li>
    <li>-- CPU : 2 (Recommended 4)</li>
    <li>-- RAM : 10GB (Recommended 16 GB)</li>
    <li>-- Disk space : 60 GB (Recommended 100 GB)</li>
    <li>-- 1 Network interface</li>
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


<pre><code>network:
    version: 2
    renderer: networkd
    ethernets:
        enp0s3:
            addresses:
              - 10.30.48.11/22
            nameservers:
              addresses: [8.8.8.8]
            routes:
              - to: default
                via:  10.30.51.254
        </code></pre>

<li>Apply new configuration and check IP address.</li>

<pre><code>netplan apply
ip addr</code></pre>

<h3>2.2 Add IP hosts</h3>

<pre><code>nano /etc/hosts
    10.30.48.10 controller
    10.30.48.11 compute
 </code></pre>

<h2>Step 3 : Additional dependencies to be installed on your VM</h2>

<pre>apt-get install -y python3-dev libffi-dev gcc libssl-dev<code>
</code></pre>

<pre>apt install apt-transport-https ca-certificates curl software-properties-common<code>
</code></pre>

<pre>curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg<code>
</code></pre>

<pre>sudo apt-get install -y docker.io python3-docker<code>
</code></pre>

<h2>Step 4 : Copy Public Key and security Level</h2>

<pre><code>echo "votre_clé_publique" >> ~/.ssh/authorized_keys
</code></pre>

<pre><code>chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh
</code></pre>

<h2>Step 5 : SSH Configuration</h2>
<li>Acces SSH Config </li>

<pre>sudo nano /etc/ssh/sshd_config<code>
</code></pre>

<pre><code>
PubkeyAuthentication yes      # Autoriser les clés SSH
PasswordAuthentication no     # Désactiver le mot de passe (optionnel)
PermitRootLogin yes           # Autoriser root (ou "prohibit-password" pour plus de sécurité)
</code></pre>


<h2>Step 6 : Controller Configuration </h2>

<li>SSH Configuration</li>

<pre><code>nano ~/.ssh/config
</code></pre>

<li>Add Compute Informations</li>

<pre><code>
    Host 10.30.48.11  # compute nova 
        User root
        IdentityFile /root/.ssh/key-openstack
</code></pre>

<li>All-in-one Configuration</li>

<pre><code>nano all-in-one
</code></pre>

<li>Add Line Below</li>
<pre><code>
[compute]
localhost         ansible_connection=local
compute           ansible_host=10.30.48.11   ansible_user=root ansible_ssh_private_key_file=/root/.ssh/key-openstack

</code></pre>


<h2>Step 7 : Deploy Kolla Ansible Openstack Compute Nova</h2>

<h3>7.1 Run bootstrap-servers using all-in-one inventory file:</h3>

<pre><code>kolla-ansible  bootstrap-servers -i ./all-in-one --limit compute
</code></pre>

<h3>7.2 Run prechecks using all-in-one inventory file:</h3>

<pre><code>kolla-ansible  prechecks -i ./all-in-one --limit compute
</code></pre>

<h3>7.3 Deploy kolla-ansible openstack:</h3>

<pre><code>kolla-ansible  deploy -i ./all-in-one --limit compute
</code></pre>



<div class="footer">
    <p>Publié par HAMDI Mohamed Amine</p>
    <p>Date : 11/11/2025</p>
</div>

</body>
</html>




