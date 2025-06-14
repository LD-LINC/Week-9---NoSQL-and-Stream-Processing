# Lesson 1: Introduction to MongoDB

---


## 2.1 What is MongoDB?

MongoDB is a leading **NoSQL database**, specifically a **document-oriented database**. It's open-source and cross-platform, designed for high performance, high availability, and automatic scaling. Unlike traditional relational databases, MongoDB stores data in flexible, **JSON-like documents**. This flexibility is a key advantage for modern, agile development.

### Why NoSQL? Why MongoDB?
Traditional relational databases (SQL) have a **rigid schema** and often face **scalability challenges**, primarily relying on vertical scaling (upgrading hardware). Queries can involve complex **joins** for denormalized data, impacting performance.

MongoDB addresses these limitations:
* **Flexible Schema:** Documents in a collection don't need a predefined structure, allowing for rapid iteration and adaptation to changing data requirements.
* **Horizontal Scalability (Sharding):** Designed to distribute data across multiple servers, enabling easy scaling out.
* **High Performance:** The document model often leads to fewer joins, resulting in faster read operations, especially for embedded data.
* **Intuitive for Developers:** Working with JSON-like documents aligns well with modern programming languages and web development paradigms, making it easier for developers.

### MongoDB Architecture & Ecosystem
Understanding MongoDB's core components is essential:
<p align="center">
  <img src="L1_Assests/MongoDB Architecture & Ecosystem - visual selection.png" alt="Core Concept" width="60%" />
</p>

* **Mongod:** The primary database process that handles data requests, manages storage, and performs core database operations. This is the server.
* **Mongos:** In a **sharded cluster**, `mongos` acts as a query router, directing client queries to the appropriate data shards.
* **Mongo:** The interactive JavaScript shell used by developers and administrators to interact directly with the database, execute commands, and perform queries.

Beyond these, MongoDB has a rich ecosystem:
* **MongoDB Compass:** A graphical user interface (GUI) for visually exploring data, executing queries, and optimizing performance.
* **MongoDB Atlas:** MongoDB's fully managed cloud database service, simplifying deployment, scaling, and maintenance.
* **Drivers:** Official drivers for most popular programming languages (Python, Node.js, Java, C#, etc.) enable applications to seamlessly connect to MongoDB.

### BSON: Binary JSON
While MongoDB documents are JSON-like, internally they are stored in **BSON (Binary JSON)**. BSON is a binary-encoded serialization of JSON-like documents. It's preferred over plain JSON because it's:
* **Lightweight:** Designed for efficient space utilization.
* **Traversable:** Allows for quick parsing and extraction of specific fields without reading the entire document.
* **Efficient:** Offers faster serialization and deserialization compared to text-based JSON.
BSON also extends JSON by supporting more data types, such as `Date`, `ObjectID`, and `BinData`, which are crucial for robust database operations.

### Documents and Collections
These are the fundamental building blocks of data in MongoDB:
* **Documents:** The basic unit of data, structured as **JSON-like key-value pairs**. They are **schema-less**, meaning fields can vary between documents in the same collection. Documents can contain embedded documents (nested objects) and arrays, enabling rich, hierarchical data structures.
* **Collections:** Groups of documents. They are analogous to tables in relational databases but do not enforce a rigid schema. All documents within a collection typically serve a similar purpose (e.g., a `users` collection contains user documents).

---

## 2.2 CRUD Operations in MongoDB

### Connecting to MongoDB (via Mongo Shell)
Before performing operations, connect to your MongoDB instance:
* Open your terminal and type `mongo` to open the shell.
* Use `show dbs` to list existing databases.
* Use `use <database_name>` to switch to or implicitly create a database.
* Type `db` to confirm the current database.

### Create (Insert) Operations
Inserting new documents into a collection:
* **`db.<collection_name>.insertOne(<document>)`**: Inserts a single document.
    * *Example:* `db.products.insertOne({ name: "Laptop", price: 1200, category: "Electronics" })`
* **`db.<collection_name>.insertMany([<document1>, <document2>, ...])`**: Inserts multiple documents as an array. This is more efficient for bulk operations.
    * *Example:* `db.products.insertMany([{ name: "Mouse", price: 25 }, { name: "Keyboard", price: 75 }])`
    * MongoDB automatically adds a unique `_id` (of type `ObjectID`) if not provided.

### Read (Find) Operations - Basics
Retrieving documents from a collection:
* **`db.<collection_name>.find()`**: Returns all documents in the collection. (Use with caution on large collections).
* **`db.<collection_name>.find(<query_filter>)`**: Returns documents matching the specified criteria.
    * *Example:* `db.products.find({ category: "Electronics" })`
* **`db.<collection_name>.findOne(<query_filter>)`**: Returns the first document that matches the query filter.
    * *Example:* `db.products.findOne({ name: "Laptop" })`

### Query Operators and Filters - Comparison
Used for matching document values against specific conditions:
* **`$eq`**: Equal to (implicit when you provide a field directly, e.g., `{ price: 100 }`).
* **`$ne`**: Not equal to.
    * *Example:* `db.products.find({ category: { $ne: "Books" } })`
* **`$gt`**: Greater than.
* **`$gte`**: Greater than or equal to.
* **`$lt`**: Less than.
* **`$lte`**: Less than or equal to.
    * *Example:* `db.products.find({ price: { $gt: 100 } })`
* **`$in`**: Matches any value in a specified array.
    * *Example:* `db.products.find({ category: { $in: ["Electronics", "Clothing"] } })`
* **`$nin`**: Matches none of the values in a specified array.

### Query Operators and Filters - Logical
Combining multiple query expressions:
* **`$and`**: Joins query clauses with a logical AND. Selects documents that satisfy all expressions. (Implicit when multiple fields are at the top level of a query).
    * *Example:* `db.products.find({ $and: [{ category: "Electronics" }, { price: { $lt: 500 } }] })`
* **`$or`**: Joins query clauses with a logical OR. Selects documents that satisfy at least one expression.
    * *Example:* `db.products.find({ $or: [{ category: "Books" }, { price: { $gt: 1000 } }] })`
* **`$not`**: Inverts the effect of a query expression.
    * *Example:* `db.products.find({ price: { $not: { $gt: 100 } } })` (finds prices <= 100)
* **`$nor`**: Joins query clauses with a logical NOR. Selects documents that fail to match all expressions.

### Query Operators and Filters - Element & Evaluation
* **Element Operators:**
    * **`$exists`**: Matches documents that have (or don't have) the specified field.
        * *Example:* `db.products.find({ description: { $exists: true } })`
    * **`$type`**: Selects documents where the value of a field is of a specified BSON type.
* **Evaluation Operators:**
    * **`$regex`**: Selects documents where values match a regular expression.
        * *Example:* `db.products.find({ name: { $regex: /^L/ } })`
    * **`$text`**: Performs full-text search (requires a text index).

### Read Operations - Projection, Sort, Skip, Limit
Refining read results:
* **Projection (`.find({}, { field: 1, _id: 0 })`)**: Specifies which fields to include (`1`) or exclude (`0`) from the results. By default, `_id` is always included.
    * *Example:* `db.products.find({}, { name: 1, price: 1, _id: 0 })`
* **Sort (`.sort({ field: 1/-1 })`)**: Orders the results. `1` for ascending, `-1` for descending.
    * *Example:* `db.products.find().sort({ price: -1 })`
* **Skip (`.skip(n)`)**: Skips a specified number of documents for pagination.
* **Limit (`.limit(n)`)**: Restricts the number of documents returned.
* **Chaining Methods**: These methods can be chained together (e.g., `db.collection.find().sort().skip().limit()`).

### Update Operations
Modifying existing documents:
* **`db.<collection_name>.updateOne(<filter>, <update_document>)`**: Updates a single document matching the filter.
* **`db.<collection_name>.updateMany(<filter>, <update_document>)`**: Updates all documents matching the filter.
* **Common Update Operators:**
    * **`$set`**: Sets the value of a field. If the field doesn't exist, it adds it.
        * *Example:* `db.products.updateOne({ name: "Laptop" }, { $set: { price: 1250 } })`
    * **`$inc`**: Increments/decrements a numeric field.
        * *Example:* `db.products.updateOne({ name: "Mouse" }, { $inc: { quantity: 1 } })`
    * **`$unset`**: Removes a field from a document.

### Delete Operations
Removing documents or collections:
* **`db.<collection_name>.deleteOne(<filter>)`**: Deletes a single document matching the filter.
* **`db.<collection_name>.deleteMany(<filter>)`**: Deletes all documents matching the filter. **Caution:** `db.collection.deleteMany({})` deletes all documents!
* **`db.<collection_name>.drop()`**: Deletes the entire collection, including all documents and indexes. This is irreversible.

---

## 2.3 Indexing and Aggregation Framework

### Performance Tuning with Indexes
An **index** is a special data structure that stores a small, ordered portion of a collection's data, along with references to the original documents.
* **Why use Indexes?**
    * **Speed up queries:** Reduces the number of documents MongoDB has to scan to find matching data.
    * **Support efficient sorts:** Queries with sort operations can use indexes if the sort order matches the index order.
    * **Enforce uniqueness:** Unique indexes ensure no two documents have the same value for a specified field.
* **Trade-offs:** Indexes consume storage space and can slow down write operations (inserts, updates, deletes) because the index also needs to be updated.

### Creating and Managing Indexes
* **Creating Indexes:**
    * **`db.<collection_name>.createIndex({ <field>: 1/-1 })`**: `1` for ascending, `-1` for descending.
        * *Example (single field):* `db.products.createIndex({ category: 1 })`
    * **Compound indexes**: Indexes on multiple fields. Order matters for query optimization.
        * *Example:* `db.products.createIndex({ category: 1, price: -1 })`
    * **Unique indexes**: Ensures unique values for a field.
        * *Example:* `db.users.createIndex({ email: 1 }, { unique: true })`
* **Viewing Indexes:** `db.<collection_name>.getIndexes()`
* **Dropping Indexes:** `db.<collection_name>.dropIndex("<index_name>")`

### Understanding the Aggregation Framework
**Aggregation** processes data records and returns computed results. It's used to group values, perform calculations (sums, averages, counts), and transform documents.
* **The Aggregation Pipeline:** A series of data processing **stages**. Documents flow through these stages, where each stage performs an operation (e.g., filtering, grouping, reshaping) and passes its output to the next stage. This allows for powerful and flexible data transformations.

### Aggregation Pipeline Stages - `$match`
* **`$match`**: Filters documents based on specified conditions, similar to a `find()` query.
* **Best Practice**: Place `$match` early in the pipeline to reduce the number of documents processed by subsequent stages, significantly improving performance.
    * *Example:* `db.products.aggregate([{ $match: { category: "Electronics" } }])`

### Aggregation Pipeline Stages - `$group`
* **`$group`**: Groups documents by a specified `_id` expression (the grouping key) and applies **accumulator expressions** to compute results for each group.
* **Common Accumulators:**
    * `$sum`: Calculates a sum.
    * `$avg`: Calculates an average.
    * `$min`, `$max`: Returns minimum/maximum value.
    * `$first`, `$last`: Returns first/last document value in group.
    * *Example:*
        ```javascript
        db.products.aggregate([
            { $group: {
                _id: "$category",
                totalProducts: { $sum: 1 },
                averagePrice: { $avg: "$price" }
            }}
        ])
        ```

### Aggregation Pipeline Stages - `$project`
* **`$project`**: Reshapes each document in the stream by selecting, renaming, adding, or removing fields. It's more powerful than `find()` projection as it allows for calculated fields.
    * *Example:*
        ```javascript
        db.products.aggregate([
            { $project: {
                _id: 0,
                productName: "$name",
                unitPrice: "$price",
                taxedPrice: { $multiply: ["$price", 1.10] }
            }}
        ])
        ```

### Aggregation Pipeline Stages - Other Useful Stages
* **`$sort`**: Sorts documents by a field.
* **`$limit`**: Passes a specified number of documents.
* **`$skip`**: Skips a specified number of documents.
* **`$unwind`**: Deconstructs an array field from the input documents, outputting a document for each element in the array. Very useful for processing data within arrays.
* **`$lookup`**: Performs a left outer join to an unsharded collection in the same database, allowing you to include related data from another collection.

### Aggregation Pipeline Example
**Scenario:** Find the top 3 most expensive products in each category.
```javascript
db.products.aggregate([
    { $sort: { category: 1, price: -1 } }, // Sort by category then price descending
    { $group: {
        _id: "$category",
        topProducts: { $push: { name: "$name", price: "$price" } } // Push all products into an array
    }},
    { $project: {
        _id: 0,
        category: "$_id",
        top3Products: { $slice: ["$topProducts", 3] } // Get the first 3 (most expensive)
    }}
])
```
This example shows how stages are chained: sort first, then group and collect, then project and slice the array.

---

## 2.4 MongoDB Schema Design

### Schema Design in NoSQL
While MongoDB is "schema-less" at the database level, a well-thought-out **logical schema** is crucial for performance, scalability, and maintainability.
* **NoSQL (MongoDB) vs. Relational:**
    * **Relational:** Focuses on normalization, rigid tables, and joins.
    * **MongoDB:** Flexible documents; denormalization (embedding) is often preferred to optimize for read performance and minimize joins.
* **Key Design Principles:**
    * **Data access patterns are paramount:** Design your schema based on how your application will query and use the data.
    * **Balance reads vs. writes:** Embedding often optimizes reads but can impact writes, and vice-versa for referencing.

### Embedding vs. Referencing
The core decision in MongoDB schema design:
* **Embedding (Denormalization):**
    * **Concept:** Store related data directly within a single document (nested).
    * **Pros:** Faster reads (single query), atomic updates for the document, reduced joins, single object access in application.
    * **Cons:** Document size limits (16MB), potential data duplication, complex updates for large embedded arrays.
    * **When to use:** One-to-one or one-to-few relationships, data that is rarely accessed independently, data that changes together.
* **Referencing (Normalization):**
    * **Concept:** Store related data in separate documents/collections and link them using `_id` references, similar to foreign keys.
    * **Pros:** Flexible relationships, avoids document size limits, reduces data duplication, simpler updates to referenced documents.
    * **Cons:** Requires multiple queries (`$lookup` for joins), non-atomic operations across documents.
    * **When to use:** One-to-many or many-to-many relationships, frequently updated data, large or unbounded embedded arrays.

