# Mongo DB - Prise de note

## Format de stockage

### Stockage de fichier JSON

- Format JSON plus répandu et unisersel
- Structure non imposé par le fichier JSON donc plus souple

## Méthode de stockage

### Document

- Ensemble de données clé-valeur
- Un document est dynamique et le schéma peut changer
- Imbrication de donnée possible (Object dans un object / Tableau d'object JSON).

#### Type standard JSON

- bool
- numeric
- string
- object
- array
- null

#### Type MongoDB stockable dans un document

- **date**: Entier signé 8 octets représentant le nombre depuis Epoch (Ne stock pas la time zone) -**objectId**: 12 octets utilisé en interne, auto-généré par la DB qui garanti l'unicité des id.
- **numberLong** et **numberInt**: Par défaut MongoDB considère que tous les valeurs numériques sont des nombres flotants encodés sur 8 octets.
- **numberDecimal**: Stockage sur 16 octets, plus précis sur les décimales.
- **binData**: Stockage de caractères qui ne peut être encoder en UTF-8.

https://www.mongodb.com/docs/manual/reference/bson-types/

#### **Désavantage**

- Duplication des données dans les documents.

### Collection

- Un ensemble de de documents (Un ensemble d'entrée dans la base de donnée)

## Mongo DB

### Avantage

- Scalable (Fait pour la montée en charge)
- Stocakge sur disque des fichiers
- Plus besoin d'ORM
- Performance sur les requêtes de gros volume de document

## MongoSH

**Connection à une DB MongoDB et pointage sur une collection**

```bash
mongosh <CONNECTION URI>
use <DB NAME>
```

**Insertion d'un document dans une DB**

```bash
mongosh
use <DB NAME>
db.<COLLECTION NAME>.insertOne({"key":"value"});
```

**Requête de récupération de tous les documents d'une collection**

```bash
db.<COLLECTION NAME>.find();
```

**Suppression d'une collection**

```bash
db.<COLLECTION NAME>.drop();
```

**Mise à jour d'un document**

```bash
db.<COLLECTION NAME>.updateOne(<filter>,{$set:{<modifications>}})
```

**Operateurs de mise à jour de documents**

```bash
##Exemple d'opérateur
$set
$min
$max
$inc
## Utilisation
##(Change le document Name: "Hogwell" Surname: "Sam" en Name: "Hogwell" Surname: "Eric" )

db.Users.updateOne({"name":"Hogwell"},{$set{"surname":"Eric"}})
```

**Operateurs conditionel $not**
Le fait de ne pas avoir le champ / clé-valeur inexistance rend la condition _not_ vrai

```bash
$not
```

https://www.mongodb.com/docs/manual/reference/operator/update/

## Indexation

Permet d'optimiser les requêtes en créant des "Index" permettant de pointer des champs spécifique des documents.
L'indexation permet d'éviter d'itérer à travers tous les documents durant une requête.

**Création d'un Index**

```bash
#Creation d'indexe
db.<COLLECTION NAME>.createIndex({"fieldToIndex":1},{"name":"INDEX NAME"})
```

**Suppression d'un Index**

```bash
#Creation d'indexe
db.<COLLECTION NAME>.createIndex("INDEX NAME")
```

## $expr Utilisation d'opérateur d'aggrégation dans le Query

```bash
db.<COLLECTION NAME>.find("$expr":{"$gt"["$multiply":["$VALUE_TO_MUL","FACTOR"],"$VALUE_TO_EVALUTATE"]})
```

**runCommand et adminCommand**

```bash
db.runCommand()
db.adminCommand()
```

# Exercices

## Exo Book

1. Requête de tous les employés de la collection

```bash
db.employees.find()
```

2. Requête pour trouver tous les employés agés de plus de 33 ans

```bash
#Utilisation d'un opérateur "greater ($gt)"
db.employees.find({"age":{"$gt":33}})
```

3. Requête récupérer et trier tous les employés par salaire (Ordre décroissant)

```bash
db.employees.find().sort({"salary":-1})
```

3. Requête pour afficher que le nom des employés et leurs jobs

```bash
#Utilisation d'un opérateur "greater ($gt)"
db.employees.find({},{"name":1,"job":1})
```

3. Requête pour compter le nombre d'employé par job

```bash
db.employees.aggregate([{{"$group":{"_id":"$job"},"count":{"$sum:1"}}}])
```

4. Requête de mise à jour des salaires pour tous les développeurs à 80000

```bash
db.employees.updateOne({"job":{"$eq":"Developer"}},{"$set":{"salary":80000}})
```

## Exo

1. Requête d'affichage de l'ID et du nom des salles qui sont des SMAC

```bash
db.salles.find({"smac":{"$eq":true}},{"_id":1,"nom":1})
```

2. Affichage du nom des salles ayant une capacité d'accueil strictement supérieur à 1000

```bash
db.salles.find({"capacite":{"$gt":1000}},{"nom":1})
```

3. Affichage de l'identifiant des salles n'ayant pas de numéro dans leurs adresses

```bash
db.salles.find({"adresse.numero":{"$exists":false}},{"nom":1})
```

4. Affichage de l'identifiant des salles ayant exactement 1 avis

```bash
db.salles.find({"avis":{"$size":1}},{"_id":1})
```

5. Affichage des salles et la leurs listes de styles des documents qui ont dans leurs programmation du blues

```bash
db.salles.find({"styles":"blues"},{"_id":1,"nom":1,"styles":1})
```

6. Affichage des salles qui ont en **premier** blues dans leurs listes de style

```bash
db.salles.find({"styles.0":{"$eq":"blues"}},{"_id":1,"nom":1,"styles":1})
```

7. Affichage des salles avec un code postal qui commence par "84" et avec une capacité strictement inférieur à 500

```bash
db.salles.find({ "adresse.codePostal": { "$regex": /^84/ } },{"name":1})
```

8. Affichage de l'identifiant des salles dont l'identifiant est pair ou le champ d'avis est vide

```bash
db.salles.find({"$or":[{"_id":{"$mod":[2.0,0]}},{"avis":{"$exists":false}}]})
```

9. Affichage le nom des salles qui ont une note compris entre 8 et 10 (Deux inclus)

```bash
db.salles.find({"$and":[{"avis.note":{"$gte":8}},{"avis.note":{"$lte":10}}]})
```

11. Affichage du nom et de la capacité des salles dont le produit de la valeur de l'ID multiplié par 100 est supérieur à la capacité

```bash

db.salles.find({"$expr":{"$gt":[{"$multiply":["$_id",100]},"$capacite"]}},{"nom":1,"_id":1,"capacite":1})
```
