<p align="center" style="text-align: center">
    <img src="https://i.imgur.com/0Mldh0I.png" />
</p>
<p align="center">
    <a href="https://github.com/CameronCT" alt="Author">
        <img src="https://img.shields.io/badge/GitHub-CameronCT-181717.svg?style=flat&logo=github" /></a>
    <a href="https://codecov.io/github/CameronCT/Mongo-ODM" alt="MongoDB">
        <img src="https://img.shields.io/codecov/c/github/CameronCT/Mongo-ODM" /></a>
    <a href="https://codecov.io/github/CameronCT/Mongo-ODM" alt="MongoDB">
        <img src="https://img.shields.io/github/issues/CameronCT/Mongo-ODM" /></a>
    <img src="https://img.shields.io/github/license/CameronCT/Mongo-ODM" />
</p>

An insanely basic implementation of some form of Object Document Mapping (not ORM) for MongoDB. This was not really made to be used as a widely available package, it does not contain severely insane levels of logging. 

## Getting Started

To get started you must install the package via your dependency manager (you can use yarn as well)

`npm install @cameronct/mongo-odm`

### Initiating the Connection

```js
import MongoODM from "@cameronct/mongo-odm";

// Basic
await MongoODM.connect();

// With Connection URI
await MongoODM.connect("mongodb://localhost:27017/my-app");

// With Custom Models Folder
await MongoODM.connect("mongodb://localhost:27017/my-app", path.join(__dirname, "path/to/models"));
```

### Creating a Model

Here is an example Model that can be found in `/models/User.js`
```js
import MongoODM from "@cameronct/mongo-odm";

const User = new MongoODM.Model("users", [
    { name: "name", type: MongoODM.FieldTypes.String, required: true },
    { name: "email", type: MongoODM.FieldTypes.String, required: true },
    { name: "password", type: MongoODM.FieldTypes.String, required: true },
    { name: "type", type: MongoODM.FieldTypes.String, default: "basic" },
], [ 
    { name: "uniqueEmail", fields: { email: "text" } },
])
```

### Using a Model

Some examples, most native functionality from MongoDB will work here.
```js
import User from "./models/User";

await User.count({ email: "@example.com" });
await User.findOne({ email: "@example.com" });
await User.findOneOrCreate({ email: "@example.com" }, { name: "test", email: "@example.com" });
await User.findOneAndUpdate({ email: "@example.com" }, { name: "test", email: "new@example.com" }, "$set");
await User.insertOne({ name: "test", email: "@example.com" });
await User.updateOne({ email: "@example.com" }, { name: "test", email: "new@example.com" }, "$set");
await User.removeOne({ email: "@example.com" });
```

### Using QueryBuilder

This is **not** a recommend way of doing my library, as you're essentially doing the same thing as using the native MongoDB driver. However, you can use the QueryBuilder class directly to interact with the Database which will skip the entire Model/Schematic aspect of the library.

The only difference is you will specify the collection name manually, like in the example it's doing the queries in the `users` collection.

```js
import MongoODM from "@cameronct/mongo-odm";

const queryBuilder = new MongoODM.QueryBuilder();

await queryBuilder.count("users", { email: "@example.com" });
await queryBuilder.findOne("users", { email: "@example.com" });
await queryBuilder.findOneOrCreate("users", { email: "@example.com" }, { name: "test", email: "@example.com" });
await queryBuilder.findOneAndUpdate("users", { email: "@example.com" }, { name: "test", email: "new@example.com" }, "$set");
await queryBuilder.insertOne("users", { name: "test", email: "@example.com" });
await queryBuilder.updateOne("users", { email: "@example.com" }, { name: "test", email: "new@example.com" }, "$set");
await queryBuilder.removeOne("users", { email: "@example.com" });
```

### Documents
When a document is inserted into the collection, we specifically add `createdAt` field for new documents using `insertOne`, `insertMany` and `findOneOrCreate`. 
We also add an `updatedAt` field for new documents using `updateOne`, `updateMany` and `findOneAndUpdate`

These parameters **cannot** be overridden inside of the Model. I may change this functionality later, but for now it is what it is.

## Tests

Inserting 50,000 documents using `insertMany`. Code for generating documents is below.
```js
function generateTestData() {
    const documents = Array.from({ length: 50000 }, (_, index) => ({
        name: `Student ${index}`,
        age: Math.floor(Math.random() * 100),
    }));

    return documents;
}
```

Results:
```
MongoDB Native Test completed in 297 milliseconds
Mongoose Test completed in 1883 milliseconds
MongoODM Test completed in 313 milliseconds
```
