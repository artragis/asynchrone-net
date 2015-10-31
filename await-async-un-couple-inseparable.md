Le C# se veut dès sa conception un langage *moderne*. Cela signifie qu'il essaie d'apprendre des erreurs et des succès des langages qui ont existé avant lui.

L'un de ces succès a été la mise en place de systèmes de coroutines et de multithreading facilement programmable.

La méthode choisie pour obtenir cette facilité est souvent[^async-python] l'introduction de deux mots clefs async et await. Arrivés en C#3 ces deux mots clefs sont inséparables.

[^async-python]: Et cela n'est pas une mince affaire comme le prouve l'exemple de la version 3.5 du langage [python](http://sametmax.com/async-await-la-feature-de-derniere-minute-de-python-3-5/)

# Utiliser async et await

Ces mots clefs sont plutôt débrouillards : ils exécutent à eux seuls une incroyable quantité d'instructions.

Avant d'en détailler les subtilité, faisons en sorte de garder un esprit clair et définisson LE terme le plus important de ce cours : ***asynchrone***.

Mettons nous dans une situation de votre vie courante : faisons cuire des cookies.

Un résumé de la recette peut être celle-ci:

```text
jusqu'à ce que la file des ingrédient soit vide
   prenez l'ingrédient
   enlevez-le de son emballage/coquille
   déterminez la quantité adéquate
   mettez la quantité dans le récipient
   mélangez
formez les cookies
mettez-les au four
attendre quelques dizaines de minutes
sortez les cookies du four
```

Pour bien comprendre notre problématique, mettez-vous dans la peau d'un système d'exploitation, ou d'un processeur.
Vous avez exécuté une série d'instruction assez basiques, et d'un coup, on vous demande d'attendre.

Alors vous attendez.

Et si quelqu'un vous demande de l'aide, vous dites "non, je dois attendre".

Avouons que cela est peu pratique. Alors vous avez une idée. Vous vous dites "je sais, je vais mettre un minuteur, et je reviendrai dans la cuisine uniquement quand il sonnera".

Voilà, vous venez de faire votre première action asynchrone.

Maintenant, vous êtes libre d'aider ceux qui vous le demandent. La seule condition: il faudra que vous soyez en mesure d'entendre le bip sonore 
et que vous ne soyez pas trop loin de la cuisine afin de ne pas laisser trop longtemps les cookies cuire.
De même il faut que vous gardiez en mémoire que vous attendez ce bip sonore.

Pour une application en C# ça sera le travail de async et await de faire tout ça. Alors allons-y, codons :

```csharp

namespace test_cookie
{
    class Program
    {
        static void Main(string[] args)
        {
            faitDesCookies();
            Console.In.ReadLine();
        }
        static async Task faitDesCookies() // indique que la méthode va faire quelque chose d'asynchrone
        {
            Queue<Ingredient> liste = Ingredient.GetRecette();
            while (liste.Count > 0)
            {
                Ingredient ingredient = liste.Dequeue();
                ingredient.Take();
                ingredient.UnPackage();
                ingredient.MesureQuantity();
                Console.Out.WriteLine("ajouté");
                Console.Out.WriteLine("On mélange");
            }
            Console.Out.WriteLine("On forme les cookies et on les met au four");
            // attend pendant 5 secondes mais ne truste pas les ressources
            await Task.Factory.StartNew(() => { Console.Out.WriteLine("On met le minuteur."); 
                                                System.Threading.Thread.Sleep(5000);
                                                Console.Out.WriteLine("DRIIIIIIIIIIIIIIIIING"); });
            Console.Out.WriteLine("Sortir les cookies du four");
        }
    }
}
```

[[secret]]
|Juste comme ça, voici la classe ingrédient, pas très bien codée :
|```csharp
|    class Ingredient
|    {
|        public string Name { get; set; }
|        public string Action { get; set; }
|        public int Quantity { get; set; }
|        public void Take()
|        {
|            Console.Out.WriteLine("J'ai pris " + Name);
|        }
|
|        public void UnPackage() {
|            Console.Out.WriteLine(Action);
|        }
|
|        public void MesureQuantity()
|        {
|            Console.Out.WriteLine(Quantity);
|        }
|
|        public static Queue<Ingredient> GetRecette()
|        {
|            Queue<Ingredient> liste = new Queue<Ingredient>();
|            liste.Enqueue(new Ingredient
|            {
|                Name = "farine",
|                Quantity = 500,
|                Action = "Rien de spécial"
|
|            });
|            liste.Enqueue(new Ingredient
|            {
|                Name = "chocolat pâtissier",
|                Quantity = 500,
|                Action = "Faire des morceaux"
|
|            });
|            liste.Enqueue(new Ingredient
|            {
|                Name = "beurre",
|                Quantity = 250,
|                Action = "Faire fondre"
|
|            });
|            liste.Enqueue(new Ingredient
|            {
|                Name = "Cassonade",
|                Quantity = 200,
|                Action = "Enlever les gros morceaux"
|
|            });
|            liste.Enqueue(new Ingredient
|            {
|                Name = "oeufs",
|                Quantity = 2,
|                Action = "Casser"
|
|            });
|            liste.Enqueue(new Ingredient
|            {
|                Name = "sucre vanillé",
|                Quantity = 2,
|                Action = "Rien de spécial"
|
|            });
|            liste.Enqueue(new Ingredient
|            {
|                Name = "sel",
|                Quantity = 1,
|                Action = "Prendre une pincée"
|
|            });
|            liste.Enqueue(new Ingredient
|            {
|                Name = "levure",
|                Quantity = 1,
|                Action = "Vider le sacher"
|
|            });
|            return liste;
|        }
|    }
|```

[[q]]
|Hey, il y a une erreur dans ton code, ta méthode elle retourne pas de Task!

Eh eh! je vous avais prévenu : async et await font vraiment beaucoup, beaucoup de choses. Et notamment il y a une sorte de "return magique" qui est fait.

Plus précisemment, quand vous dites qu'une méthode est async, vous annoncez que ça sera une *tâche* qui va devoir un jour attendre.
Du coup le compilateur va regarder le type de retour de la fonction. Et il s'attend à soit `Task` soit `Task<Un type personnel>`.

Une fois cela fait, il va *instancier* cet objet, et y exécuter le code de la méthode. Si vous avez dit `async Task` il comprend "la tâche à exécuter ne retourne rien".
C'est pourquoi je n'ai pas besoin de faire de `return` dans ma fonction. A l'opposé si j'avais demandé un `async Task<int>`
il se serait attendu à ce que je retourne un `int`.

[[a]]
|Vous trouverez dans la documentation que `async void` est aussi possible MAIS il est fortement déconseillé.
|Il n'est là que pour un seul cas : quand vous voulez créer un event listener. Nous reviendrons plus tard sur ce cas.

Plus tard je fais donc appelle à await. Ce dernier va dire au programme principal "bon là je vais devoir attendre un signal,
alors je te laisse libre mais revient me voir rapidement".

[[q]]
|Et ton histoire de Task factomachin là?

await n'est capable d'attendre que des fonctions asynchrone. On a vu qu'en fait une fonction asynchrone est une Task qui a été créé par async.
Mais on peut aussi la créer nous même,pour cela, il faut utiliser `Task<Votre Type De Retour>.Factory.StartNew(()=>lecode de la fonction; return ce_qu_il_faut)`.
C'est à la main la même chose que lorsqu'on avait fait async.

# L'objet Task volume 1

Task est un objet particulièrement intéressant car il ouvre les portes d'un nombre assez conséquent de fonctionnalités et de concepts.
Nous l'explorerons tout au long de ce cours, mais je tenais à vous introduire un deuxième concept important dans ce cours : le **parallélisme**.

Reprenons notre recette de cookies, et pallions à un problème majeur : nous n'avons pas nettoyé notre cuisine !

L'idée serait de dire "nettoyons notre cuisine pendant que nous attendons".

Techniquement, il existe deux grandes solutions pour paralléliser des tâches :

- Utiliser deux unités de calculs différentes, autrement dit les coeurs de votre processeurs
- Utiliser les *coroutines*.

L'idée derrière les coroutines est celle-ci :

- lancer un "cadenceur" en tâche de fond;
- lancer les coroutines une par une dans le processus du cadenceur;
- régulièrement le cadenceur vérifie l'état des coroutines et avertit le programme principal de leur fin.

![Schéma de fonctionnement d'une coroutine](http://zestedesavoir.com/media/galleries/2583/3b9d9738-7543-4038-b8f7-42c287b99130.png.960x960_q85.jpg)

L'idée maintenant sera de lancer les `Task` asynchrones puis de demander "attend qu'elles soient toutes finies".

Et ça se fait très simplement :


```csharp
List<Task> tasks = new List<Task>();
// On lance les tâches mais on ne les attend pas
tasks.Add(Task.Factory.StartNew(() => { Console.Out.WriteLine("On met le minuteur."); 
                                                System.Threading.Thread.Sleep(5000);
                                                Console.Out.WriteLine("DRIIIIIIIIIIIIIIIIING"); });
tasks.Add(Task.Factory.StartNew(()=> {System.Threading.Thread.Sleep(4000);/*trop facile de nettoyer :p*/});
await Task.WhenAll(tasks);// puis on attend que tout soit fini
```

[[a]]
|Une fonction `WaitAll` existe mais ce n'est pas une méthode asynchrone, elle met en pause le programme. Je vous conseille d'utiliser cette méthode dans votre fonction
|main qui ne peut être asynchrone. A l'opposé, partout ailleurs, vous souhaiterez la majorité du temps utiliser `WhenAll`

Si vos `Task` retournaient une valeur, un entier par exemple, vous pourriez retrouver l'ensemble des valeurs dans le tableau que retourne `WhenAll`.

[[i]]
|L'objet `Task` est vraiment primordial, il est à la base de toute un paradigme de programmation asynchrone appellé
|["Task-based Asynchronous Pattern"](https://msdn.microsoft.com/en-us/library/hh873175.aspx) qu'on peut traduire par "Programmation Asynchrone basée sur les Tâches". 
|Vous vous doutes bien qu'il existe de ce fait deux autre "Patterns", nous le verrons plus tard. Je vous conseille néanmoins d'utiliser le TAP.

