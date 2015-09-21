Le but du TP qui va suivre est de vous permettre de vous mettre en confiance avec les notions
que nous venons de voir tout en vous permettant d'*explorer* l'API.

Il se découpe en trois morceaux:

1. Le programme de base que vous devez être capable de faire seul. La correction sera néanmoins donnée.
2. Quelques pistes d'amélioration qui s'appuient sur les notions déjà vues, aucune correction ne sera donnée néanmoins. Je vous conseille de poser vos questions sur le forum avec le tag [async][C#].
3. Une notion pratique un peu plus avancée qui demandera un peu d'exploration et de compréhension.

# L'exercice de base

## Énoncé

Votre mission, si vous l'acceptez : créer un système qui permet d'extraire tous les liens présents dans plusieurs pages web.
Une fois ces liens obtenus, il vous faudra envoyer une requête vers ces liens et sauvegarder la page html qui leur est associé.
En fin de programme, vous afficherez le temps mis pour récupérer toutes ces pages.

Vous pouvez utiliser une application console, WinForm, WPF à votre convenance.

Pour envoyer des requêtes, je vous conseille de regarder [HttpWebRequest](https://msdn.microsoft.com/fr-fr/library/system.net.webrequest.getresponseasync%28v=vs.110%29.aspx).
Pour traiter le HTML vous pouvez utiliser le package [HtmlAgility](http://htmlagilitypack.codeplex.com/) ou si vous aimez Linq : LinqToXML.

Pour mieux observer l'asynchronisme des tâches, je vous conseille de terminer chaque tâche par un `Console.WriteLine("Nom de la tâche " + une_autre_info)`. De même, je vous
conseille d'exécuter plusieurs fois le même programme pour observer l'ordre d'exécution des tâches.
Enfin, ayiez un oeil sur votre gestionnaire de ressources. Pour y accéder, utiliser `....msc`
ou bien via le gestionaire des tâche ||ctrl||+||Maj||+||echap|| puis "ouvrir le gestionnaire de ressource".

![Gestionnaire de ressources]() 

[[i]]
|Notre but n'est pas de faire un aspirateur complet de site web, juste de télécharger les pages actuelles et d'en tirer les liens et d'eux même les télécharger.
|Vous pourrez bien sûr ajouter vos propres amélioration pour faire un aspirateur MAIS, je tenais à vous rappeler que si vous désirez aspirer un site web
|il est fortement recommandé d'en demander l'autorisation au propriétaire afin que vous ne mettiez pas en périle sa disponibilité.

## Correction


# Quelques améliorations

Comme je l'ai dit avant de proposer la correction vous pouvez tout à fait songer à aspirer un site web entier.

Pour cela il faudra donc être un peu plus efficace sur la manière de faire :

- il faudra faire en sorte que les liens vers l'extérieur du site ne soient pas pris en compte;
- il faudra trouver un moyen de ne pas télécharger deux fois le même lien;
- il faudra aussi télécharger les images et les ressources externes telles que les CSS et les stocker dans le bon sous dossier;



# Allons plus loin : la progression des tâches
