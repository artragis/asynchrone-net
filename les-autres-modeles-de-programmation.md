# EventBased Asynchronous Pattern

Vous vous souvenez sûrement de la première utilité que j'ai donnée à la programmation asynchrone dans le premier chapitre:
éviter de bloquer le programme lorsqu'il attend quelque chose.

Nous avons observé la manière la plus conseillée de faire lorsqu'on utilise C# : le task-based asynchron pattern.
Mais une des premières méthodes qui a été historiquement développée dans le langage utilisait une philosophie différente directement basée
sur la programmation événementielle.

En C# implémenter un évènement est très simple il suffit de déclarer dans notre classe `public event TypeEvenement OnEvenement;` puis
d'y ajouter les actions qui doivent se passer lorsque l'événement est déclenché.

De ce fait tout un pan de la programmation asynchrone peut être pilotée à partir d'événements. C'est ce qu'on appelle l'EventBased Asynchron Pattern. 

Le principe est de se baser sur une convention qui appellera les méthodes et les événements associés au fur et à a mesure. Les règles sont les suivantes :

1. Créer une méthode FaireQuelqueChose. Elle sera charger de faire la chose de manière synchrone ou bloquante
1. Créer la méthode FaireQuelqueChose**Async**
    - elle ne retourne rien
    - elle a les mêmes paramètres que FaireQuelqueChose
1. Créez un événement qui s'appelle FaireQuelqueChoseCompleted et qui a la signature que vous désirez par exemple `public event FaireQuelqueChoseCompletedHandler FaireQuelqueChoseCompleted;`
1. Le plus simple étant de définir FaireQuelqueChoseCompletedHandler comme ceci : `public delegate void FaireQuelqueChoseCompletedHandler (object sender, FaireQuelqueChoseCompletedEventArgs e);` (sans oublier de définir FaireQuelqueChoseCompletedEventArgs)


Bref vous l'aurez compris, ça peut vite devenir long et répétitif à implémenter, tous les conseils se trouvent [ici](https://msdn.microsoft.com/en-us/library/e7a34yad%28v=vs.110%29.aspx). Sachez juste que les BackgroundWorker du framework .NET fonctionnent selon ce principe.

#Asyncronous Programming Model

En complément du EAP, le framework .NET propose d'ajouter une autre manière de faire de l'Asynchrone à partir d'une interface nommée IAsyncResult.

Le principe est celui-ci : 

- séparer la tâche en trois parties :
    - le lancement (BeginFaireQuelqueChose)
    - faire la chose
    - terminer et traiter le résultat (EndFaireQuelqueChose)

Ce patron de conception est détaillé [ici](https://msdn.microsoft.com/en-us/library/ms228963%28v=vs.110%29.aspx) et est présent dans les objets tels que HttpWebRequest comme vous avez pu le constater.