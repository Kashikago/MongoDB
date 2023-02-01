# TP Météo

## Préparation des données

Pour ce TP nous allons utiliser les données météos de Seatle (https://www.kaggle.com/datasets/ananthr1/weather-prediction)

![indexation](img/importation.PNG)

Avant l'importation on défini les types pour chaque champ.

## Indexation des données météo

Création d'un index pour les dates des données météo avec le fonction `createIndex()`.
![indexation](img/indexation.PNG)
Listing des indexes présent sur la collection seatleWeather
![indexation](img/listIndexes.PNG)

## Requête MongoDB

A. Recherche des enregistrement de valeur supérieur à 25C° durant la période d'été (Juin,Juillet et Août)

```bash
##
db.seatleWeather.find(
    {"$and":
        [{"temp_max":{"$gt":25}},
            {"$expr":
                {"$or":
                    [{"$eq":[{"$month":"$date"},6]},
                    {"$eq":[{"$month":"$date"},7]},
                    {"$eq":[{"$month":"$date"},8]}]
                }
           }
        ]
    })
```

Résultat de la requête:
![SummerTempRequest](img/summerTempRequest.PNG)

B.1 La collection de document ne possède pas de donnée au sujet de la pression atmosphérique et l'identifiant de station météo.
Pour palier à ce manque voici la commande qui ajoute une valeur aléatoire pour la pression atmosphérique et l'identification des stations.

```bash
## Ajout de la pression atmosph"rique
db.seatleWeather.updateMany({},[{"$set":{"atmoPressure":{"$multiply":[{"$rand":{}},100]}}}])
## Ajout de l'ID des stations météo
db.seatleWeather.updateMany({},[{"$set":{"stationId":{"$floor":{"$multiply":[{"$rand":{}},4]}}}}])
```

B.2 Triage des stations métérologique par pression atmosphérique

```bash
db.seatleWeather.find({},{"stationId":1,"atmoPressure":1,"date":1}).sort({"atmoPressure":-1})
```

Résultat de la requête:
![SortAtmoPressure](img/triage_station_meteo_pression.PNG)

## Framework d'agrégation

A. Calcule de la moyenne des températures de chaque station météo pour chaque mois.

```js
//Pipeline utilisé par le framework d'agrégation
var pipeline = [
  {
    $group: {
      _id: {
        year: { $year: "$date" },
        month: { $month: "$date" },
        stationId: "$stationId",
      },
      avgTemp: { $avg: "$temp_max" },
    },
  },
  { $sort: { _id: 1 } },
];

db.seatleWeather.aggregate(pipeline);
```

Résultat de la requête:
![AverageTempForEachStation](img/agregation_temp_by_month.PNG)

B. Retrouver la station météo qui a enregisté la plus haute température en été.

```js
var pipeline = [
  {
    $match: {
      $expr: {
        $or: [
          { $eq: [{ $month: "$date" }, 6] },
          { $eq: [{ $month: "$date" }, 7] },
          { $eq: [{ $month: "$date" }, 8] },
        ],
      },
    },
  },
  { $sort: { temp_max: -1 } },
  { $group: { _id: "$stationId", topTemp: { $first: "$$ROOT" } } },
  { $replaceWith: "$topTemp" },
];
```
