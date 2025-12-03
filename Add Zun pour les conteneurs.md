<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    
</head>
<body>

<h1>Ajouter le composant Zun dans le controller</h1>

<h2>Step 1 : Activer Zun sur le all-in-one </h2>
<pre><code>
enable_zun: "yes"
enable_kuryr: "yes"
enable_etcd: "yes"
docker_configure_for_zun: "yes"
</code></pre>

<h2>Step 2 : Creer le dossier cni avec les permissions</h2>
<pre><code>
mkdir -p /opt/cni/bin
chmod 755 /opt/cni /opt/cni/bin
</code></pre>

<h2>Step 3 : Deployer Zun</h2>
<pre><code>
kolla-ansible reconfigure -i all-in-one 
</code></pre>

<div class="footer">
    <p>Publié par HAMDI Mohamed Amine</p>
    <p>Date : 02/12/2025</p>
</div>

</body>
</html>
