
MongoDB Vocab
============


Secletors
---------
`{field: value}` is used to find any docments where `field` is qual to a value. `{field1: value1, field2: value2}` is how you do a statment.

---
__Comparision Operations__
- `$lt` Less then
- `$lte` Less then or equal
- `$gt`  Greater than
- `$gte` Greater then or equal
- `$ne`  Not equal

```javascript
db.people.find({gender: {$ne: 'f'},
                    weight: {$gt: 150}})
```

---
__Element Operator__
The `$exists` operator is used for matching the presence or absence or a field

```javascript
// Returns people who are not vampires
db.people.find({
    vampires: {$exists: false}})
```

---
__Logical Operators__
The `$in` operator is used for mathing one of the several values that we pass as an array(technically a comparision operator)
```javascript
// Retunrs any person that loves apples, or oranges
db.people.find({
    loves: {$in: ['apples', 'orange']}})
```

If we want to join seach query we use the `$and` operator which returns all matchign conditions of both clauses
```javascript
//Returns all inventory that is not equal to 1.99
//and has a price field
db.inventory.find({$and: [{price: {$ne: 1.99}, {price: {$exists:true}}}]})
```
The oppositeof `$and` is the `$nor` operator

If we want to OR rather than AND several condition on diffrent fields use the `$or` opporator and assing it an array
```javascript
//Return all females which either love apples or weight less than 250
db.people.find({gender: 'f',
                $or: [{loves: 'apples'},
                      {weight: {$lt: 250}}]})
```

---
__Evaluation Operators__
`$text` performs a text search on the content of the fields indexed with a text index.
`{ $text: { $search: <string>, $language: <string> } }`

`$regex` selects documents where values match a specific reg expression
```javascript
{ <field>: { $regex: /pattern/, $options: '<options>' } }
```

`$mod` performs a modulo operation on the value of the field and selects documnet with a specificed result.
```javascript
{ field: { $mod: [ divisor, remainder ] } }
```

---
---
---
Updating
----

__Update: Replace Vs. $set__
In `upadte`'s sinplest form it takes two parameters
+ The selector (where) to use
+ What updates to apply to the fields
```javascript
//Timmy got fat and we need to update his record
db.people.update({name: 'Timmy'},
                 {$set: {weight: 500}})
```
The `$set` operator is critical otherwise the db will replace Timmy's record with just the weight field.

For incrementing number fields use the `$inc` operator
```javascript
//If Tyler has a birthday we have to update his age
db.people.update({name: 'Tyler'},
                 {$inc:{age: 1}})
```

By using the `$push` operator we can add new values to fields. And the reason Timmy has got so fat is becuase he found out he fucking loves sugar.
```javascript
//Adds sugar as one of Timmy's loves
db.people.update({name: 'Timmy'},
                 {$push: {loves: 'sugar'}})
```

---
__Upserts__
A `upsert` updates a document and if it is not found it inserts a new collection or updates the exsisting one, and `update` fully supports it. To enable this pass a third peramater `{upser:true}`

For example lets say we have a lets say Tyler gets a car although he has never owned a car before so thus there is not currently a car field for him. We would use `upsert` in this case.
```javascript
//Adds a car field if one does not exist, and if it does updates it
db.people.update({name: 'Tyler'},
                 {$inc: {cars: 1}}, {upsert:true})
```

---
__Multiple Updates__
By defult `update` only updates a single document. So lets say there is an outbreak of a some nasty std's and all your people in your collection get vaccinated. To reflect this change in the db we have to set `{multi: true}`
```javascript
//Updates all documents
db.people.update({},
                 {$set: {vaccinated: true}},
                 {multi: true});
```

---
---
---
Find
----
Ps. _The result from `find` is a `cursor`_

__Field Selection__
`find` takes a second opetional paramater called `projection`. This parameter is the list of fields we want to retrieve or exclude. So if you want to find just peopels names, you would use a `projection` paramater.
```javascript
//Return only the name field
db.people.find({}, {name:1})
```
By defualt the `_id` field is always returned, although if you dont want the `_id` included specify it with your `projection` as such `{name: 1, _id:0}`
```javascript
//Return only the name field plus exculde _id
db.people.find({}, {name:1, _id:0})
```

---
__Ordering__
By using `sort` you can return a ordered cursor via a chain. Specift the fields that you want to sort on as a JSON doc using 1 for ascending and -1 for desending
```javascript
//Order people by wieght - heaviest first
db.people.find().sort({weight: -1})

//Abc order by name, then most number of cars
db.people.find().sort({name: 1,
                       cars: -1});
```
As a side note: since MongoDB is a relational db you can use an index for sorting. However, Mongo limits the size of your sorts without an index.

---
__Paging__
Paging results can be accomplished via the `limit` and `skip` cursor method. To get the second and third heaviest person in our collection we would do the following.
```javascript
//Returns the second and third heaviest people
db.people.find()
         .sort({weight: -1})
         .limit(2)
         .skip(1)
```

---
__Counting__
To find the count you use the `count` operator!
```javascript
//See how many people weight over 200
db.people.find({weight: {$gt: 200}}).count()
```

---
---
---
Data Modeling
-------------
__No Joins__
Well since I have very limited SQL DB experiance this is A-OK with me. Although, to join data you have to do so within your applications code. Essentially you need to issue a second query to `find` the relavant data in a second collection.
```javascript
db.employees.insert({_id: ObjectId(
    "4d85c7039ab0fd70a117d730"),
    name: 'Leto'})
```
Now let's add two more employees whos manager is Leto
```javascript
db.employees.insert({_id: ObjectId(
    "4d85c7039ab0fd70a117d731"),
    name: 'Duncan',
    manager: ObjectId(
    "4d85c7039ab0fd70a117d730")});
db.employees.insert({_id: ObjectId(
    "4d85c7039ab0fd70a117d732"),
    name: 'Moneo',
    manager: ObjectId(
    "4d85c7039ab0fd70a117d730")});
```
And to find all of Leto's employees you would just do the following
```javascript
db.employees.find({manager: ObjectId(
    "4d85c7039ab0fd70a117d730")})
```

---
__Arrays and Embedded Documents__
REmember that Mongo supporst arrays as first class objs of a document. Well this is very useful when dealing with many-to-one or many-to-many relationships. If an employee could have two managers, you could simply store it in an array.
```javascript
//Employee has two managers
db.employees.insert({_id: ObjectId(
    "4d85c7039ab0fd70a117d733"),
    name: 'Siona',
    manager: [ObjectId(
    "4d85c7039ab0fd70a117d730"),
    ObjectId(
    "4d85c7039ab0fd70a117d732")] })
```
Of particular interest is that for some documents, `manager` can be a scalar value, while for others it can be an array. Our first `find` query will work for both.
```javascript
db.employees.find({manager: ObjectId(
    "4d85c7039ab0fd70a117d730")})
```

Mongo also supports embedded docs. As such:
```javascript
db.employees.insert({_id: ObjectId(
    "4d85c7039ab0fd70a117d734"),
    name: 'Ghanima',
    family: {mother: 'Chani',
        father: 'Paul',
        brother: ObjectId(
    "4d85c7039ab0fd70a117d730")}})
```
You then use the dot-notation to query
```javascript
db.employees.find({
    'family.mother': 'Chani'})
```