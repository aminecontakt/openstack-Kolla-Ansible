<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    
</head>
<body>

<h1>Ajouter le composant Cinder dans le controller</h1>

<h2>Step 1 : activer les drivers spécifiques au domaine </h2>
<li>Dans /etc/kolla/keystone/keystone.conf , activer les drivers spécifiques au domaine</li>

<pre><code>
[identity]
domain_specific_drivers_enabled = True
domain_configurations_from_database = True
</code></pre>

<li>Redémarrer le conteneur Keystone après modification</li>

<pre><code>
docker restart keystone
</code></pre>

<li>Dans /etc/kolla/horizon/_9999-custom-settings.py , activer l’option multi-domaine pour l’interface Horizon</li>

<pre><code>
OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True
OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = 'Default'
</code></pre>

<li>Redémarrer Horizon</li>

<pre><code>
docker restart horizon
</code></pre>

<li>Attendre le demarrage du conteneur Horizon</li>
<pre><code>
docker ps
</code></pre>


<h2>Step 2 : Gestion des domaines, Projets, utilisateurs et Roles </h2>



<pre><code>
openstack domain create --description "Description du domaine" domain-demo

openstack project create --domain domain-demo projet-demo

openstack user create --domain domain-demo --password Password123 utilisateur-demo

openstack role add --user utilisateur-demo --project projet-demo admin

</code></pre>


<div class="footer">
    <p>Publié par HAMDI Mohamed Amine</p>
    <p>Date : 11/11/2025</p>
</div>

</body>
</html>
