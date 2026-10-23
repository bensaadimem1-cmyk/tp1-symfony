# TP2 — Réponses aux questions

## Question 1 — Avantage d'un FormType séparé
Un FormType sépare la construction du formulaire de la logique du contrôleur.
Le contrôleur reste court et lisible, le formulaire est **réutilisable**
(ex : le même `ArticleType` sert à la création ET à la modification),
et il est plus facile à maintenir et à tester.

## Question 2 — Cycle de vie d'un formulaire
1. **Affichage** : le contrôleur crée le formulaire avec `createForm()` et l'envoie à Twig, qui l'affiche vide.
2. **Soumission** : l'utilisateur remplit et envoie le formulaire (requête POST).
3. **handleRequest()** : Symfony lit la requête et remplit automatiquement l'entité avec les données.
4. **isSubmitted() / isValid()** : Symfony vérifie que le formulaire a été envoyé et que les contraintes de validation sont respectées.
5. **Enregistrement** : si tout est valide, `persist()` puis `flush()` enregistrent l'entité en base.
6. **Redirection** : on redirige avec un message flash. Si le formulaire est invalide, il est réaffiché avec les erreurs.

## Question 3 — form_row() vs form_label() + form_widget()
`form_row()` affiche en une seule fois le **label, le champ et les erreurs**.
`form_label()` + `form_widget()` (+ `form_errors()`) affichent chaque partie **séparément**.
On préfère `form_row()` pour un rendu rapide et standard, et les fonctions séparées
quand on veut une mise en page personnalisée (ex : label et champ sur la même ligne, ordre différent).

## Question 4 — Validation client vs serveur
- **Côté client** (HTML5, JavaScript) : faite dans le navigateur, rapide et confortable pour l'utilisateur, mais **facilement contournable** (désactiver JS, modifier le HTML, envoyer une requête directement).
- **Côté serveur** (Symfony Validator) : faite par l'application, **impossible à contourner**.

Il faut **toujours** valider côté serveur car on ne doit jamais faire confiance aux données reçues :
c'est la seule garantie de l'intégrité des données et de la sécurité de l'application.

## Question 5 — Migration de la relation et clé étrangère
La migration ajoute une colonne et une contrainte dans la table `article` :
```sql
ALTER TABLE article ADD categorie_id INT DEFAULT NULL;
ALTER TABLE article ADD CONSTRAINT FK_... FOREIGN KEY (categorie_id) REFERENCES categorie (id);
CREATE INDEX IDX_... ON article (categorie_id);
```
(ainsi que le `CREATE TABLE categorie`).

Une **clé étrangère** est une colonne qui fait référence à la clé primaire d'une autre table.
Ici, `article.categorie_id` pointe vers `categorie.id`. La base garantit ainsi qu'un article
ne peut pas être lié à une catégorie qui n'existe pas.

## Question 6 — createFormBuilder() vs FormType séparé
- **Avantages de createFormBuilder()** : rapide à écrire, tout est au même endroit, pratique pour un petit formulaire simple utilisé une seule fois.
- **Inconvénients** : le formulaire n'est pas réutilisable (il faut le recopier pour la modification), le contrôleur devient long et moins lisible.

Le **FormType** est donc préférable dès que le formulaire est utilisé plusieurs fois ou devient complexe.
C'est pourquoi nous avons créé `CategorieType` en partie 5.

## Question 7 — Pourquoi pas de persist() lors de la modification ?
L'article est récupéré depuis la base (par le ParamConverter / EntityValueResolver) :
il est donc déjà **géré (managed)** par Doctrine. Doctrine surveille ses modifications
grâce à l'**Unit of Work** (unité de travail). Au moment du `flush()`, il détecte
automatiquement les changements et génère la requête `UPDATE`.
`persist()` ne sert qu'à faire connaître une **nouvelle** entité à Doctrine.

## Question 8 — Attaque CSRF et danger du GET
Une attaque **CSRF** (Cross-Site Request Forgery) consiste à faire exécuter une action à un
utilisateur connecté **à son insu**, depuis un autre site (ex : un lien ou une image piégée
qui déclenche une requête vers notre application avec la session de la victime).

Utiliser **GET** pour supprimer est dangereux car une simple URL suffit à déclencher la suppression :
un lien, une balise `<img>`, un robot d'indexation ou le préchargement du navigateur peuvent l'appeler.
On utilise donc **POST** (qui ne se déclenche pas par un simple lien) et un **token CSRF**,
une valeur secrète générée par le serveur et vérifiée avec `isCsrfTokenValid()`.
Un site extérieur ne peut pas connaître ce token, donc il ne peut pas forger la requête.