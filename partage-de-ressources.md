Lorsque l'on exécute un programme ligne après ligne, on ne se rend pas compte combien on se facilite la tâche en ce qui concerne les ressources.

Par exemple, revenons à notre recette.

Lorsque nous avions parallélisé nos tâches, nous avions d'un côté la mise en route du four et d'un autre le nettoyage de la cuisine.

Pour simplifier les choses j'avais juste dit qu'à chaque fois on ne faisait qu'attendre. Mais en fait ça va un peu plus loin.

Pour "faire cuire" disons que la tâche se dégage ainsi:

```bash
prendre le plat
ouvrir le four
mettre le plat dans le four
fermer le four
attendre 15 minutes
ouvrir le four
prendre le plat
enlever les cookies du plat
poser le plat dans l'évier pour nettoyage
```

tandis que la partie "nettoyage" se détaille en :

```bash
nettoyer le plan de travail
pour chaque ustensile ou plat dans l'évier:
    nettoyer l'ustensile
	rincer l'ustensile
	essuyer l'ustensile
	ranger l'ustensile
```

[[q]]
|Du coup le plat à cookie, on le lave ou pas?

En effet, que faisons nous? On peut se rendre compte que notre plat pose problème : il est utilisé par les deux tâches.
Je pense qu'il est évident pour tous qu'il ne peut à la fois être dans le four et être nettoyé.
Alors, que faire?

En fait il faudra modifier la seconde tâche pour dire:

```bash
nettoyer le plan de travail
pour chaque ustensile ou plat dans l'évier:
    attendre que l'ustensile soit disponible
    nettoyer l'ustensile
	rincer l'ustensile
	essuyer l'ustensile
	ranger l'ustensile
```

[[a]]
|La solution paraît facile dit comme ça, mais, elle peut mener à pas mal de problèmes, notamment un *dead lock* c'est à dire une coroutine A 
|qui attend qu'une coroutine B libère une ressource. Pendant ce temps la coroutine B attend que A libère la même ressource.

Ce cas arrivera souvent quand vous voudrez partager une liste d'objets entre plusieurs Task pour que certaines consomme la liste alors que d'autres y ajoute des objets.

Je vous conseille fortement d'éviter de partager des ressources. Préférez exécuter d'abord toutes les tâches qui obtiennent des objets puis celles qui les consomme.

Par contre si vous n'y arrivez pas, lire la suite vous permettra de vous en sortir.

Pour attendre qu'une ressource soit "disponible", il existe globalement deux possibilités mises en place par le système d'exploitation:

- La sémaphore (qui peut être un domaine très complèxe à comprendre)
- Le MutEx, abréviation de *Mutuellement Exclusif*.

L'un comme l'autre sont à envisager dans le cadre du *multi thread* et du *multiprocess*. Mais ils demandent des notions que ce cours
n'a pas pour but d'aborder. Du coup, nous allons utiliser ce que .NET appelle [`SemaphoreSlim`](https://msdn.microsoft.com/fr-fr/library/system.threading.semaphoreslim%28v=vs.110%29.aspx?f=255&MSPPError=-2147217396),
qui est un objet qui permet de gérer les 
ressources des coroutines/Task.

Cet objet se trouve dans `System.Threading` et il se base sur deux compteurs :

- le premier est le compteur de ressources libre. A priori le nombre de ressource libre initial doit valoir 1 ou 0. 
- le second est le compteur de ressource disponibles au maximum. A priori il sera toujours de 1.

L'utilisation se fait ainsi:

```csharp
        static void Main(string[] args)
        {
            Task t = RunAsync();
            t.Wait();
            Console.In.ReadLine();
        }
        public static async Task RunAsync()
        {
            using (SemaphoreSlim verrouRessource = new SemaphoreSlim(0, 1))// on crée un sémaphore qui ne peut libérer qu'une ressource mais qui la marque comme occupée pour l'instant
            {
                Queue<int> ressourcePartage = new Queue<int>();
				//on prépare les tâches
                List<Task> taches = new List<Task>();
                Console.Out.WriteLine("Start write");
                taches.Add(EnvoieDansLaFile(ressourcePartage, verrouRessource));
                Console.Out.WriteLine("Start read");
                taches.Add(LireLaFile(ressourcePartage, verrouRessource));
                verrouRessource.Release();//on peut commencer
                await Task.WhenAll(taches.ToArray());
            }
        }
        public static async Task EnvoieDansLaFile(Queue<int> file, SemaphoreSlim verrou)
        {
            for (int i = 0; i < 15; i++)
            {
                await verrou.WaitAsync();//on attendq que la ressource soit libre
                file.Enqueue(i);//on utilise la ressource
                verrou.Release();//on dit qu'on a finit d'utiliser la ressource pour l'instant
                System.Console.Out.WriteLine("On a ajouté " + i.ToString());
                Thread.Sleep(100);//on attend histoire que l'exemple soit utile
            }
        }

        public static async Task LireLaFile(Queue<int> file, SemaphoreSlim verrou)
        {
            bool first = true;
            await verrou.WaitAsync();//on va utiliser la ressource
            while (file.Count > 0 || first)
            {
                // pour éviter le cas où cette tâche serait exécutée avant la première fois où on met un entier
                if (file.Count > 0 && first)
                {
                    first = false;
                }
                System.Console.Out.WriteLine(file.Dequeue());
                verrou.Release();//on enlève le verrou
                System.Console.Out.WriteLine("on attend le prochain tour");
                Thread.Sleep(150);
                await verrou.WaitAsync();//on va utiliser la ressource au prochain tour de boucle
            }
        }
```

Le résultat montre bien le phénomène :

![Exécution en parallèle avec ressource partagée](archive:partage.png)

[[i]]
|Vous noterez que j'ai initié mon `SemaphoreSlim` avec aucune ressource disponible. Cela m'a permis de déclencher les tâches de manière à ce qu'elles exécutent toute la partie *préparatoire* puis bloque sur la ressource.
|Ensuite j'ai dit que la ressource était libre, ce qui a eu pour conséquence de lancer les tâches.

# Le cas particulier de la fenêtre graphique

Si jusque là j'ai présenté les "ressources" comme quelque chose dans laquelle on peut lire ou écrire (ce qui peut être une liste, une imprimante, un objet connecté...),
il existe un cas particulier qu'il me faut absolument vous présenter : l'interface graphique.

Si ce cas est si particulier, c'est qu'il est particulièrement verrouillé pour éviter de donner au logiciel un état instable.
De plus, votre interface graphique doit toujours être fluide. Et c'est là que les problèmes arrivent. En effet, si votre interface
doit réagir à une action de l'utilisateur, disons un click, l'évènement est exécuté de manière synchrone au moteur de rendu de la fenêtre.

Cela pose donc un problème lorsque vous devez faire des calculs importants ou que vous devez *attendre*. Pendant le temps de l'exécution
de votre évènement, l'interface sera figée. 

Heureusement, l'objet Task et à async/awaint vous éviteront ce soucis : le thread principal continuera de s'exécuter
de manière fluide.

## Cas numéro 1 : la modification de l'interface se fait avant ou après la tâche, pas pendant

Pour gérer ce cas, vous n'aurez pas vraiment de nouveau concept à apprendre, vous serez peut être simplement heureux d'apprendre
qu'un délégué peut tout à fait être async. Par exemple, si vous avez une tâche asynchrone à réaliser au click :

```csharp

public async Task VotreTache(){
   await QuelquechoseDeLong();
}

private async void DoClick(object sender, EventArg args){ //votre délégué pour le click
    this.Button1.Text = "Before the task"
    await VotreTache();
    this.Button1.Text = "After the task"
}
```

Rien de très compliqué, on notera simplement que le listener d'événement est le seul cas où l'on tolère que votre fonction asynchrone
retourne `void`.

## Cas numéro 2: la tâche elle-même modifie l'interface au cours de son exécution

Ce cas arrivera par exemple si votre tâche récupère des données de plusieurs sources différentes. Les affiche, puis les traite
pour par exemple changer la couleur d'un indicateur ou l'état d'une barre de progression.

Il me faut alors dire que temporairement, le temps de changer l'interface, la tâche (ou une sous tâche) doit s'exécuter
dans le thread de la GUI. Pour cela, nous allons utiliser la notion de *contexte d'exécution*. 

Si les `Task` peuvent être démarrées dans un thread différent, ce n'est pas forcément le cas. Vous pouvez leur demander, 
de s'exécuter dans le même thread que la tâche parente ou dans un contexte qui ne demande pas l'établissement de nouveaux thread.

A l'opposée, les parties de la tâche que vous désirez paralléliser peuvent être lancées dans une `threadpool`.

```csharp
public async Task VotreTache(){
   
   await QuelquechoseDeLongQuiNeTouchePasALaGui().ConfigureAwait(continueOnCapturedContext:false);
   await QuelquechoseDeLongQuiToucheALaGui().ConfigureAwait(continueOnCapturedContext:true);
}

private async void DoClick(object sender, EventArg args){ //votre délégué pour le click
    this.Button1.Text = "Before the task"
    await VotreTache().ConfigureAwait(continueOnCapturedContext:true);//on continue sur la GUI
    this.Button1.Text = "After the task"
}
```

*[GUI]: Graphical UserInterface
