# 🍃 MongoDB Commands: Basic to Advanced

A comprehensive reference guide covering almost everything you need in MongoDB.

---

## Table of Contents

1. [Connecting to MongoDB](#1-connecting-to-mongodb)
2. [mongosh Shell Commands](#2-mongosh-shell-commands)
3. [Database Operations](#3-database-operations)
4. [Collection Operations](#4-collection-operations)
5. [CRUD Operations](#5-crud-operations)
6. [Query Operators](#6-query-operators)
7. [Update Operators](#7-update-operators)
8. [Sorting, Limiting & Projections](#8-sorting-limiting--projections)
9. [Indexes](#9-indexes)
10. [Aggregation Pipeline](#10-aggregation-pipeline)
11. [Aggregation Operators](#11-aggregation-operators)
12. [Lookup (Joins)](#12-lookup-joins)
13. [Schema Validation](#13-schema-validation)
14. [Transactions](#14-transactions)
15. [Relationships & Data Modeling](#15-relationships--data-modeling)
16. [Text Search](#16-text-search)
17. [Geospatial Queries](#17-geospatial-queries)
18. [GridFS (Large Files)](#18-gridfs-large-files)
19. [Change Streams](#19-change-streams)
20. [User & Role Management](#20-user--role-management)
21. [Backup & Restore](#21-backup--restore)
22. [Performance & Query Analysis](#22-performance--query-analysis)
23. [Replication](#23-replication)
24. [Sharding](#24-sharding)
25. [Atlas Search](#25-atlas-search)
26. [Advanced Features](#26-advanced-features)

---

## 1. Connecting to MongoDB

```bash
# Start mongosh (default localhost:27017)
mongosh

# Connect to specific host and port
mongosh "mongodb://localhost:27017"

# Connect to a specific database
mongosh "mongodb://localhost:27017/mydb"

# Connect with username and password
mongosh "mongodb://username:password@localhost:27017/mydb"

# Connect with authentication database
mongosh "mongodb://username:password@localhost:27017/mydb?authSource=admin"

# Connect to MongoDB Atlas
mongosh "mongodb+srv://username:password@cluster.mongodb.net/mydb"

# Connect with TLS
mongosh "mongodb://localhost:27017/mydb" --tls

# Connect using legacy mongo shell
mongo --host localhost --port 27017 -u username -p password --authenticationDatabase admin
```

---

## 2. mongosh Shell Commands

```javascript
// Help
help                          // General help
db.help()                     // Database methods help
db.collection.help()          // Collection methods help

// Database navigation
show dbs                      // List all databases
show databases                // Same as above
use mydb                      // Switch to (or create) database
db                            // Show current database

// Collections
show collections              // List collections in current DB
show tables                   // Same as above

// Exit
exit
quit()
Ctrl+C

// Load a JS file
load("script.js")

// Enable pretty print
db.employees.find().pretty()

// Run command from shell
mongosh --eval "db.employees.countDocuments()"

// Clear screen
cls

// Check server status
db.serverStatus()

// Run admin commands
db.adminCommand({ listDatabases: 1 })
```

---

## 3. Database Operations

```javascript
// Show current database
db

// Show all databases
show dbs

// Switch to / create a database (created on first write)
use mydb

// Create database by inserting a document
use mydb
db.employees.insertOne({ name: "John" })

// Drop current database
db.dropDatabase()

// Get database stats
db.stats()

// Get database name
db.getName()

// List all collections
db.getCollectionNames()

// Run a raw command
db.runCommand({ dbStats: 1 })

// Copy database (deprecated in newer versions, use mongodump instead)
db.adminCommand({ copydb: 1, fromdb: "sourcedb", todb: "targetdb" })

// Database size
db.stats().dataSize
```

---

## 4. Collection Operations

```javascript
// Create collection explicitly
db.createCollection("employees")

// Create collection with options
db.createCollection("logs", {
  capped: true,           // Fixed size, auto-deletes oldest
  size: 10485760,         // Max size in bytes (10MB)
  max: 1000               // Max number of documents
})

// Create collection with schema validation
db.createCollection("employees", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["name", "email"],
      properties: {
        name:  { bsonType: "string" },
        email: { bsonType: "string" }
      }
    }
  }
})

// Rename collection
db.employees.renameCollection("staff")

// Drop collection
db.employees.drop()

// Check if collection exists
db.getCollectionNames().includes("employees")

// Collection stats
db.employees.stats()

// Count documents
db.employees.countDocuments()
db.employees.estimatedDocumentCount()  // Fast approximate count

// Collection info
db.getCollectionInfos({ name: "employees" })
```

---

## 5. CRUD Operations

### INSERT

```javascript
// Insert one document
db.employees.insertOne({
  first_name: "John",
  last_name:  "Doe",
  email:      "john@example.com",
  salary:     75000,
  department: "IT",
  tags:       ["senior", "backend"],
  address: {
    city:    "New York",
    country: "USA"
  },
  hired_at:   new Date("2022-03-15"),
  is_active:  true
})

// Insert many documents
db.employees.insertMany([
  { first_name: "Jane", last_name: "Smith",   salary: 82000, department: "HR" },
  { first_name: "Bob",  last_name: "Johnson", salary: 68000, department: "IT" },
  { first_name: "Alice",last_name: "Williams",salary: 91000, department: "Finance" }
])

// Insert many — continue on error (ordered: false)
db.employees.insertMany([...], { ordered: false })

// Insert with custom _id
db.employees.insertOne({ _id: "emp001", name: "Custom ID Employee" })
```

### FIND (READ)

```javascript
// Find all documents
db.employees.find()

// Find with filter
db.employees.find({ department: "IT" })

// Find one document
db.employees.findOne({ department: "IT" })

// Find by _id
const { ObjectId } = require('mongodb')
db.employees.findOne({ _id: ObjectId("64abc123...") })

// Find with projection (include fields)
db.employees.find({ department: "IT" }, { first_name: 1, salary: 1 })

// Find with projection (exclude fields)
db.employees.find({}, { password: 0, __v: 0 })

// Find nested fields
db.employees.find({ "address.city": "New York" })

// Find in array
db.employees.find({ tags: "senior" })

// Pretty print
db.employees.find().pretty()

// Convert to array
db.employees.find().toArray()

// Count results
db.employees.countDocuments({ department: "IT" })
```

### UPDATE

```javascript
// Update one document
db.employees.updateOne(
  { _id: ObjectId("64abc123...") },
  { $set: { salary: 80000 } }
)

// Update many documents
db.employees.updateMany(
  { department: "IT" },
  { $set: { is_active: true } }
)

// Replace entire document (except _id)
db.employees.replaceOne(
  { _id: ObjectId("64abc123...") },
  { first_name: "New", last_name: "Doc", salary: 50000 }
)

// Upsert (update or insert if not found)
db.employees.updateOne(
  { email: "new@example.com" },
  { $set: { first_name: "New", salary: 60000 } },
  { upsert: true }
)

// findOneAndUpdate (returns document)
db.employees.findOneAndUpdate(
  { _id: ObjectId("64abc123...") },
  { $inc: { salary: 5000 } },
  { returnDocument: "after" }   // Returns updated document
)

// findOneAndReplace
db.employees.findOneAndReplace(
  { email: "john@example.com" },
  { first_name: "John", email: "john@example.com", salary: 90000 },
  { returnDocument: "after" }
)
```

### DELETE

```javascript
// Delete one document
db.employees.deleteOne({ _id: ObjectId("64abc123...") })

// Delete many documents
db.employees.deleteMany({ is_active: false })

// Delete all documents in collection
db.employees.deleteMany({})

// findOneAndDelete (returns deleted document)
db.employees.findOneAndDelete({ salary: { $lt: 30000 } })

// Drop collection (removes collection and all documents)
db.employees.drop()
```

---

## 6. Query Operators

### Comparison Operators

```javascript
// $eq — equal (default)
db.employees.find({ salary: { $eq: 75000 } })

// $ne — not equal
db.employees.find({ department: { $ne: "IT" } })

// $gt, $gte — greater than (or equal)
db.employees.find({ salary: { $gt: 70000 } })
db.employees.find({ salary: { $gte: 70000 } })

// $lt, $lte — less than (or equal)
db.employees.find({ salary: { $lt: 50000 } })
db.employees.find({ salary: { $lte: 50000 } })

// $in — matches any value in array
db.employees.find({ department: { $in: ["IT", "HR", "Finance"] } })

// $nin — not in array
db.employees.find({ department: { $nin: ["IT", "HR"] } })
```

### Logical Operators

```javascript
// $and
db.employees.find({
  $and: [
    { department: "IT" },
    { salary: { $gt: 70000 } }
  ]
})

// $or
db.employees.find({
  $or: [
    { department: "IT" },
    { salary: { $gt: 90000 } }
  ]
})

// $nor (neither condition true)
db.employees.find({
  $nor: [
    { department: "IT" },
    { is_active: false }
  ]
})

// $not
db.employees.find({ salary: { $not: { $gt: 80000 } } })

// Implicit AND (shorthand)
db.employees.find({ department: "IT", salary: { $gt: 70000 } })
```

### Element Operators

```javascript
// $exists — field exists
db.employees.find({ phone: { $exists: true } })
db.employees.find({ phone: { $exists: false } })

// $type — field is of type
db.employees.find({ salary: { $type: "number" } })
db.employees.find({ hired_at: { $type: "date" } })
db.employees.find({ name: { $type: ["string", "null"] } })
```

### Array Operators

```javascript
// $all — array contains all elements
db.employees.find({ tags: { $all: ["senior", "backend"] } })

// $elemMatch — at least one array element matches all conditions
db.employees.find({
  scores: { $elemMatch: { $gte: 80, $lt: 100 } }
})

// $size — array has exact number of elements
db.employees.find({ tags: { $size: 3 } })
```

### Evaluation Operators

```javascript
// $regex — regular expression
db.employees.find({ email: { $regex: "@gmail\\.com$" } })
db.employees.find({ first_name: { $regex: /^jo/i } })  // Case-insensitive

// $expr — use aggregation expressions in queries
db.employees.find({ $expr: { $gt: ["$salary", "$target_salary"] } })

// $where — JavaScript expression (slow, avoid in production)
db.employees.find({ $where: "this.salary > 70000" })

// $mod — modulo
db.employees.find({ id: { $mod: [2, 0] } })  // Even IDs

// $jsonSchema
db.employees.find({
  $jsonSchema: {
    required: ["email"],
    properties: { salary: { minimum: 0 } }
  }
})
```

---

## 7. Update Operators

### Field Operators

```javascript
// $set — set field value
db.employees.updateOne({ _id: id }, { $set: { salary: 80000 } })

// $unset — remove a field
db.employees.updateOne({ _id: id }, { $unset: { phone: "" } })

// $rename — rename a field
db.employees.updateMany({}, { $rename: { "fname": "first_name" } })

// $inc — increment/decrement a number
db.employees.updateOne({ _id: id }, { $inc: { salary: 5000 } })
db.employees.updateOne({ _id: id }, { $inc: { salary: -2000 } })

// $mul — multiply a field
db.employees.updateOne({ _id: id }, { $mul: { salary: 1.10 } })

// $min — update only if new value is less than current
db.employees.updateOne({ _id: id }, { $min: { salary: 50000 } })

// $max — update only if new value is greater than current
db.employees.updateOne({ _id: id }, { $max: { salary: 100000 } })

// $setOnInsert — set fields only during upsert insert
db.employees.updateOne(
  { email: "new@example.com" },
  {
    $set: { salary: 60000 },
    $setOnInsert: { created_at: new Date() }
  },
  { upsert: true }
)

// $currentDate — set to current date
db.employees.updateOne({ _id: id }, { $currentDate: { updated_at: true } })
```

### Array Update Operators

```javascript
// $push — append to array
db.employees.updateOne({ _id: id }, { $push: { tags: "manager" } })

// $push with $each — push multiple elements
db.employees.updateOne({ _id: id }, {
  $push: { tags: { $each: ["lead", "senior"] } }
})

// $push with $sort and $slice — sorted, limited array
db.employees.updateOne({ _id: id }, {
  $push: {
    scores: {
      $each: [95, 88],
      $sort: -1,
      $slice: 5   // Keep only top 5
    }
  }
})

// $addToSet — add to array only if not already present
db.employees.updateOne({ _id: id }, { $addToSet: { tags: "senior" } })

// $pop — remove first (-1) or last (1) element
db.employees.updateOne({ _id: id }, { $pop: { tags: 1 } })

// $pull — remove elements matching condition
db.employees.updateOne({ _id: id }, { $pull: { tags: "junior" } })
db.employees.updateOne({ _id: id }, { $pull: { scores: { $lt: 50 } } })

// $pullAll — remove all matching elements
db.employees.updateOne({ _id: id }, { $pullAll: { tags: ["junior", "intern"] } })

// $ — positional operator (update first matching array element)
db.employees.updateOne(
  { _id: id, "scores.type": "math" },
  { $set: { "scores.$.value": 95 } }
)

// $[] — update all array elements
db.employees.updateOne({ _id: id }, { $inc: { "scores.$[].value": 5 } })

// $[identifier] — filtered positional operator
db.employees.updateOne(
  { _id: id },
  { $set: { "scores.$[elem].value": 100 } },
  { arrayFilters: [{ "elem.type": "math" }] }
)
```

---

## 8. Sorting, Limiting & Projections

```javascript
// Sort ascending (1) / descending (-1)
db.employees.find().sort({ salary: -1 })
db.employees.find().sort({ department: 1, salary: -1 })

// Limit results
db.employees.find().limit(10)

// Skip results (for pagination)
db.employees.find().skip(20).limit(10)  // Page 3

// Projection — include fields (1)
db.employees.find({}, { first_name: 1, salary: 1 })

// Projection — exclude fields (0)
db.employees.find({}, { password: 0 })

// _id is included by default; exclude it
db.employees.find({}, { _id: 0, first_name: 1, salary: 1 })

// Project nested fields
db.employees.find({}, { "address.city": 1 })

// Project array slice
db.employees.find({}, { tags: { $slice: 2 } })     // First 2 elements
db.employees.find({}, { tags: { $slice: -2 } })    // Last 2 elements
db.employees.find({}, { tags: { $slice: [1, 2] } }) // Skip 1, take 2

// Project first matching array element
db.employees.find(
  { scores: { $elemMatch: { $gte: 90 } } },
  { "scores.$": 1 }
)

// Chain everything
db.employees
  .find({ department: "IT" }, { first_name: 1, salary: 1, _id: 0 })
  .sort({ salary: -1 })
  .skip(0)
  .limit(5)

// Cursor methods
const cursor = db.employees.find()
cursor.hasNext()
cursor.next()
cursor.forEach(doc => printjson(doc))
cursor.toArray()
cursor.count()    // Deprecated, use countDocuments
```

---

## 9. Indexes

```javascript
// Create single field index
db.employees.createIndex({ email: 1 })

// Create descending index
db.employees.createIndex({ salary: -1 })

// Create unique index
db.employees.createIndex({ email: 1 }, { unique: true })

// Create compound index
db.employees.createIndex({ department: 1, salary: -1 })

// Create text index (for full-text search)
db.employees.createIndex({ first_name: "text", last_name: "text" })

// Create text index on all string fields
db.employees.createIndex({ "$**": "text" })

// Create geospatial index
db.locations.createIndex({ coordinates: "2dsphere" })

// Create hashed index (for sharding)
db.employees.createIndex({ _id: "hashed" })

// Wildcard index (indexes all fields)
db.employees.createIndex({ "$**": 1 })

// Partial index (only indexes documents matching filter)
db.employees.createIndex(
  { salary: 1 },
  { partialFilterExpression: { is_active: true } }
)

// Sparse index (skips documents missing the field)
db.employees.createIndex({ phone: 1 }, { sparse: true })

// TTL index (auto-delete after N seconds)
db.sessions.createIndex({ created_at: 1 }, { expireAfterSeconds: 3600 })

// Create index in background (non-blocking, older versions)
db.employees.createIndex({ salary: 1 }, { background: true })

// Index with name
db.employees.createIndex({ department: 1 }, { name: "idx_department" })

// List all indexes
db.employees.getIndexes()

// Get index stats
db.employees.aggregate([{ $indexStats: {} }])

// Drop one index
db.employees.dropIndex("idx_department")
db.employees.dropIndex({ email: 1 })

// Drop all indexes (except _id)
db.employees.dropIndexes()

// Hide index (disable without dropping — MongoDB 4.4+)
db.employees.hideIndex({ email: 1 })
db.employees.unhideIndex({ email: 1 })

// Rebuild indexes
db.employees.reIndex()
```

---

## 10. Aggregation Pipeline

The aggregation pipeline processes documents through a sequence of stages.

```javascript
// Basic pipeline structure
db.employees.aggregate([
  { $stage1: { ... } },
  { $stage2: { ... } },
  { $stage3: { ... } }
])

// $match — filter documents (like find)
db.employees.aggregate([
  { $match: { department: "IT", salary: { $gt: 60000 } } }
])

// $project — reshape documents
db.employees.aggregate([
  {
    $project: {
      full_name: { $concat: ["$first_name", " ", "$last_name"] },
      salary: 1,
      annual_salary: { $multiply: ["$salary", 12] }
    }
  }
])

// $group — group and aggregate
db.employees.aggregate([
  {
    $group: {
      _id: "$department",
      headcount:   { $sum: 1 },
      total_salary:{ $sum: "$salary" },
      avg_salary:  { $avg: "$salary" },
      max_salary:  { $max: "$salary" },
      min_salary:  { $min: "$salary" }
    }
  }
])

// $sort
db.employees.aggregate([
  { $group: { _id: "$department", avg_salary: { $avg: "$salary" } } },
  { $sort: { avg_salary: -1 } }
])

// $limit and $skip
db.employees.aggregate([
  { $match: { is_active: true } },
  { $sort: { salary: -1 } },
  { $skip: 0 },
  { $limit: 10 }
])

// $unwind — deconstruct array to multiple documents
db.employees.aggregate([
  { $unwind: "$tags" },
  { $group: { _id: "$tags", count: { $sum: 1 } } }
])

// $unwind with options
db.employees.aggregate([
  {
    $unwind: {
      path: "$tags",
      includeArrayIndex: "tagIndex",
      preserveNullAndEmptyArrays: true
    }
  }
])

// $addFields — add new fields
db.employees.aggregate([
  {
    $addFields: {
      annual_salary: { $multiply: ["$salary", 12] },
      name_upper: { $toUpper: "$first_name" }
    }
  }
])

// $replaceRoot — replace the document root
db.employees.aggregate([
  { $replaceRoot: { newRoot: "$address" } }
])

// $count — count pipeline documents
db.employees.aggregate([
  { $match: { department: "IT" } },
  { $count: "it_employees" }
])

// $facet — multiple pipelines in one aggregation
db.employees.aggregate([
  {
    $facet: {
      by_department: [
        { $group: { _id: "$department", count: { $sum: 1 } } }
      ],
      salary_ranges: [
        {
          $bucket: {
            groupBy: "$salary",
            boundaries: [0, 50000, 75000, 100000, 200000],
            default: "Other"
          }
        }
      ]
    }
  }
])

// $bucket — group into ranges
db.employees.aggregate([
  {
    $bucket: {
      groupBy: "$salary",
      boundaries: [0, 40000, 70000, 100000],
      default: "Above 100K",
      output: {
        count: { $sum: 1 },
        avg_salary: { $avg: "$salary" }
      }
    }
  }
])

// $bucketAuto — auto-range grouping
db.employees.aggregate([
  {
    $bucketAuto: {
      groupBy: "$salary",
      buckets: 4,
      output: { count: { $sum: 1 }, avg: { $avg: "$salary" } }
    }
  }
])

// $sample — random sample of documents
db.employees.aggregate([{ $sample: { size: 5 } }])

// $out — write results to a new collection
db.employees.aggregate([
  { $match: { salary: { $gt: 80000 } } },
  { $out: "high_earners" }
])

// $merge — merge results into an existing collection
db.employees.aggregate([
  { $group: { _id: "$department", count: { $sum: 1 } } },
  {
    $merge: {
      into: "dept_stats",
      on: "_id",
      whenMatched: "merge",
      whenNotMatched: "insert"
    }
  }
])

// Full pipeline example
db.employees.aggregate([
  { $match:   { is_active: true } },
  { $lookup: {
      from: "departments",
      localField: "department_id",
      foreignField: "_id",
      as: "dept_info"
  }},
  { $unwind: "$dept_info" },
  { $group: {
      _id: "$dept_info.name",
      headcount: { $sum: 1 },
      avg_salary: { $avg: "$salary" }
  }},
  { $sort:  { avg_salary: -1 } },
  { $limit: 5 },
  { $project: {
      department: "$_id",
      headcount: 1,
      avg_salary: { $round: ["$avg_salary", 2] },
      _id: 0
  }}
])
```

---

## 11. Aggregation Operators

### Arithmetic

```javascript
{ $add:      ["$price", "$tax"] }
{ $subtract: ["$salary", "$deductions"] }
{ $multiply: ["$salary", 12] }
{ $divide:   ["$total", "$count"] }
{ $mod:      ["$value", 2] }
{ $abs:      "$value" }
{ $ceil:     "$value" }
{ $floor:    "$value" }
{ $round:    ["$value", 2] }
{ $sqrt:     "$value" }
{ $pow:      ["$base", "$exp"] }
{ $log:      ["$value", 10] }
{ $trunc:    "$value" }
```

### String

```javascript
{ $concat:      ["$first_name", " ", "$last_name"] }
{ $toUpper:     "$name" }
{ $toLower:     "$name" }
{ $trim:        { input: "$name" } }
{ $ltrim:       { input: "$name" } }
{ $rtrim:       { input: "$name" } }
{ $substr:      ["$name", 0, 5] }
{ $strLenCP:    "$name" }
{ $split:       ["$email", "@"] }
{ $indexOfCP:   ["$email", "@"] }
{ $replaceOne:  { input: "$text", find: "foo", replacement: "bar" } }
{ $replaceAll:  { input: "$text", find: "foo", replacement: "bar" } }
{ $regexMatch:  { input: "$email", regex: /gmail/ } }
{ $regexFind:   { input: "$text", regex: /\d+/ } }
{ $regexFindAll:{ input: "$text", regex: /\w+/ } }
```

### Date

```javascript
{ $year:        "$hired_at" }
{ $month:       "$hired_at" }
{ $dayOfMonth:  "$hired_at" }
{ $dayOfWeek:   "$hired_at" }
{ $hour:        "$hired_at" }
{ $minute:      "$hired_at" }
{ $second:      "$hired_at" }
{ $dateToString:{ format: "%Y-%m-%d", date: "$hired_at" } }
{ $dateToParts: { date: "$hired_at" } }
{ $dateFromParts: { year: 2024, month: 3, day: 15 } }
{ $dateAdd:     { startDate: "$hired_at", unit: "month", amount: 6 } }
{ $dateDiff:    { startDate: "$start", endDate: "$end", unit: "day" } }
```

### Array

```javascript
{ $size:        "$tags" }
{ $first:       "$scores" }
{ $last:        "$scores" }
{ $arrayElemAt: ["$scores", 0] }
{ $in:          ["IT", "$departments"] }
{ $concatArrays:["$arr1", "$arr2"] }
{ $slice:       ["$tags", 2] }
{ $filter: {
    input: "$scores",
    as:    "score",
    cond:  { $gte: ["$$score", 80] }
  }
}
{ $map: {
    input: "$tags",
    as:    "tag",
    in:    { $toUpper: "$$tag" }
  }
}
{ $reduce: {
    input:       "$scores",
    initialValue:0,
    in: { $add: ["$$value", "$$this"] }
  }
}
{ $zip: { inputs: ["$arr1", "$arr2"] } }
{ $indexOfArray: ["$tags", "senior"] }
{ $reverseArray: "$scores" }
{ $sortArray:    { input: "$scores", sortBy: { score: -1 } } }
{ $setUnion:     ["$a", "$b"] }
{ $setIntersection: ["$a", "$b"] }
{ $setDifference:["$a", "$b"] }
```

### Conditional

```javascript
// $cond (if/then/else)
{ $cond: { if: { $gte: ["$salary", 80000] }, then: "High", else: "Low" } }
{ $cond: [{ $gte: ["$salary", 80000] }, "High", "Low"] }

// $ifNull (fallback for null)
{ $ifNull: ["$phone", "N/A"] }

// $switch (case/when)
{
  $switch: {
    branches: [
      { case: { $lt: ["$salary", 40000] }, then: "Entry" },
      { case: { $lt: ["$salary", 70000] }, then: "Mid" },
      { case: { $lt: ["$salary", 100000] }, then: "Senior" }
    ],
    default: "Executive"
  }
}
```

### Type Conversion

```javascript
{ $toString:  "$salary" }
{ $toInt:     "$string_salary" }
{ $toLong:    "$value" }
{ $toDouble:  "$value" }
{ $toDecimal: "$value" }
{ $toBool:    "$value" }
{ $toDate:    "$date_string" }
{ $toObjectId:"$id_string" }
{ $convert: { input: "$value", to: "int", onError: 0, onNull: 0 } }
{ $type:      "$value" }           // Returns type as string
```

### Accumulators (in $group)

```javascript
{ $sum:       "$salary" }
{ $avg:       "$salary" }
{ $min:       "$salary" }
{ $max:       "$salary" }
{ $count:     {} }
{ $first:     "$name" }
{ $last:      "$name" }
{ $push:      "$name" }            // Array of all values
{ $addToSet:  "$tag" }             // Array of unique values
{ $stdDevPop: "$salary" }
{ $stdDevSamp:"$salary" }
{ $mergeObjects: "$details" }
{ $accumulator: { /* custom JS accumulator */ } }
```

---

## 12. Lookup (Joins)

```javascript
// Basic $lookup (like LEFT JOIN)
db.employees.aggregate([
  {
    $lookup: {
      from:         "departments",     // Collection to join
      localField:   "department_id",   // Field in employees
      foreignField: "_id",             // Field in departments
      as:           "department_info"  // Output array field
    }
  },
  { $unwind: "$department_info" }     // Flatten the array
])

// $lookup with pipeline (correlated subquery)
db.employees.aggregate([
  {
    $lookup: {
      from: "projects",
      let:  { emp_id: "$_id" },
      pipeline: [
        { $match: { $expr: { $eq: ["$lead_id", "$$emp_id"] } } },
        { $project: { name: 1, budget: 1 } }
      ],
      as: "led_projects"
    }
  }
])

// Multiple lookups
db.orders.aggregate([
  {
    $lookup: {
      from: "customers",
      localField: "customer_id",
      foreignField: "_id",
      as: "customer"
    }
  },
  { $unwind: "$customer" },
  {
    $lookup: {
      from: "products",
      localField: "product_id",
      foreignField: "_id",
      as: "product"
    }
  },
  { $unwind: "$product" }
])

// Simulate INNER JOIN (exclude non-matches)
db.employees.aggregate([
  {
    $lookup: {
      from: "departments",
      localField: "department_id",
      foreignField: "_id",
      as: "dept"
    }
  },
  { $match: { dept: { $ne: [] } } }  // Exclude no-match documents
])

// $graphLookup — recursive lookup (org hierarchy, trees)
db.employees.aggregate([
  { $match: { name: "CEO" } },
  {
    $graphLookup: {
      from:             "employees",
      startWith:        "$_id",
      connectFromField: "_id",
      connectToField:   "manager_id",
      as:               "reports",
      depthField:       "level",
      maxDepth:         3
    }
  }
])
```

---

## 13. Schema Validation

```javascript
// Add validation to existing collection
db.runCommand({
  collMod: "employees",
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["first_name", "last_name", "email", "salary"],
      properties: {
        first_name: {
          bsonType: "string",
          description: "Must be a string and is required"
        },
        last_name: {
          bsonType: "string"
        },
        email: {
          bsonType: "string",
          pattern: "^.+@.+\\..+$",
          description: "Must be a valid email"
        },
        salary: {
          bsonType: "number",
          minimum: 0,
          description: "Must be a non-negative number"
        },
        department: {
          bsonType: "string",
          enum: ["IT", "HR", "Finance", "Sales", "Engineering"]
        },
        tags: {
          bsonType: "array",
          items: { bsonType: "string" }
        },
        address: {
          bsonType: "object",
          required: ["city"],
          properties: {
            city:    { bsonType: "string" },
            country: { bsonType: "string" }
          }
        }
      }
    }
  },
  validationLevel:  "moderate",  // "off", "moderate", "strict"
  validationAction: "error"      // "error" or "warn"
})

// View validation rules
db.getCollectionInfos({ name: "employees" })[0].options.validator

// Remove validation
db.runCommand({ collMod: "employees", validator: {} })
```

---

## 14. Transactions

```javascript
// Multi-document ACID transactions (requires Replica Set)
const session = db.getMongo().startSession()

try {
  session.startTransaction({
    readConcern:  { level: "snapshot" },
    writeConcern: { w: "majority" }
  })

  const employeesCol  = session.getDatabase("mydb").employees
  const departmentsCol = session.getDatabase("mydb").departments

  employeesCol.updateOne(
    { _id: ObjectId("...") },
    { $set: { salary: 90000 } },
    { session }
  )

  departmentsCol.updateOne(
    { name: "IT" },
    { $inc: { budget: -15000 } },
    { session }
  )

  session.commitTransaction()
  print("Transaction committed")

} catch (error) {
  session.abortTransaction()
  print("Transaction aborted:", error)

} finally {
  session.endSession()
}

// Transactions with retryable writes
const txnOptions = {
  readPreference: "primary",
  readConcern:    { level: "local" },
  writeConcern:   { w: "majority" }
}

// Using withTransaction helper (auto-retries)
session.withTransaction(async () => {
  // operations here
}, txnOptions)
```

---

## 15. Relationships & Data Modeling

```javascript
// Embedded documents (one-to-one, one-to-few)
db.employees.insertOne({
  name: "John",
  address: {               // Embedded
    street: "123 Main St",
    city: "New York",
    zip: "10001"
  },
  contacts: [              // Embedded array
    { type: "email", value: "john@work.com" },
    { type: "phone", value: "555-1234" }
  ]
})

// Referenced documents (one-to-many, many-to-many)
// departments collection
db.departments.insertOne({ _id: ObjectId("dept001"), name: "IT" })

// employees collection
db.employees.insertOne({
  name: "John",
  department_id: ObjectId("dept001")  // Reference to department
})

// Query with reference
const dept = db.departments.findOne({ name: "IT" })
db.employees.find({ department_id: dept._id })

// Two-way referencing (for many-to-many)
// employees: { _id, name, project_ids: [ObjectId, ...] }
// projects:  { _id, name, member_ids: [ObjectId, ...] }

// Denormalization (duplicate data for read performance)
db.orders.insertOne({
  customer_id: ObjectId("..."),
  customer_name: "John Doe",   // Duplicated for fast reads
  customer_email: "john@example.com",
  items: [
    {
      product_id: ObjectId("..."),
      product_name: "Widget",  // Duplicated
      price: 9.99,
      qty: 2
    }
  ]
})

// Bucket pattern (time-series)
db.sensor_readings.insertOne({
  sensor_id: "sensor_01",
  date: new Date("2024-01-01"),
  readings: [
    { time: new Date("2024-01-01T00:00:00"), value: 22.5 },
    { time: new Date("2024-01-01T00:05:00"), value: 22.7 }
  ],
  count: 2
})

// Subset pattern (keep only recent/relevant data in main doc)
// Full reviews in a separate collection, top 3 in product doc
db.products.insertOne({
  name: "Widget",
  top_reviews: [          // Subset — only top 3
    { rating: 5, text: "Great!" },
    { rating: 4, text: "Good" }
  ],
  avg_rating: 4.5,
  review_count: 234
})
```

---

## 16. Text Search

```javascript
// Create text index
db.articles.createIndex({ title: "text", body: "text" })

// Weighted text index
db.articles.createIndex(
  { title: "text", body: "text" },
  { weights: { title: 10, body: 1 }, name: "article_text_idx" }
)

// Text index on all string fields
db.articles.createIndex({ "$**": "text" })

// Basic text search
db.articles.find({ $text: { $search: "mongodb tutorial" } })

// Phrase search
db.articles.find({ $text: { $search: "\"mongodb tutorial\"" } })

// Exclude term
db.articles.find({ $text: { $search: "mongodb -sql" } })

// Case-insensitive (default) and diacritic-insensitive
db.articles.find({ $text: { $search: "cafe", $diacriticSensitive: false } })

// With relevance score
db.articles.find(
  { $text: { $search: "mongodb performance" } },
  { score: { $meta: "textScore" } }
).sort({ score: { $meta: "textScore" } })

// Text search in aggregation
db.articles.aggregate([
  { $match:   { $text: { $search: "mongodb" } } },
  { $project: { title: 1, score: { $meta: "textScore" } } },
  { $sort:    { score: { $meta: "textScore" } } },
  { $limit:   5 }
])

// Language-specific search
db.articles.find({ $text: { $search: "bonjour", $language: "french" } })
```

---

## 17. Geospatial Queries

```javascript
// Create 2dsphere index (for GeoJSON)
db.locations.createIndex({ coordinates: "2dsphere" })

// Store GeoJSON point
db.locations.insertOne({
  name: "Coffee Shop",
  coordinates: {
    type: "Point",
    coordinates: [-73.9857, 40.7484]   // [longitude, latitude]
  }
})

// Store GeoJSON polygon
db.zones.insertOne({
  name: "Manhattan",
  area: {
    type: "Polygon",
    coordinates: [[[...], [...], [...]]]
  }
})

// Find nearby points ($near — sorted by distance)
db.locations.find({
  coordinates: {
    $near: {
      $geometry: { type: "Point", coordinates: [-73.9857, 40.7484] },
      $maxDistance: 1000,    // Meters
      $minDistance: 0
    }
  }
})

// Find within circle ($geoWithin + $centerSphere)
db.locations.find({
  coordinates: {
    $geoWithin: {
      $centerSphere: [[-73.9857, 40.7484], 1 / 3963.2]  // Radius in miles
    }
  }
})

// Find within a polygon
db.locations.find({
  coordinates: {
    $geoWithin: {
      $geometry: {
        type: "Polygon",
        coordinates: [[[lng1, lat1], [lng2, lat2], [lng3, lat3], [lng1, lat1]]]
      }
    }
  }
})

// Find which zone a point intersects
db.zones.find({
  area: {
    $geoIntersects: {
      $geometry: { type: "Point", coordinates: [-73.9857, 40.7484] }
    }
  }
})

// Aggregation with $geoNear (must be first stage)
db.locations.aggregate([
  {
    $geoNear: {
      near: { type: "Point", coordinates: [-73.9857, 40.7484] },
      distanceField: "distance_meters",
      maxDistance: 2000,
      spherical: true,
      query: { category: "restaurant" }
    }
  },
  { $limit: 10 }
])

// 2d index for flat coordinates (legacy)
db.locations.createIndex({ pos: "2d" })
db.locations.find({ pos: { $near: [40, -73], $maxDistance: 0.5 } })
```

---

## 18. GridFS (Large Files)

```bash
# Store a file
mongofiles -d mydb put /path/to/large_file.mp4

# Retrieve a file
mongofiles -d mydb get large_file.mp4

# List files
mongofiles -d mydb list

# Delete a file
mongofiles -d mydb delete large_file.mp4

# Search files
mongofiles -d mydb search "video"
```

```javascript
// GridFS via mongosh
// Files are stored in fs.files and fs.chunks collections

// List all files
db.fs.files.find()

// Find file metadata
db.fs.files.findOne({ filename: "large_file.mp4" })

// GridFS via Node.js driver
const { GridFSBucket } = require('mongodb')
const bucket = new GridFSBucket(db)

// Upload
const uploadStream = bucket.openUploadStream("myfile.txt")
fs.createReadStream("/path/to/file").pipe(uploadStream)

// Download
const downloadStream = bucket.openDownloadStreamByName("myfile.txt")
downloadStream.pipe(fs.createWriteStream("/path/to/output"))
```

---

## 19. Change Streams

```javascript
// Watch all changes on a collection
const changeStream = db.employees.watch()

changeStream.forEach(change => {
  printjson(change)
})

// Watch specific operations
const changeStream = db.employees.watch([
  { $match: { operationType: { $in: ["insert", "update", "delete"] } } }
])

// Watch with full document on update
const changeStream = db.employees.watch([], {
  fullDocument: "updateLookup"
})

// Watch on a database (all collections)
const changeStream = db.watch()

// Watch on all databases
const changeStream = db.getMongo().watch()

// Resume after interruption (using resumeToken)
let resumeToken
const changeStream = db.employees.watch()

changeStream.forEach(change => {
  resumeToken = change._id
  printjson(change)
})

// Resume from token
const newStream = db.employees.watch([], { resumeAfter: resumeToken })

// Start from specific timestamp
const newStream = db.employees.watch([], {
  startAtOperationTime: new Timestamp(1609459200, 1)
})
```

---

## 20. User & Role Management

```javascript
// Create user
db.createUser({
  user: "alice",
  pwd:  "securepassword",
  roles: [
    { role: "readWrite", db: "mydb" },
    { role: "read",      db: "reporting" }
  ]
})

// Create user with password prompt
db.createUser({ user: "alice", pwd: passwordPrompt(), roles: ["readWrite"] })

// Built-in roles
// Database roles: read, readWrite, dbAdmin, dbOwner, userAdmin
// Admin roles:    readAnyDatabase, readWriteAnyDatabase, userAdminAnyDatabase,
//                 dbAdminAnyDatabase, clusterAdmin, superuser, root

// Create custom role
db.createRole({
  role: "employeeReader",
  privileges: [
    {
      resource: { db: "mydb", collection: "employees" },
      actions:  ["find"]
    }
  ],
  roles: []
})

// Grant role to user
db.grantRolesToUser("alice", [{ role: "dbAdmin", db: "mydb" }])

// Revoke role from user
db.revokeRolesFromUser("alice", [{ role: "dbAdmin", db: "mydb" }])

// Update user password
db.updateUser("alice", { pwd: "newpassword" })

// Update user roles
db.updateUser("alice", {
  roles: [{ role: "read", db: "mydb" }]
})

// Show user info
db.getUser("alice")
db.getUsers()

// Drop user
db.dropUser("alice")

// Drop all users in database
db.dropAllUsers()

// Auth as user
db.auth("alice", "password")
```

---

## 21. Backup & Restore

```bash
# Dump entire MongoDB instance
mongodump --out /backup/dump

# Dump specific database
mongodump --db mydb --out /backup/dump

# Dump specific collection
mongodump --db mydb --collection employees --out /backup/dump

# Dump with query filter
mongodump --db mydb --collection employees \
  --query '{"is_active": true}' --out /backup/dump

# Dump compressed
mongodump --db mydb --gzip --out /backup/dump

# Dump to archive file
mongodump --db mydb --archive=/backup/mydb.archive

# Restore entire dump
mongorestore /backup/dump

# Restore specific database
mongorestore --db mydb /backup/dump/mydb

# Restore specific collection
mongorestore --db mydb --collection employees \
  /backup/dump/mydb/employees.bson

# Restore from archive
mongorestore --archive=/backup/mydb.archive

# Drop target before restoring
mongorestore --drop --db mydb /backup/dump/mydb

# Export to JSON/CSV
mongoexport --db mydb --collection employees --out employees.json
mongoexport --db mydb --collection employees --type csv \
  --fields first_name,last_name,salary --out employees.csv

# Import from JSON
mongoimport --db mydb --collection employees --file employees.json

# Import from CSV
mongoimport --db mydb --collection employees --type csv \
  --headerline --file employees.csv

# Import (drop existing first)
mongoimport --db mydb --collection employees --drop --file employees.json

# Import multiple documents per line (array)
mongoimport --db mydb --collection employees --jsonArray --file employees.json
```

---

## 22. Performance & Query Analysis

```javascript
// Explain query plan
db.employees.find({ department: "IT" }).explain()

// Explain with execution stats
db.employees.find({ department: "IT" }).explain("executionStats")

// Full explain (all plans considered)
db.employees.find({ department: "IT" }).explain("allPlansExecution")

// Explain aggregation
db.employees.aggregate([...]).explain("executionStats")

// Check if index is used (look for IXSCAN vs COLLSCAN)
db.employees.find({ salary: { $gt: 70000 } }).explain("executionStats")

// Hint to force index
db.employees.find({ salary: { $gt: 70000 } }).hint({ salary: 1 })
db.employees.find({ salary: { $gt: 70000 } }).hint("idx_salary")

// Disable index (force collection scan)
db.employees.find({ salary: { $gt: 70000 } }).hint({ $natural: 1 })

// Profile slow operations
db.setProfilingLevel(1, { slowms: 100 })   // Log queries > 100ms
db.setProfilingLevel(2)                    // Log all operations
db.setProfilingLevel(0)                    // Disable profiling

// View profiling level
db.getProfilingStatus()

// Query the profiler log
db.system.profile.find().sort({ ts: -1 }).limit(5).pretty()
db.system.profile.find({ millis: { $gt: 100 } }).pretty()

// Current operations
db.currentOp()
db.currentOp({ active: true, secs_running: { $gt: 5 } })

// Kill an operation
db.killOp(opid)

// Server status
db.serverStatus()
db.serverStatus().connections
db.serverStatus().opcounters

// Collection stats
db.employees.stats()
db.employees.stats().totalIndexSize

// Database stats
db.stats()

// Validate collection (check consistency)
db.employees.validate()
db.employees.validate({ full: true })

// Compact collection (reclaim space)
db.runCommand({ compact: "employees" })

// Measure query time
const start = new Date()
db.employees.find({ department: "IT" }).toArray()
print("Time:", new Date() - start, "ms")
```

---

## 23. Replication

```javascript
// Initialize replica set (run on primary)
rs.initiate({
  _id: "myReplicaSet",
  members: [
    { _id: 0, host: "mongo1:27017" },
    { _id: 1, host: "mongo2:27017" },
    { _id: 2, host: "mongo3:27017" }
  ]
})

// Check replica set status
rs.status()

// Check replica set config
rs.conf()

// Add member to replica set
rs.add("mongo4:27017")

// Add arbiter (votes but no data)
rs.addArb("mongo5:27017")

// Remove member
rs.remove("mongo4:27017")

// Step down primary (triggers election)
rs.stepDown()

// Force reconfiguration
rs.reconfig(config, { force: true })

// Check if current node is primary
rs.isMaster()

// Read from secondaries (in mongosh)
db.setReadPref("secondary")
db.setReadPref("secondaryPreferred")
db.setReadPref("primaryPreferred")
db.setReadPref("nearest")

// Freeze secondary (prevent it from becoming primary)
rs.freeze(120)   // Freeze for 120 seconds

// Oplog info
db.getReplicationInfo()
db.printReplicationInfo()
db.printSecondaryReplicationInfo()
```

---

## 24. Sharding

```javascript
// Enable sharding on a database
sh.enableSharding("mydb")

// Shard a collection by a key
sh.shardCollection("mydb.employees", { department: 1 })

// Shard with hashed key (even distribution)
sh.shardCollection("mydb.logs", { _id: "hashed" })

// Shard with compound key
sh.shardCollection("mydb.orders", { region: 1, order_date: 1 })

// Sharding status
sh.status()

// Add a shard
sh.addShard("shard1/mongo1:27017,mongo2:27017")

// Remove a shard
db.adminCommand({ removeShard: "shard1" })

// Check sharding status of collection
db.employees.getShardDistribution()

// Move a chunk manually
db.adminCommand({
  moveChunk: "mydb.employees",
  find: { department: "IT" },
  to: "shard2"
})

// Split a chunk
db.adminCommand({
  split: "mydb.employees",
  middle: { department: "M" }
})

// Balancer status
sh.getBalancerState()
sh.isBalancerRunning()

// Enable/disable balancer
sh.startBalancer()
sh.stopBalancer()
```

---

## 25. Atlas Search

```javascript
// Create Atlas Search index (via Atlas UI or API)
// Example index definition:
{
  "mappings": {
    "dynamic": true,
    "fields": {
      "first_name": [{ "type": "string", "analyzer": "lucene.standard" }],
      "salary":     [{ "type": "number" }]
    }
  }
}

// Basic Atlas Search query
db.employees.aggregate([
  {
    $search: {
      index: "default",
      text: {
        query: "John",
        path:  "first_name"
      }
    }
  }
])

// Fuzzy search
db.employees.aggregate([
  {
    $search: {
      text: {
        query:  "Jhon",
        path:   "first_name",
        fuzzy:  { maxEdits: 1 }
      }
    }
  }
])

// Search with score
db.employees.aggregate([
  {
    $search: {
      text: { query: "engineer", path: "job_title" }
    }
  },
  {
    $project: {
      name:  1,
      score: { $meta: "searchScore" }
    }
  },
  { $sort: { score: -1 } }
])

// Autocomplete
db.employees.aggregate([
  {
    $search: {
      autocomplete: {
        query: "Joh",
        path:  "first_name"
      }
    }
  }
])

// Compound query (must / should / mustNot / filter)
db.employees.aggregate([
  {
    $search: {
      compound: {
        must:   [{ text:   { query: "engineer", path: "title" } }],
        should: [{ range:  { path: "salary", gte: 80000 } }],
        mustNot:[{ text:   { query: "intern",   path: "title" } }],
        filter: [{ equals: { path: "is_active", value: true } }]
      }
    }
  }
])

// Search facets
db.employees.aggregate([
  {
    $searchMeta: {
      facet: {
        operator: { text: { query: "engineer", path: "title" } },
        facets: {
          departmentFacet: { type: "string", path: "department" },
          salaryFacet: {
            type: "number",
            path: "salary",
            boundaries: [40000, 70000, 100000]
          }
        }
      }
    }
  }
])
```

---

## 26. Advanced Features

### Bulk Operations

```javascript
// Bulk write (mix of operations)
db.employees.bulkWrite([
  { insertOne: { document: { name: "Alice", salary: 70000 } } },
  { updateOne: { filter: { name: "Bob" }, update: { $set: { salary: 80000 } } } },
  { deleteOne: { filter: { name: "Charlie" } } },
  {
    replaceOne: {
      filter: { name: "Dave" },
      replacement: { name: "Dave", salary: 90000 },
      upsert: true
    }
  }
], { ordered: false })   // Continue on error

// Unordered bulk (parallel, faster)
const bulk = db.employees.initializeUnorderedBulkOp()
bulk.insert({ name: "A" })
bulk.find({ name: "B" }).update({ $set: { salary: 60000 } })
bulk.find({ name: "C" }).remove()
bulk.execute()
```

### Capped Collections

```javascript
// Create capped collection
db.createCollection("event_log", { capped: true, size: 5242880, max: 1000 })

// Check if capped
db.event_log.isCapped()

// Convert to capped
db.runCommand({ convertToCapped: "event_log", size: 5242880 })

// Tailable cursor (like tail -f on capped collection)
const cursor = db.event_log.find().addOption(DBQuery.Option.tailable)
```

### Time Series Collections (MongoDB 5.0+)

```javascript
// Create time series collection
db.createCollection("sensor_data", {
  timeseries: {
    timeField:     "timestamp",
    metaField:     "sensor_id",
    granularity:   "seconds"   // "minutes" or "hours"
  },
  expireAfterSeconds: 86400    // Auto-delete after 1 day
})

// Insert time series data
db.sensor_data.insertMany([
  { sensor_id: "sensor_01", timestamp: new Date(), temperature: 22.5, humidity: 60 },
  { sensor_id: "sensor_01", timestamp: new Date(), temperature: 22.7, humidity: 61 }
])

// Query time series
db.sensor_data.find({
  sensor_id: "sensor_01",
  timestamp: { $gte: new Date("2024-01-01"), $lt: new Date("2024-01-02") }
})

// Aggregate time series
db.sensor_data.aggregate([
  { $match: { sensor_id: "sensor_01" } },
  {
    $group: {
      _id: {
        $dateTrunc: { date: "$timestamp", unit: "hour" }
      },
      avg_temp: { $avg: "$temperature" }
    }
  },
  { $sort: { _id: 1 } }
])
```

### Transactions with Retries

```javascript
// Retry logic for transient errors
function runTransactionWithRetry(txnFunc, session) {
  while (true) {
    try {
      txnFunc(session)
      break
    } catch (error) {
      if (error.hasErrorLabel("TransientTransactionError")) {
        print("Transient error, retrying...")
        continue
      } else {
        throw error
      }
    }
  }
}

function commitWithRetry(session) {
  while (true) {
    try {
      session.commitTransaction()
      print("Transaction committed.")
      break
    } catch (error) {
      if (error.hasErrorLabel("UnknownTransactionCommitResult")) {
        print("Unknown commit result, retrying...")
        continue
      } else {
        throw error
      }
    }
  }
}
```

### Miscellaneous Useful Commands

```javascript
// Copy collection within same database
db.employees.aggregate([{ $out: "employees_backup" }])

// Copy collection to another database
db.employees.aggregate([{ $out: { db: "archive", coll: "employees_2023" } }])

// Random document
db.employees.aggregate([{ $sample: { size: 1 } }])

// Check MongoDB version
db.version()

// Server build info
db.adminCommand({ buildInfo: 1 })

// List all databases with sizes
db.adminCommand({ listDatabases: 1 })

// Force garbage collection
db.runCommand({ repairDatabase: 1 })

// Get last error
db.getLastError()

// Set write concern globally
db.getMongo().setWriteConcern({ w: "majority", wtimeout: 5000 })

// Object ID operations
const id = new ObjectId()
id.getTimestamp()          // Extract creation time from ObjectId

// Convert ObjectId to string and back
id.toString()
ObjectId("64abc123...")

// Binary data
new BinData(0, Base64.encode("binary data"))

// NumberLong and NumberInt
NumberLong("9007199254740993")
NumberInt(42)
NumberDecimal("3.14159")

// Regular expressions
db.employees.find({ email: /gmail\.com$/i })

// JavaScript execution in aggregation
db.employees.aggregate([
  {
    $function: {
      body: function(salary) { return salary * 1.1 },
      args: ["$salary"],
      lang: "js"
    }
  }
])

// Map-Reduce (legacy, prefer aggregation)
db.employees.mapReduce(
  function() { emit(this.department, this.salary) },
  function(key, values) { return Array.sum(values) },
  { out: "dept_total_salary" }
)
```

---

## Quick Reference Cheatsheet

| Task | Command |
|------|---------|
| Connect to DB | `mongosh "mongodb://user:pass@host/db"` |
| Show databases | `show dbs` |
| Switch database | `use mydb` |
| Show collections | `show collections` |
| Insert one | `db.col.insertOne({...})` |
| Insert many | `db.col.insertMany([{...}, {...}])` |
| Find all | `db.col.find()` |
| Find one | `db.col.findOne({filter})` |
| Update one | `db.col.updateOne({filter}, {$set:{...}})` |
| Update many | `db.col.updateMany({filter}, {$set:{...}})` |
| Delete one | `db.col.deleteOne({filter})` |
| Delete many | `db.col.deleteMany({filter})` |
| Count | `db.col.countDocuments({filter})` |
| Aggregate | `db.col.aggregate([{$stage},...])` |
| Create index | `db.col.createIndex({field: 1})` |
| List indexes | `db.col.getIndexes()` |
| Explain query | `db.col.find({}).explain("executionStats")` |
| Drop collection | `db.col.drop()` |
| Drop database | `db.dropDatabase()` |
| Dump database | `mongodump --db mydb --out /backup` |
| Restore database | `mongorestore --db mydb /backup/mydb` |
| Export JSON | `mongoexport --db mydb --collection col --out data.json` |
| Import JSON | `mongoimport --db mydb --collection col --file data.json` |

---

*MongoDB version reference: Most commands work on MongoDB 4.4+. Time series (5.0+), Atlas Search requires MongoDB Atlas. Transactions require a Replica Set or Sharded Cluster.*
