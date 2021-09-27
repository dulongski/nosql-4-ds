Bases de Datos No Relacionales
Sebastian Dulong Salazar
C.U. 188172
# Tarea 2

`use trainingsessions`

4. ¿Cómo podemos saber si los tuiteros hispanohablantes interactúan más en las noches?

Tomé noche de 19:00 a 23:59:59 según la página https://www.inmsol.com/es/gramatica-espanola/la-hora/#:~:text=las%20partes%20del%20día%20se,Madrugada%20de%2024%20a%206.


```javascript
db.tweets.aggregate([
{$lookup: {from:"primarydialects","localField":"user.lang","foreignField":"lang","as":"language"}},
{$lookup: {from:"languagenames","localField":"language.locale","foreignField":"locale","as":"fulllocale"}},
{$match:{"fulllocale.languages":/Español/,"created_at":/ (19|2[0-3]):[0-5][0-9]:[0-5][0-9]/}},
{$group: {_id:"$fulllocale.languages", "conteo": {$count:{}}}}
])
```

1407 interactúan por las noches. 
```javascript
db.tweets.aggregate([
{$lookup: {from:"primarydialects","localField":"user.lang","foreignField":"lang","as":"language"}},
{$lookup: {from:"languagenames","localField":"language.locale","foreignField":"locale","as":"fulllocale"}},
{$match:{"fulllocale.languages":/Español/,"created_at":/ (0[0-9]|1[0-8]):[0-5][0-9]:[0-5][0-9]/}},
{$group: {_id:"$fulllocale.languages", "conteo": {$count:{}}}}
])
```

978 interactúan en un horario distinto a la noche.

1407/(1407+978) = .589 

Un 58.9% de hispanohablantes interactúan por la noche (la mayoría).


5. ¿Cómo podemos saber de dónde son los tuiteros que más tiempo tienen en la plataforma?

``` javascript
db.tweets.aggregate([
{$addFields: { "user.created_at": { "$toDate": "$user.created_at" }}}, //la función $toDate cabia string a ISODate para poder ordenarlos ascendentemente por fecha
{$project:{"user.created_at":1,"user.time_zone":1}},
{$sort: {"user.created_at":1}}
])
```

6. ¿En intervalos de 7:00:00pm a 6:59:59am y de 7:00:00am a 6:59:59pm, de qué países la mayoría de los tuits?

De 19:00:00 hrs a 6:59:59 hrs:

``` javascript
db.tweets.aggregate([
{$match:{"created_at":/ (19|2[0-3]|0[0-6]):[0-5][0-9]:[0-5][0-9]/}},
{$group: {_id:"$user.time_zone", "conteo": {$count:{}}}}
]).sort({conteo:-1})
```

De 7:00:00 hrs a 18:59:59 hrs:

``` javascript
db.tweets.aggregate([
{$match:{"created_at":/ (0[7-9]|1[0-8]):[0-5][0-9]:[0-5][0-9]/}},
{$group: {_id:"$user.time_zone", "conteo": {$count:{}}}}
]).sort({conteo:-1})
```

7. ¿De qué país son los tuiteros más famosos de nuestra colección?

Supondremos que fama es equiparable a número de seguidores:

``` javascript
db.tweets.aggregate([
{$group: {_id:{"lugar":"$user.time_zone","seguidores":"$user.followers_count"}}},
{$sort: {"_id.seguidores":-1}},
{$limit : 10}
])
```
