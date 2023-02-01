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

## GeoJson

### L'opérateur $nearSphere

```bash
# Retourne les documents par distance dans l'ordre décroissant
$nearsphere:{
    $geometry:{
        "type":"point",
        "coordinates":[<LONGITUDE>,<LATITUDE>],
        "$maxDistance": <OPTIONAL>,
        "$minDistance": <OPTIONAL>,
    }
}
```

### Recherche des documents par distance

```bash
db.avignon.find({"localisation":{$nearSphere:{$geometry:opera}}},{"_id":0,"nom":1})
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

13. Affichage des codes postaux des salles

```bash
db.salles.find({},{"adresse.codePostal":1,"_id":0})
```

14. Mettre à jour toute les salles en rajoutant 100 personnes à leur capacité

```bash
db.salles.updateMany({},{"$inc":{"capacite":100}})
```

15. Ajouter jazz dans les salles qui n'en programme pas

```bash
db.salles.updateMany({"styles":{"$nin":["jazz"]}},{"$push":{"styles":"jazz"}})
```

16. Retirer funk à toute les salles dont les id ne sont pas égal à 2 ou 3

```bash
db.salles.updateMany({"$nor":[{"_id":{"$eq":2}},{"_id":{"$eq":3}}]},{"$pull":{"styles":"funk"}})
```

17. Ajouter Techno et Reggae au salles qui ont l'ID 3

```bash
db.salles.updateMany({"_id":{"$eq":3}},{"$addToSet":{"styles":{"$each":["Techno","Regae"]}}})
```

18. Pour les salles dont le nom commence par "P" (majuscule ou miniscule), augmenter la capacité de 150 places et rajouter un champ contact avec un document téléphone.

```bash
 db.salles.updateMany({"nom":{"$regex":/^p/i}},{"$inc":{"capacite":150},"$set":{"contact":{"telephone":"04 11 94 00 10"}}})
```

19. Rajouter un tableau de document "avis" avec un document composé d'un champ _"date"_ qui vaut la date courante et un champ _"note"_ valant 10 à tous les documents commençant par _AEIOU_ peu importe la casse.

```bash
db.salles.updateMany({"nom":{"$regex":/[^aeiou]+$/}},{"$set":{"avis":[{"date":new Date(),"note":10}]}})
```

20. En mode upsert, mettre à jour le document dont le nom commence par _Z_. Le champ capacité passe à 50 et le SMAC passe à false

```

```

### Exercice opérateur $where

10.
11.

## Exo Index

**Exercice 1**

```bash
#Proposition de création d'indexe
db.salles.createIndex({"capacite":1,"adresse.codePostal":1})
```

La création d'index qui couvre la capacité et le code postal permettra d'optimiser les requêtes de **lecture** récurantes sur ces deux champs.

## Exo Validation

```json
//Fichier de validation
{
  "$jsonSchema": {
    "bsonType": "object",
    "title": "Salles ObjectValidation",
    "required": [
      "nom",
      "capacite",
      "adresse",
      "adresse.codePostal",
      "adresse.ville"
    ],
    "properties": {
      "nom": {
        "bsonType": "string",
        "description": "'Nom' is required (type string)",
        "minLength": 1
      },
      "capacite": {
        "bsonType": "int",
        "minimum": 1
      },
      "adresse.codePostal": {
        "bsonType": "string",
        "minLength": 1
      },
      "adresse.ville": {
        "bsonType": "string",
        "minLength": 1
      }
    }
  }
}
```

1. Durant une tentative d'insertion d'une nouvelle salle avec une validation mise en place, nous avons une erreur de validation signalant qui manque le champ code postal.

Le document n'est pas inséré dans la collection.

Pour régulariser l'insertion du nouveau document il faut ajouter le champ adresse.codePostal avec une valeur.

2. On peut apercevoir qu'après la requête que la base de donnée itère à travers tous les documents et vérifie la validité de chaque document.
