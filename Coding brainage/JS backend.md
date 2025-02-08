---
Status: In progress
tags:
 - backend
---
# Node.js

## навигация между версиями

```Shell
nvm install 10.14.0
nvm use 10.15.10
```

## настройка express + mongo orm (mongoose)

orm Model + REST controller + route mapper

```JavaScript
/**
 * postModel.js
 * ORM MODEL
 */
const mongoose = require("mongoose");

const userSchema = new mongoose.Schema({
  username: {
    type: String,
    required: [true, "User must have username"],
    unique: true,
  },
  password: {
    type: String,
    required: [true, "User must have password"],
  },
});

const User = mongoose.model("User", userSchema);

module.exports = User;
```

```JavaScript
/**
 * postController.js
 * defining REST controller functions
 */

const Post = require("../models/postModel");

exports.getAllPosts = async (req, res, next) => {
  try {
    const posts = await Post.find();
    res.status(200).json({
      status: "Success",
      data: posts,
      results: posts.length,
    });
  } catch (e) {
    res.status(500).json({
      status: "Internal server error",
      error: e.toString(),
    });
  }
};

exports.getPost = async (req, res, next) => {
  try {
    const post = await Post.find(req.params.id);
    res.status(200).json({
      status: "Success",
      data: post,
    });
  } catch (e) {
    res.status(500).json({
      status: "Internal server error",
      error: e.toString(),
    });
  }
};

exports.createPost = async (req, res, next) => {
  try {
    const post = await Post.create(req.body);
    res.status(200).json({
      status: "Success",
      data: post,
    });
  } catch (e) {
    res.status(500).json({
      status: "Internal server error",
      error: e.toString(),
    });
  }
};

exports.updatePost = async (req, res, next) => {
  try {
    const post = await Post.findByIdAndUpdate(req.params.id, req.body, {
      new: true,
      runValidators: true,
    });
    res.status(200).json({
      status: "Success",
      data: post,
    });
  } catch (e) {
    res.status(500).json({
      status: "Internal server error",
      error: e.toString(),
    });
  }
};

exports.deletePost = async (req, res, next) => {
  try {
    await Post.findByIdAndDelete(req.params.id);
    res.status(200).json({
      status: "Success",
    });
  } catch (e) {
    res.status(500).json({
      status: "Internal server error",
      error: e.toString(),
    });
  }
};
```

```JavaScript
/**
 * postRouter.js
 * mapping routes onto rest controller functions
 */

const express = require("express");

const postController = require("../controllers/postController");

const router = express.Router();

// localhost:3000/
router
  .route("/")
  .get(postController.getAllPosts)
  .post(postController.createPost);

router
  .route("/:id")
  .get(postController.getPost)
  .patch(postController.updatePost)
  .delete(postController.deletePost);

module.exports = router;
```

## настройка express-session + redis

```Bash
/**
 * index.js
 */

const session = require("express-session");
const redis = require("redis");

let RedisStore = require("connect-redis")(session);

let redisClient = redis.createClient({
  host: REDIS_URL, // перменная окружения
  port: REDIS_PORT, // перменная окружения
});

app.use(
  session({
    store: new RedisStore({ client: redisClient }),
    secret: SESSION_SECRET,
    cookie: {
      secure: false,
      resave: false,
      saveUninitialized: false,
      httpOnly: true,
      maxAge: 30000,
    },
  })
);
```

почитать дополнительно про настройки сессий в миддлвэр: [https://www.npmjs.com/package/express-session](https://www.npmjs.com/package/express-session)

  

express CORS: [https://expressjs.com/en/resources/middleware/cors.html](https://expressjs.com/en/resources/middleware/cors.html)

- express
- nest.js