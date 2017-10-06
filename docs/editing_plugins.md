Pour Lodel 0.9+ : implémentation d'un système de plugin

== Droits ==


Seuls les administreurs Lodel peuvent installer/désinstaller un plugin via la partie administration Lodel. Il est également possible de modifier la configuration par défaut du plugin qui sera effective par défaut à l'activation du plugin pour un site, et/ou de l'activer/désactiver pour l'intégralité de l'installation.

À partir de là, il est possible pour chaque site d'activer/désactiver et configurer un plugin installé via la partie administration du site.



== Déclencheurs (triggers) ==


Pour le moment, 8 triggers sont disponibles :
* preauth : déclenché avant l'authentification de l'utilisateur
* postauth : déclenché après l'authentification de l'utilisateur
* prelogin : déclenché avant le login de l'utilisateur
* postlogin : déclenché après le login de l'utilisateur
* preview : déclenché avant l'affichage d'une page
* postview : déclenché après l'affichage d'une page
* preedit : déclenché avant l'exécution d'une action par le controlleur
* postedit : déclenché après l'exécution d'une action par le controlleur


== Démarrage ==


Un plugin est un répertoire contenu dans /share/plugins/custom/.
Le nom du répertoire correspond au nom du plugin.
Au minimum, le plugin doit contenir les fichiers suivants :
* config.xml : fichier XML de configuration.
* nom_du_plugin.php : une classe ou des fonctions

Si vous utilisez une classe, elle doit impérativement étendre la classe abstraite 'Plugin' définie dans lodel/scripts/plugin.php, et définir les 2 méthodes 'enableAction' et 'disableAction', qui seront appellées à l'activation/désactivation du plugin.
De plus, la classe 'Plugin' va remplir automatiquement la propriété protégée $_config avec la configuration du plugin pour le site, et une méthode protégée '_checkRights' qui prend en argument un niveau d'utilisateur (entier, voir le fichier /lodel/scripts/auth.php pour les valeurs) et retourne un boolean selon si l'utilisatuer a un niveau supérieur ou égale à celui spécifié.

== config.xml ==


Le fichier de configuration doit contenir au moins une information qui correspond au type d'appel qui sera effectué par Lodel : 'hookType', qui peut avoir comme valeur 'class' ou 'func', qui désignent respectivement que le plugin est développé en utilisant une classe, ou des fonctions.

Voici un exemple commenté d'un fichier de configuration :

<pre>
<?xml version="1.0" encoding="utf-8" ?>
<LodelPlugin>
	<title>nom du plugin</title>
	<description>description du plugin</description>
	<sql>0</sql> <!-- mettre à 1 si le plugin ajoute une table dans la base de données. 
                          Attention, dans ce cas il faut qu'un fichier dao.php soit présents dans le répertoire du plugin, et contienne la déclaration des champs de la table. 
                          Voir les fichiers contenus dans /lodel/scripts/dao/ pour des exemples. -->
	<hookType>class</hookType> <!-- type d'appel, peut valoir func ou class -->
	<triggers>preview,preedit,postedit</triggers> <!-- liste des triggers utilisés par le plugin. 
                          Il faut définir, pour chaque trigger, la fonction/méthode correspondante qui sera appellée par Lodel -->
	<parameters> <!-- liste des paramètres de configuration du plugin -->
        <!-- attributs : name (nom du champ), 
                         title (nom affiché dans l'interface), 
                         type (type du champ, doit correspondre à un type connu de Lodel), 
                         defaultValue (correspond à la valeur par défaut du paramètre), 
                         allowedValues (valeurs autorisées), 
                         required (indique si le paramètre est obligatoire) -->
		<param name="authorized_tags" title="Balises HTML autorisées" type="text" defaultValue="em,strong,h1,h2,h3,sup,span,a" allowedValues="" required="true"/>
		<param name="userrights" title="Droits utilisateurs" type="select" allowedValues="20,30,40,128" defaultValue="40" required="true"/>
	</parameters>
</LodelPlugin>
</pre>


Vous pouvez utiliser les variables de traductions dans le title du plugin ainsi que le title des paramètres. Pour celà, mettre un underscore '_' en début du title :
<pre>
title="_site" // sera transformé en [@LODELADMIN.SITE]
</pre>

== Exemple ==

Voici un exemple d'un plugin rajoutant le temps de calcul de la page en commentaire HTML, seulement pour les utilisateurs authentifié avec un niveau supérieur à visiteur. Il utilise le trigger 'postview' déclenché après la génération de la page, et la variable statique $microtime contenant l'heure de début de génération de la page.

<pre>
<?php
// ma classe doit absolument étendre la classe de base 'Plugins'
class monPlugin extends Plugins
{
    // pas besoin d'initialiser quoique ce soit à l'activation/désactivation du plugin
    // il faut toutefois les déclarer pour respecter la cohérence avec la classe parente
    public function enableAction(&$context, &$error) {}
    public function disableAction(&$context, &$error) {}

    public function postview(&$context)
    {
        if(!parent::_checkRights(LEVEL_REDACTOR)) { return; }
        // on récupère le contenu de la page
        $page = View::$page;
        // on récupère l'heure de début de génération
        $time = View::$microtime;
        // on calcule et concatène le temps de génération au contenu de la page
        $page .= '<!-- Page generated in '.round(microtime(true) - $time, 6).'s -->';
        // on remplace le contenu de la page dans la vue
        View::$page = $page;
    }
}
?>
</pre>


Le même plugin n'utilisant pas de classe :

<pre>
function monPlugin_postview(&$context)
{
    if(!C::get('redactor', 'lodeluser')) { return; }
    // on récupère le contenu de la page
    $page = View::$page;
    // on récupère l'heure de début de génération
    $time = View::$microtime;
    // on calcule et concatène le temps de génération au contenu de la page
    $page .= '<!-- Page generated in '.round(microtime(true) - $time, 6).'s -->';
    // on remplace le contenu de la page dans la vue
    View::$page = $page;    
}
</pre>

== Exemple : afficher la météo d'une ville précise dans l'interface lodel ==

Tous les détails : http://blog.lodel.org/173
