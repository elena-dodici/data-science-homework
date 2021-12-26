## Homework 4
### Q1:

Query1: "Online" Status

```
db.bikestation.find({

	'extra.status':"online"

}).count()

``` 
 Result
 ```
 33
 ```
 

Query2: "Offline" Status
```
db.bikestation.find({

	'extra.status':"offline"

}).count()
```
  

 Result
 ```
 28
 ```
 
---------

### Q2:
Query
```
db.bikestation.find({

	'extra.status':{$nin:["online","offline"]}

}).count()

OR

db.bikestation.find(
	{$and:[{"extra.status":  {$ne:"offline"} },
	{"extra.status":  {$ne:"online"}} ]
}).count()

```

 Result
 ```
 4
 ```
 
---------

### Q3:

Query
```
db.bikestation.find({

	'extra.status':{$nin:["online","offline"]}

		},{'_id':0,'extra.status':1})
```

Result
```
{ "extra" : { "status" : "maintenance" } }
{ "extra" : { "status" : "maintenance" } }
{ "extra" : { "status" : "maintenance" } }
{ "extra" : { "status" : "maintenance" } }
```
---------

### Q4:

Query
```
db.bikestation.find({

	"extra.status":"online", 
	'extra.score':	{$gte:4}

},{'_id':0,'name':1}
).sort({'name':1})

OR

db.bikestation.find({

	$and:[{"extra.status":"online"}, 
	{'extra.score':{$gte:4}}]
	},{'_id':0,'name':1}
	
).sort({'name':1})
```

Result
```
{ "name" : "02. Pettiti" }
{ "name" : "04. Reggia" }
{ "name" : "06. Municipio" }
{ "name" : "08. San Marchese" }
{ "name" : "10. Gallo Praile" }
{ "name" : "Belfiore" }
{ "name" : "Borromini" }
{ "name" : "Castello 1" }
{ "name" : "Corte d`Appello" }
{ "name" : "Giolitti 1" }
{ "name" : "Politecnico 1" }
{ "name" : "Politecnico 3" }
{ "name" : "Porta Palatina" }
{ "name" : "Principi d`Acaja 1" }
{ "name" : "Principi d`Acaja 2" }
{ "name" : "San Francesco da Paola" }
{ "name" : "Sant´Anselmo" }
{ "name" : "Tribunale" }

```
---------

### Q5:

Query
```
db.bikestation.find(

	{$and:[{"extra.status":"offline"},

		{$or:[{"empty_slots":{$gt:0}},
	          {"free_bikes": {$gt:0}}
             ]}
         ]}
,{'_id':0,'name':1}
).sort({'name':1})

OR
db.bikestation.find(

{"extra.status":"offline",

	$or:[{"empty_slots":{$gt:0}},
		 {"free_bikes": {$gt:0}}]
		 
},{'_id':0,'name':1}
).sort({'name':1})

```
Result
```
{ "name" : "05. Corso Garibaldi" }
{ "name" : "06. Le Serre" }
```
---------

### Q6:
Query
```
db.bikestation.aggregate([

	{$group:{_id:null,
	         totalReview:{$sum:"$extra.reviews"}}
	}
])
```
Result
```
{ "_id" : null, "totalReview" : 15311 }
```
---------

### Q7:
Query
```
db.bikestation.aggregate(
	
	{ $group:{ _id:"$extra.score",
			totalReview:{$sum:1}}
	},
	{ $sort:{'totalReview':-1} }
)

```
Result
```
{ "_id" : 4, "totalReview" : 9 }
{ "_id" : 3.9, "totalReview" : 9 }
{ "_id" : 4.2, "totalReview" : 7 }
{ "_id" : 4.1, "totalReview" : 5 }
{ "_id" : 3.5, "totalReview" : 4 }
{ "_id" : 3, "totalReview" : 4 }
{ "_id" : 3.4, "totalReview" : 3 }
{ "_id" : 1, "totalReview" : 3 }
{ "_id" : 2.8, "totalReview" : 2 }
{ "_id" : 3.7, "totalReview" : 2 }
{ "_id" : 4.4, "totalReview" : 2 }
{ "_id" : 4.3, "totalReview" : 2 }
{ "_id" : 4.5, "totalReview" : 2 }
{ "_id" : 2.1, "totalReview" : 1 }
{ "_id" : 1.4, "totalReview" : 1 }
{ "_id" : 4.7, "totalReview" : 1 }
{ "_id" : 2.7, "totalReview" : 1 }
{ "_id" : 2.5, "totalReview" : 1 }
{ "_id" : 3.8, "totalReview" : 1 }
{ "_id" : 1.2, "totalReview" : 1 }
```
---------

### Q8:
Query
```
db.bikestation.aggregate([

	{ $match:{'extra.status':
	   {$in:["online","offline"]}}
    },

	{ $group:{ _id:"$extra.status",
	         AveRating:{$avg:"$extra.score"}}
	}
])

```
Result
```
{ "_id" : "online", "AveRating" : 3.8454545454545452 }
{ "_id" : "offline", "AveRating" : 3.0285714285714285 }
```
---------

### Q9:
Query1(free_bike=0)
```
db.bikestation.aggregate([

	{ $match:{"free_bikes":{$eq:0}}},
	{ $group:{_id:null,
			AveRating:{$avg:"$extra.score"}

     }}

])

OR

db.bikestation.mapReduce(

	function(){emit(null,this.extra.score);},
	function(key,values){return Array.avg(values);},

{
	query:{"free_bikes":0},
	out:" AveRating_noBike "
})
//run Mapreduce Function
db. AveRating_noBike.find()



```
Result
```
{ "_id" : null, "AveRating" : 3.2305555555555556 }
```


Query2 (free_bike>0)

```
db.bikestation.aggregate([

	{ $match:{"free_bikes":{$gt:0}}},
	{ $group:{_id:null,
			AveRating:{$avg:"$extra.score"}
    }}
])

OR

db.bikestation.mapReduce(
	
	function(){emit(null,this.extra.score);},
	function(key,values){return Array.avg(values);},
{
	query:{"free_bikes":{$gt:0}},
	out:“AveRating_AtLeastOne"
})

//run Mapreduce Function
db. AveRating_AtLeastOne.find()

```
Result
```
{ "_id" : null, "AveRating" : 3.8758620689655174 }
```
---------

### Q10:

Query(status=online&free_bike=0)
```
db.bikestation.mapReduce(

function(){emit(null,this.extra.score);},
function(key,values){return Array.avg(values);},

{
	query:{"free_bikes":0,
		   "extra.status":"online"},
	out:" AveRating_noBike_Online"
})

//run Mapreduce Function
db. AveRating_noBike_Online.find()
```
Result
```
{ "_id" : null, "AveRating" : 3.8758620689655174 }
```

---------

### Q11:
Query
```
db.bikestation.createIndex({ location:"2dsphere"})

db.bikestation.find(

	{"free_bikes":{$gt:0},
	location:{$near : {$geometry:{type:'Point',
	                  coordinates:[45.07456,7.69463]},
}}},{"_id":0,"name":1,'free_bikes':1}).sort({'coordinates':1}).limit(3)
```
Result
```
{ "free_bikes" : 5, "name" : "Palermo 2" }
{ "free_bikes" : 5, "name" : "Castello 1" }
{ "free_bikes" : 4, "name" : "San Francesco da Paola" }
```
---------

### Q12:
Query
```
//define a cursor
var politoCoor = db.bikestation.find({'name':"Politecnico 4"},{'location.coordinates':1,'_id':0})

//create an index
db.bikestation.createIndex({ location:"2dsphere"})
db.bikestation.find(

	{"free_bikes":{$gt:0},
	location:{$near : {$geometry:{type:'Point',
	coordinates:politoCoor.toArray()[0].location.coordinates,}
	}}},
	{"_id":0,"name":1,'free_bikes':1}).sort({'coordinates':1}).limit(3)
```
Result
```
{ "free_bikes" : 9, "name" : "Politecnico 1" }
{ "free_bikes" : 5, "name" : "Politecnico 3" }
{ "free_bikes" : 3, "name" : "Tribunale" }
```


