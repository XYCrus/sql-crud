# SQL CRUD

## Part 0: Preview

In this repository I will perform SQL table creation, import and query. All the dataset used in this repository are generated from [mockaroo.com](https://mockaroo.com). 

## Part I: Restaurants

### Table Structure

- **Category** (genre of food)
- **Price tier** (cheap, medium, or expensive)
- **Neighborhood** (a particular NYC neighborhood)
- **Opening hours** (for simplicity, I assume each restaurant has the same opening hours every day)
- **Average rating** (out of 5 stars)
- **Good for kids** (true or false)
- **Comments** (messages that customers may leave)

### Mock dataset
Link: [restaurants.csv](data/restaurants.csv)

### Create tables 
```
.mode csv
.headers on

CREATE TABLE restaurants (
    id INTEGER PRIMARY KEY,
    name TEXT,
    category TEXT,
    price_tier TEXT,
    neighborhood TEXT,
    opening_hours INTEGER,
    average_rating REAL,
    good_for_kid BLOB;
    review_id INTEGER
);

CREATE TABLE reviews (
    id INTEGER PRIMARY KEY,
    comment TEXT
);

CREATE TABLE temp (
    id INTEGER PRIMARY KEY,
    name TEXT,
    category TEXT,
    price_tier TEXT,
    neighborhood TEXT,
    opening_hours INTEGER,
    average_rating REAL,
    good_for_kid TEXT
);

.import --csv --skip 1 data/restaurants.csv temp

INSERT INTO restaurants (id, name, category, price_tier, neighborhood, opening_hours, average_rating, good_for_kid) SELECT * FROM temp;

DROP TABLE temp;
```

### Queries

1. Find all cheap restaurants in 'Chinatown'.
```
SELECT name FROM restaurants WHERE price_tier = 'cheap' AND neighborhood = 'Chinatown';
```

2. Find all restaurants of 'Japanese Cuisine' with 3 stars or more, ordered by the number of stars in descending order.
```
SELECT name FROM restaurants WHERE average_rating > 3 AND category = 'Japanese Cuisine' ORDER BY average_rating DESC;
```

3. Find all restaurants that are open now.
```
SELECT name FROM restaurants WHERE opening_hours > strftime('%H', 'now');
```

4. Leave a review for restaurant 'id == 777'.
```
INSERT INTO reviews (id, comment) VALUES (1, 'Absolutely love this place!');

UPDATE restaurants SET review_id = 1 WHERE id = 777;
```

5. Delete all restaurants that are not good for kids.
```
DELETE FROM restaurants WHERE good_for_kid = 'False';
```

6. Find the number of restaurants in each NYC neighborhood.
```
SELECT neighborhood, count(name) FROM restaurants GROUP BY neighborhood;
```

## Part II: Snapchat

### Database App Function

#### Users Table

Users can register for the app by supplying their...

- email
- password
- handle (i.e. username).

#### Messages:

- Messages consist of text only.
- Messages are sent from one user to another specific user.
- Messages become invisible immediately after view and don't show up in the app thereafter.
- Messages are never actually deleted from the database table, even when invisible to the user.

#### Stories:

- Stories consist of text only.
- Stories are public and every user can see them.
- Stories become invisible 24 hours after posting and don't show up in the app thereafter.
- Stories are never deleted from the database table, even when invisible to the user.

### Mock dataset

Links: 
1. [1000 Users Dataset](data/users.csv)
1. [1000 Messages Dataset](data/messages.csv)
1. [1000 Stories Dataset](data/stories.csv)

### Create tables

```
.mode csv
.headers on

CREATE TABLE posts (
    id INTEGER PRIMARY KEY,
    content TEXT,
    visibility TEXT,
    type TEXT,
    hour_ago INTEGER,
    from_user_id INTEGER,
    to_user_id INTEGER
);

CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    email TEXT,
    password TEXT,
    handle TEXT
);

CREATE TABLE temp_stories (
    id INTEGER PRIMARY KEY,
    content TEXT,
    visibility TEXT,
    type TEXT,
    hour_ago INTEGER,
    from_user_id INTEGER
);

.import --csv --skip 1 data/messages.csv posts
.import --csv --skip 1 data/stories.csv temp_stories
.import --csv --skip 1 data/users.csv users

INSERT INTO posts (id, content, visibility, type, hour_ago, from_user_id) SELECT * FROM temp_stories;

DROP TABLE temp_stories;
``` 

### Queries
1. Register a new User.
```
INSERT INTO users (id, email, password, handle) VALUES (1001, '7777@nyu.edu', '5555', 'Ga7777');
```

2. Create a new Message sent by User '777' to User '1554'.
```
INSERT INTO posts (id, content, visibility, type, hour_ago, from_user_id, to_user_id) VALUES (2001, 'thxgrader', 'true', 'message', 20, 777, 1554);
```

3. Create a new Story by User '778'.
```
INSERT INTO posts (id, content, visibility, type, hour_ago, from_user_id) VALUES (2002, 'thankyougrader', 'true', 'story', 21, 778);
```

4. Show the 10 most recent visible Messages and Stories, in order of recency.
```
SELECT content FROM posts WHERE visibility = 'true' ORDER BY hour_ago ASC LIMIT 10;
```

5. Show the 10 most recent visible Messages sent by User '215' to User '170', in order of recency.
```
SELECT content FROM posts WHERE from_user_id = 215 AND to_user_id = 170 AND visibility = 'true' ORDER BY hour_ago ASC LIMIT 10;
```

6. Make all Stories that are more than 24 hours old invisible.
```
UPDATE posts SET visibility = 'false' WHERE hour_ago > 24;
```

7. Show all invisible Messages and Stories, in order of recency.
```
SELECT content FROM posts WHERE visibility = 'false' ORDER BY hour_ago ASC;
```

8. Show the number of posts by each User.
```
SELECT users.id, count(posts.id) FROM posts INNER JOIN users ON posts.from_user_id = users.id WHERE 1 GROUP BY users.id;
```

9. Show the post text and email address of all posts and the User who made them within the last 24 hours.
```
SELECT users.email, posts.content FROM posts INNER JOIN users ON posts.from_user_id = users.id WHERE posts.hour_ago < 24;
```

10. Show the email addresses of all Users who have not posted anything yet.
```
SELECT users.email FROM users LEFT JOIN posts ON posts.from_user_id = users.id WHERE posts.from_user_id IS NULL;
```