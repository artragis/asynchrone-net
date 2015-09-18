Vous vous souvenez sûrement de la première utilité que j'ai donnée à la programmation asynchrone dans le premier chapitre:
éviter de bloquer le programme lorsqu'il attend quelque chose.

Nous avons observé la manière la plus conseillée de faire lorsqu'on utilise C# : le task-based asynchron pattern.
Mais une des premières méthodes qui a été historiquement développée dans le langage utilisait une philosophie différente directement basée
sur la programmation événementielle.

En C# implémenter un évènement est très simple il suffit de déclarer dans notre classe `public event TypeEvenement OnEvenement;` puis
d'y ajouter les actions qui doivent se passer lorsque l'événement est déclenché.

De ce fait 