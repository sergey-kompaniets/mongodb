1. Накатить бэкап базы.
```
mongodump --db users
use users
db.dropDatabase()
  { "dropped" : "users", "ok" : 1 }
mongorestore --db users dump/users
  done
```

2. Найти средний возраст людей в системе.
```
let avgAge = db.users.aggregate([{
 $group: {
  _id: null,
  avgAge: {
   $avg: "$age"
  }
 }
}]).toArray();
```
Response
```
{
 "_id": null,
 "avgAge": 30.38862559241706
}
let avg = avgAge[0].avgAge;
```
3. Найти средний возраст в штате Аляска.
```
avgAge = db.users.aggregate([{
 $match: {
  address: {
   $regex: "Alaska"
  }
 }
}, {
 $group: {
  _id: null,
  avgAgeAlaska: {
   $avg: "$age"
  }
 }
}]).toArray();
```
Response
```
{
 "_id": null,
 "avgAgeAlaska": 31.5
}
let avg_alaska = avgAge[0].avgAgeAlaska;
```
4. Начиная от Math.ceil(avg + avg_alaska) (порядковый номер документа в БД ) найти первого человека с другом по имени Деннис.
```
let number = Math.ceil(avg + avg_alaska);
db.users.findOne({
 $and: [{
  friends: {
   $elemMatch: {
    name: {
     $regex: "Dennis"
    }
   }
  }
 }, {
  index: {
   $gte: number
  }
 }]
});
```
Response
```
{
 "_id":{"$oid":"5adf3c1544abaca147cdd47c"},
 "index": 306,
 "name": "Esperanza Blevins"
}
```
5. Найти **активных** людей из того же штата, что и предыдущий человек и посмотреть какой фрукт любят больше всего в этом штате (аггрегация).
```
db.users.aggregate(
 [{
  $match: {
   address: {
    $regex: "Utah"
   },
   isActive: true 
   }
  },
  {
  $group: {
   _id: "$favoriteFruit",
   count: {
    $sum: 1
   }
  }
 }]
);
```
Response
```
{ "_id" : "strawberry", "count" : 1 }
{ "_id" : "apple", "count" : 4 }
{ "_id" : "banana", "count" : 2 }
```
6. Найти саммого раннего зарегистрировавшегося пользователя с таким любимым фруктом.
```
db.users.find({
 favoriteFruit: {
  $regex: "apple"
 }
}).sort({
 registered: 1
}).limit(1);
```
Response
```
{
 "_id": ObjectId("5adf3c1544abaca147cdd568"),
 "name": "Magdalena Compton",
 "registered": "2014-01-02T10:16:56 -02:00",
 "favoriteFruit": "apple"
}
```
7. Добавить этому пользовелю свойтво: { features: 'first apple eater' }.
```
db.users.update({
 "_id": ObjectId("5adf3c1544abaca147cdd568")
}, {
 $set: {
  "features": "first apple eater"
 }
});
```
Response
```
WriteResult({
 "nMatched": 1,
 "nUpserted": 0,
 "nModified": 1
})
```
8. Удалить всех любителей клубники (написать количество удаленных пользователей).
```

db.users.remove({
 favoriteFruit: {
  $regex: "strawberry"
 }
});
```
Response
```
WriteResult({ "nRemoved" : 253 })
```
