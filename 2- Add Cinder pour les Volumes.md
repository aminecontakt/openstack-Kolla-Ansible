<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    
</head>
<body>

<h1>Ajouter le composant Cinder dans le controller</h1>

<h2>Step 1 : Désactiver IPv6 </h2>
<pre><code>
sysctl -w net.ipv6.conf.all.disable_ipv6=1
sysctl -w net.ipv6.conf.default.disable_ipv6=1
</code></pre>

<li>Vérification</li>

<pre><code>
cat /proc/sys/net/ipv6/conf/all/disable_ipv6
# doit afficher 1
</code></pre>

<li>Tester le Ping</li>
<pre><code>
ping www.google.com
ping 8.8.8.8
</code></pre>

<li>Puis appliquer</li>

<pre><code>
sysctl -p
</code></pre>

<h2>Step 2 : Configurer Docker pour n’utiliser qu’IPv4 </h2>

<li>Editer /etc/docker/daemon.json :</li>

<pre><code>
nano /etc/docker/daemon.json
</code></pre>

<pre><code>
{
  "bridge": "none",
  "ip-forward": false,
  "iptables": false,
  "log-opts": {
    "max-file": "5",
    "max-size": "50m"
  },
  "ipv6": false
}

</code></pre>

<li>Redemarrer Docker ou le controller</li>

<pre><code>
systemctl restart docker
rebbot
</code></pre>

<h2>Step 3 : Configurer le stockage </h2>

<li>Ajouter un disque</li>
<li>Configuration du volume</li>

<pre><code>
pvcreate /dev/sdb
vgcreate cinder-volumes /dev/sdb
</code></pre>

<h2>Step 4 : Activer Cinder dans la configuration Kolla </h2>

<li>Configuration du volume</li>

<pre><code>
/etc/kolla/globals.yml
</code></pre>
<pre><code>
enable_cinder: "yes"

# Backend LVM
enable_cinder_backend_lvm: "yes"
# Si tu n’utilises que LVM (pas de Ceph, NFS, etc.) vérifie que les autres backends sont à "no".
# enable_cinder_backend_nfs: "no"
# enable_cinder_backend_iscsi: "no"   # pour backend externe, pas pour LVM interne
</code></pre>

<li>Vérifier que dans ton inventory (all-in-one ou autre), le nœud contrôleur qui a /dev/sdb est bien dans le groupe [cinder-volume:children] (sinon ajoute-le) </li>
<pre><code>
nano all-in-one
</code></pre>
<pre><code>
[storage]
controller-node-name ansible_host=IP ...
#cinder
[cinder-volume:children]
storage
</code></pre>

<h2>Step 5 : Déployer / reconfigurer Cinder</h2>
<pre><code>
kolla-ansible reconfigure -i all-in-one
</code></pre>

<li>Tester l'état des conteneurs </li>
<pre><code>
docker ps
</code></pre>

<div class="footer">
    <p>Publié par HAMDI Mohamed Amine</p>
    <p>Date : 02/12/2025</p>
</div>

</body>
</html>
