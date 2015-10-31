En informatique, il existe un besoin constant dans les programmes : la performance. Plus précisemment, le but
est d'accomplir le plus rapidement possible une liste de tâches.

Très souvent, les performances sont apportées par un algorithme de meilleure qualité. Mais il n'est pas rare que
l'engorgement subsiste. Certains vous encourageront alors à passer à des "langages performant" tels que le C++.

Ce comportement n'est pas forcément adapté à votre besoin, développer dans ces langages peut, parfois, s'avérer
très complexe. En plus changer de technologie, c'est long.

Alors, continuez à utiliser C# ! VOus n'aurez pas besoin d'un doctorat en programmation système pour rendre vos
programmes performant. C# possède plein d'outils qui vous permettront d'optimiser tout ça **simplement**.

Avant tout, je vous propose de vous poser trois questions :

- Combien de temps mon programme passe-t-il à attendre?
- Y a-t-il plusieurs traitements que je puisse faire en parallèle?
- Mon programme utilise-t-il aux mieux les ressources de l'ordinateur (exemple : un processeur multicoeur)?

Vous vous rendrez vite compte qu'en informatique, on peut tout à fait faire un bébé en un mois du moment qu'on a 10 femmes[^typo],
et que cela est rendu possible par les technique d'asynchronisme.

[^typo]:oui, j'ai bien dit 10 femmes, c'est pas une erreur de calcul.

[[i]]
|Le tutoriel se veut accessible mais il faudra que vous ayez les bases en programmation.
|J'entends par là savoir les structures de contrôle de base (du `if` au `using`) et savoir ce que représente une classe. Il serait intéressant que vous ayez déjà créé une interface graphique.

[[a]]
|Ce tutoriel vous sera particulièrement utile si vous faites des applications mobiles. Dès qu'une tâche demande d'attendre, les applications mobiles vous obligent à utiliser l'asynchrone.