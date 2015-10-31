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
ou bien via le gestionaire des tâche ||ctrl||+||Maj||+||echap|| puis "ouvrir le gestionnaire de ressources".

![Le gestionnaire de ressources nous indique le nombre de thread. 30 dans notre cas](http://zestedesavoir.com/media/galleries/2583/31af9781-e26d-4642-9431-89869da90fd3.png.960x960_q85.png)

[[i]]
|Notre but n'est pas de faire un aspirateur complet de site web, juste de télécharger les pages actuelles et d'en tirer les liens et d'eux même les télécharger.
|Vous pourrez bien sûr ajouter vos propres amélioration pour faire un aspirateur MAIS, je tenais à vous rappeler que si vous désirez aspirer un site web
|il est fortement recommandé d'en demander l'autorisation au propriétaire afin que vous ne mettiez pas en périle sa disponibilité.

## Correction
[[secret]]
| 
| ```csharp
| using HtmlAgilityPack;
| using System;
| using System.Collections.Generic;
| using System.IO;
| using System.Linq;
| using System.Net;
| using System.Text;
| using System.Threading.Tasks;
| using System.Xml.Linq;
| 
| namespace WebGetter
| {
|     class Program
|     {
|         static void Main(string[] args)
|         {
|             List<string> urls = new List<string>();
|             string[] _urls = {"http://francoisdambrine.me"};
|             urls.AddRange(_urls);
|             GetLinksThenDownloadPages(urls);
|             Console.ReadLine();
|         }
|         static async void GetLinksThenDownloadPages(IEnumerable<string> urls)
|         {
|             List<string> downloadedUrls = await GetLinks(urls);
|             await DownloadFilesFromLinks(downloadedUrls);
|         }
|         static async Task<List<string>> GetLinks(IEnumerable<string> startUrls)
|         {
|             List<Task<List<string>>> tasks = new List<Task<List<string>>>();
|             foreach(string url in startUrls)
|             {
|                 tasks.Add(DownloadLinksFromUrlAsync(url));
|             }
|             return (from list in await Task<List<string>>.WhenAll(tasks)
|                     from link in list
|                     select link).ToList();
|         }
| 
|         static async Task<List<string>> DownloadLinksFromUrlAsync(string url)
|         {
|             HttpWebRequest request = WebRequest.Create(url) as HttpWebRequest;
|             HttpWebResponse response = await request.GetResponseAsync() as HttpWebResponse;
|             using (var streamReader = new StreamReader(response.GetResponseStream()))
|             {
|                 string htmlCode = streamReader.ReadToEnd();
|                 HtmlDocument parsed = new HtmlDocument();
|                 parsed.LoadHtml(htmlCode);
|                 IEnumerable<string> aElement = from element in parsed.DocumentNode.Descendants()
|                                                where element.Name.ToString().ToLower() == "a"
|                                                where element.Attributes["href"] != null
|                                                select element.Attributes["href"].Value;
|                 return aElement.ToList();
|             }
| 
|         }
| 
|         static async Task WriteFileFromUrl(string url)
|         {
|             try {
|                 HttpWebRequest request = WebRequest.Create(url) as HttpWebRequest;
|                 HttpWebResponse response = await request.GetResponseAsync() as HttpWebResponse;
|                 Guid g = Guid.NewGuid();
|                 using (StreamReader streamReader = new StreamReader(response.GetResponseStream()))
|                 {
|                     using (StreamWriter writer = new StreamWriter(g.ToString() + ".html"))
|                     {
|                         await writer.WriteAsync(await streamReader.ReadToEndAsync());
|                         Console.WriteLine(g.ToString() + ".html : " + url);
|                     }
|                 }
|             }
|             catch(UriFormatException exception)
|             {
|                 Console.Error.WriteLine(exception.Message);
|             }
|             catch(WebException exception)
|             {
|                 Console.Error.WriteLine(url + " was correct but not downloaded due to " + exception.Message);
|             }
|         }
| 
|         static async Task DownloadFilesFromLinks(IEnumerable<string> list)
|         {
|             List<Task> tasks = new List<Task>();
|             foreach(string url in list)
|             {
|                 tasks.Add(WriteFileFromUrl(url));
|             }
|             await Task.WhenAll(tasks);
|         }
|     }
| }
| 
| ```
| Code: correction

Si vous avez une version de visualstudio qui le support, vous pourrez observer sur le profileur que le processeur est vraiment peu utilisé par notre programme.

![Le profileur nous dit que le processeur attend beaucoup](http://zestedesavoir.com/media/galleries/2583/82331b99-9ab2-4558-97c5-adb243da24ae.png.960x960_q85.png)

# Quelques améliorations

Comme je l'ai dit avant de proposer la correction vous pouvez tout à fait songer à aspirer un site web entier.

Pour cela il faudra donc être un peu plus efficace sur la manière de faire :

- il faudra faire en sorte que les liens vers l'extérieur du site ne soient pas pris en compte;
- il faudra trouver un moyen de ne pas télécharger deux fois le même lien;
- il faudra aussi télécharger les images et les ressources externes telles que les CSS et les stocker dans le bon sous dossier;



# Allons plus loin : la progression des tâches

Le TAP ne permet pas à priori d'ajouter le concept de "progression", dans le flot d'exécution des tâches.
En effet, comme le TAP ne s'occupe que du concept de "tâche", ce que le programme sait faire avec async et await c'est démarrer la tâche, détecter qu'elle
est terminée et nettoyer le contexte.

Pour s'occuper de la progression, il faudra *déléguer* cette fonctionnalité à une autre classe. Le Framework .NET vous propose alors une interface `IProgress<T>`
pour vous assurer qu'à chaque fois que vous devez faire cette délégation elle est faite sur le même modèle.

Vous devrez donc choisir le type de données qui sert à notifier la progression. La pratique la plus courante au sein du framework .NET est d'implémenter
un `IProgress<long>` qui vous permet de notifier une estimation numérique de l'avancée.

Par exemple, si vous êtes en train de lire un fichier, vous pouvez faire cette classe:

```csharp

class AfficherLesProgresDeLecture: IProgress<long>{
    public DateTime StartDate{get; private set;}
    public AfficherProgresDeLecture(){
        StartDate = DateTime.Now();
    }
    public Report(long value){
        int seconds = (DateTime.Now() - StartDate).Seconds
        Console.out.WriteLine(value.ToString() + " octets lus en " + seconds.ToString() + "secondes");
    }
}
```

Ensuite il suffit d'appeler `await stream.ReadAsync(buffer, 0, 1024, new AfficherLesProgresDeLecture());`.
Ainsi la méthode ReadAsync appellera la méthode Report à chaque fois qu'elle lit 1024 octets.

[[i]]
|Notons que pour vous éviter à réimplémenter une classe complète mais juste la méthode Report, la classe `Progress<T>` 
|existe et vous permet de préciser l'ensemble des méthodes à réaliser à chaque rapport du progrès grâce à l'évènement `OnReport`.

Je vous laisse explorer un peu l'API pour créer une barre de progression dans la ligne de commande ou pour mettre à jour le composant ProgressBar
dans une application graphique.

Si vous avez besoin d'aide, n'hésitez pas à poser vos questions sur le forum avec les tag [async][C#].



