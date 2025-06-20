### Activity 1. Advanced Querying and Projections

**Objective:** This activity will challenge you to write more specific and efficient queries using a wider range of MongoDB query operators. You'll also learn to shape your query results using projections.

**Instructions:** [Download Movie Dataset CSV](./L1_Assests/Movie%20Dataset.csv)

**Solution:**
**A. Movies with High IMDb and Metacritic Scores:**
 - Find all movies released after 2010 that have an IMDb rating greater than 8.5 AND a Metacritic score greater than 80.
 - Only display their title, director, year, and their IMDb and Metacritic ratings.

```javascript
db.movies.find(
  {
    year: { $gt: 2010 },
    "ratings.imdb": { $gt: 8.5 },
    "ratings.metacritic": { $gt: 80 }
  },
  {
    title: 1,
    director: 1,
    year: 1,
    "ratings.imdb": 1,
    "ratings.metacritic": 1,
    _id: 0
  }
)
```

**B. Movies by Specific Directors or Genres:**
 - Find movies directed by "Quentin Tarantino" OR "Steven Spielberg".
 - Additionally, filter these results only to include movies that are either "Sci-Fi" or "Animation" genres.
 - Show only the title, director, and genres fields.

 ```javascript
db.movies.find(
  {
    $and: [
      {
        $or: [
          { director: "Quentin Tarantino" },
          { director: "Steven Spielberg" }
        ]
      },
      {
        $or: [
          { genres: "Sci-Fi" },
          { genres: "Animation" }
        ]
      }
    ]
  },
  {
    title: 1,
    director: 1,
    genres: 1,
    _id: 0
  }
)
```
**C. Movies with Specific Cast Members (Case-Insensitive) and No Metacritic Score:**
 - Find movies that feature "Leonardo DiCaprio" OR "Tom Hanks" in the cast (case-insensitive search for names).
 - Include only movies where the `ratings.metacritic` field exists.
 - Display title, cast, and `ratings.metacritic`.
```javascript
db.movies.find(
  {
    $or: [
      { cast: { $regex: "Leonardo DiCaprio", $options: "i" } },
      { cast: { $regex: "Tom Hanks", $options: "i" } }
    ],
    "ratings.metacritic": { $exists: true }
  },
  {
    title: 1,
    cast: 1,
    "ratings.metacritic": 1,
    _id: 0
  }
)
```
**D. Oscar-Nominated Dramas with a Specific Number of Nominations:**
 - Find movies that are Oscar-nominated, are of the "Drama" genre, and have received between 5 and 10 (inclusive) Oscar nominations.
```javascript
db.movies.find(
  {
    isOscarNominated: true,
    genres: "Drama",
    "awards.nominations": { $gte: 5, $lte: 10 }
  },
  {
    title: 1,
    year: 1,
    isOscarNominated: 1,
    genres: 1,
    "awards.nominations": 1,
    _id: 0
  }
).sort({ year: 1 }).limit(5)
```
---
### Activity 2. Intermediate Aggregation Pipeline

**Objective:** This activity will further develop your aggregation skills by combining multiple pipeline stages to derive meaningful insights from the dataset.

**Instructions:** [Download Movie Dataset CSV](./L1_Assests/Movie%20Dataset.csv)

**A. Directors by Average Rotten Tomatoes Score (Top 3):**
 - Calculate the average Rotten Tomatoes rating for each director.
 - Filter out directors who have fewer than 2 movies in the dataset.
 - Sort the results by the average Rotten Tomatoes rating in descending order.
 - Limit the output to the top 3 directors.

```javascript
// 🎬 Directors by Average Rotten Tomatoes Score (Top 3)
db.movies.aggregate([
  {
    $group: {
      _id: "$director",
      avgRottenTomatoes: { $avg: "$ratings.rottenTomatoes" },
      movieCount: { $sum: 1 }
    }
  },
  {
    $match: {
      movieCount: { $gte: 2 } // Only include directors with 2 or more movies
    }
  },
  {
    $sort: {
      avgRottenTomatoes: -1 // Sort by average rating descending
    }
  },
  {
    $limit: 3 // Get the top 3
  }
])
```
**B. Oscar-Winning Movies by Decade:**
 - Group movies by the decade they were released (e.g., 1990s, 2000s).
 - Count how many Oscar-winning movies `(awards.oscarsWon > 0)` fall into each decade.
 - Also, calculate the total number of Oscar wins for each decade.
 - Sort the decades in ascending order.
```javascript
// 🏆 Oscar-Winning Movies by Decade
db.movies.aggregate([
  {
    $match: {
      "awards.oscarsWon": { $gt: 0 } // Only consider Oscar-winning movies
    }
  },
  {
    $project: {
      _id: 0,
      title: 1,
      year: 1,
      oscarsWon: "$awards.oscarsWon",
      decade: { $multiply: [ { $floor: { $divide: ["$year", 10] } }, 10 ] }
    }
  },
  {
    $group: {
      _id: "$decade",
      totalOscarWinningMovies: { $sum: 1 },
      totalOscarsWon: { $sum: "$oscarsWon" }
    }
  },
  {
    $sort: {
      _id: 1 // Sort by decade ascending
    }
  }
])
```
**C. Top 5 Actors by Total IMDb Rating of Their Movies:**
 - Unwind the cast array to treat each actor's movie as a separate document.
 - Group by actor and calculate the sum of IMDb ratings for all movies they've been in.
 - Sort by the total IMDb rating in descending order.
 - Limit to the top 5 actors.
```javascript
// 🌟 Top 5 Actors by Total IMDb Rating of Their Movies
db.movies.aggregate([
  {
    $unwind: "$cast" // Deconstruct the cast array
  },
  {
    $group: {
      _id: "$cast", // Group by actor
      totalImdbRating: { $sum: "$ratings.imdb" },
      movieCount: { $sum: 1 } // Count movies per actor
    }
  },
  {
    $sort: {
      totalImdbRating: -1 // Sort by total IMDb rating descending
    }
  },
  {
    $limit: 5 // Get the top 5 actors
  }
])
```
---

 ### Activity 3. Personal Blog System

**Objective:** To design and implement a simple MongoDB-based database to manage a personal blog system, using the structure and content provided in the `post.csv` file. The system should allow efficient storage, retrieval, and management of blog posts, including metadata such as title, author, content, tags, and timestamps. The goal is to transition from a flat CSV data format to a structured, queryable, and scalable NoSQL data model using MongoDB, thereby enhancing data organization and supporting future blog application development.

**Instructions:** 
 - [Download posts CSV](./L1_Assests/posts.csv)
 - Based on the following data model, create two collections in MongoDB: `posts` and `comments`. Use the field definitions to structure your documents accordingly.

   **Solutions:**

**A. Data Insertion:**
 - Insert Posts
   
```javascript
use blogDB;

db.posts.insertMany([
  {
    _id: ObjectId("60c72b2f9f1b2c3d4e5f6a7b"),
    title: "Getting Started with MongoDB",
    content: "This post covers the basics of MongoDB setup and CRUD operations...",
    author: "Alice",
    tags: ["MongoDB", "NoSQL", "Database"],
    publishedDate: ISODate("2023-01-15T10:00:00Z"),
    status: "published",
    views: 120
  },
  {
    _id: ObjectId("60c72b2f9f1b2c3d4e5f6a7c"),
    title: "Advanced Aggregation Techniques",
    content: "Explore complex data analysis with the MongoDB Aggregation Framework...",
    author: "Bob",
    tags: ["MongoDB", "Aggregation", "Data Analysis"],
    publishedDate: ISODate("2023-02-20T14:30:00Z"),
    status: "published",
    views: 85
  },
  {
    _id: ObjectId("60c72b2f9f1b2c3d4e5f6a7d"),
    title: "Schema Design Best Practices",
    content: "Understanding embedding vs. referencing for optimal MongoDB schemas...",
    author: "Alice",
    tags: ["MongoDB", "Schema Design"],
    publishedDate: ISODate("2023-03-10T09:15:00Z"),
    status: "draft",
    views: 40
  }
]);

```
Below Output shows the successful data Insertion.

```json
{
  "acknowledged": true,
  "insertedIds": {
    "0": ObjectId("60c72b2f9f1b2c3d4e5f6a7b"),
    "1": ObjectId("60c72b2f9f1b2c3d4e5f6a7c"),
    "2": ObjectId("60c72b2f9f1b2c3d4e5f6a7d")
  }
}
```
 - Insert Comments for the first post

```javascript
db.comments.insertMany([
  {
    postId: ObjectId("60c72b2f9f1b2c3d4e5f6a7b"),
    author: "Charlie",
    commentText: "Very helpful introduction, thanks!",
    commentDate: ISODate("2023-01-16T11:00:00Z")
  },
  {
    postId: ObjectId("60c72b2f9f1b2c3d4e5f6a7b"),
    author: "Diana",
    commentText: "Clear and concise examples.",
    commentDate: ISODate("2023-01-16T15:30:00Z")
  }
]);
```
Below output show successful Comment Insertion
```javascript
acknowledged: true,
insertedIds: {
  '0': ObjectId('684c4c7d08129a7fd36b128c'),
  '1': ObjectId('684c4c7d08129a7fd36b128d')
}
```
- Query the comments collection

```javascript
db.comments.find();
```
Below output shows contents of the comments collection
```javascript
[
  {
    _id: ObjectId("684c4e7ebb697a02bd6b128c"),
    postId: ObjectId("60c72b2f9f1b2c3d4e5f6a7b"),
    author: "Charlie",
    commentText: "Very helpful introduction, thanks!",
    commentDate: ISODate("2023-01-16T11:00:00.000Z")
  },
  {
    _id: ObjectId("684c4e7ebb697a02bd6b128d"),
    postId: ObjectId("60c72b2f9f1b2c3d4e5f6a7b"),
    author: "Diana",
    commentText: "Clear and concise examples.",
    commentDate: ISODate("2023-01-16T15:30:00.000Z")
  }
]
```

**B. Querying Posts:**
 - Find all published posts.
```javascript
db.posts.find({ status: "published" });
```
Below output show all posts where status is published
```javascript
[
  {
    _id: ObjectId('60c72b2f9f1b2c3d4e5f6a7b'),
    title: 'Getting Started with MongoDB',
    content: 'This post covers the basics of MongoDB setup and CRUD operations...',
    author: 'Alice',
    tags: ['MongoDB', 'NoSQL', 'Database'],
    publishedDate: ISODate('2023-01-15T10:00:00.000Z'),
    status: 'published',
    views: 120
  },
  {
    _id: ObjectId('60c72b2f9f1b2c3d4e5f6a7c'),
    title: 'Advanced Aggregation Techniques',
    content: 'Explore complex data analysis with the MongoDB Aggregation Framework...',
    author: 'Bob',
    tags: ['MongoDB', 'Aggregation', 'Data Analysis'],
    publishedDate: ISODate('2023-02-20T14:30:00.000Z'),
    status: 'published',
    views: 85
  }
]
```      

 - Find posts with a specific tag (e.g., "MongoDB").

```javascript
db.posts.find({ tags: "MongoDB" });
```

Below output show all posts with tag "MongoDB"

```javascript
[
  {
    _id: ObjectId('60c72b2f9f1b2c3d4e5f6a7b'),
    title: 'Getting Started with MongoDB',
    content: 'This post covers the basics of MongoDB setup and CRUD operations...',
    author: 'Alice',
    tags: ['MongoDB', 'NoSQL', 'Database'],
    publishedDate: ISODate('2023-01-15T10:00:00.000Z'),
    status: 'published',
    views: 120
  },
  {
    _id: ObjectId('60c72b2f9f1b2c3d4e5f6a7c'),
    title: 'Advanced Aggregation Techniques',
    content: 'Explore complex data analysis with the MongoDB Aggregation Framework...',
    author: 'Bob',
    tags: ['MongoDB', 'Aggregation', 'Data Analysis'],
    publishedDate: ISODate('2023-02-20T14:30:00.000Z'),
    status: 'published',
    views: 85
  },
  {
    _id: ObjectId('60c72b2f9f1b2c3d4e5f6a7d'),
    title: 'Schema Design Best Practices',
    content: 'Understanding embedding vs. referencing for optimal MongoDB schemas...',
    author: 'Alice',
    tags: ['MongoDB', 'Schema Design'],
    publishedDate: ISODate('2023-03-10T09:15:00.000Z'),
    status: 'draft',
    views: 40
  }
]
```

 - Find posts published in a specific year (assuming 2023 for our data)
```javascript
db.posts.find({
  publishedDate: {
    $gte: ISODate("2023-01-01T00:00:00Z"),
    $lt: ISODate("2024-01-01T00:00:00Z")
  }
});
```
Below output shows all the posts published in year 2023
```javascript
[
  {
    _id: ObjectId('60c72b2f9f1b2c3d4e5f6a7b'),
    title: 'Getting Started with MongoDB',
    content: 'This post covers the basics of MongoDB setup and CRUD operations...',
    author: 'Alice',
    tags: ['MongoDB', 'NoSQL', 'Database'],
    publishedDate: ISODate('2023-01-15T10:00:00.000Z'),
    status: 'published',
    views: 120
  },
  {
    _id: ObjectId('60c72b2f9f1b2c3d4e5f6a7c'),
    title: 'Advanced Aggregation Techniques',
    content: 'Explore complex data analysis with the MongoDB Aggregation Framework...',
    author: 'Bob',
    tags: ['MongoDB', 'Aggregation', 'Data Analysis'],
    publishedDate: ISODate('2023-02-20T14:30:00.000Z'),
    status: 'published',
    views: 85
  },
  {
    _id: ObjectId('60c72b2f9f1b2c3d4e5f6a7d'),
    title: 'Schema Design Best Practices',
    content: 'Understanding embedding vs. referencing for optimal MongoDB schemas...',
    author: 'Alice',
    tags: ['MongoDB', 'Schema Design'],
    publishedDate: ISODate('2023-03-10T09:15:00.000Z'),
    status: 'draft',
    views: 40
  }
]
```
 - Find the title and author of all posts, sorted by `publishedDate descending`.
```javascript
db.posts.find(
  {},
  {
    title: 1,
    author: 1,
    _id: 0
  }
).sort({ publishedDate: -1 });
```
Below output shows title & author of all posts sorted by publish date
```javascript
[
  { title: 'Schema Design Best Practices', author: 'Alice' },
  { title: 'Advanced Aggregation Techniques', author: 'Bob' },
  { title: 'Getting Started with MongoDB', author: 'Alice' }
]
```

**C. Updating Posts:**
 - Change the status of a draft post to published.
```javascript
db.posts.updateOne(
  { title: "Schema Design Best Practices" },
  { $set: { status: "published" } }
);
```
Below output confirms that the status is updated.
```javascript
{
  acknowledged: true,
  insertedId: null,
  matchedCount: 1,
  modifiedCount: 1,
  upsertedCount: 0
}
```
 - Increment the views count for a specific post.
```javascript
db.posts.updateOne(
  { title: "Getting Started with MongoDB" },
  { $inc: { views: 1 } }
);
```
Below output shows the views count is changed
```javascript
{
  acknowledged: true,
  insertedId: null,
  matchedCount: 1,
  modifiedCount: 1,
  upsertedCount: 0
}
```
**D. Aggregations:**
 - Count the total number of posts by each author.
```javascript
db.posts.aggregate([
  {
    $group: {
      _id: "$author",
      totalPosts: { $sum: 1 }
    }
  }
]);
```
Below output gives the count of the total number of posts by each author
```javascript
[
  { _id: "Alice", totalPosts: 2 },
  { _id: "Bob", totalPosts: 1 }
]
```
 - Find the post with the most views.
```javascript
db.posts.find().sort({ views: -1 }).limit(1);
```
Below output show post with most views
```javascript
{
  _id: ObjectId("60c72b2f9f1b2c3d4e5f6a7b"),
  title: "Getting Started with MongoDB",
  content: "This post covers the basics of MongoDB setup and CRUD operations...",
  author: "Alice",
  tags: ["MongoDB", "NoSQL", "Database"],
  publishedDate: ISODate("2023-01-15T10:00:00.000Z"),
  status: "published",
  views: 121
}
```
