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
```
nettoyer le plan de travail
pour chaque ustensile ou plat dans l'évier:
    attendre que l'ustensile soit disponible
    nettoyer l'ustensile
	rincer l'ustensile
	essuyer l'ustensile
	ranger l'ustensile
```
