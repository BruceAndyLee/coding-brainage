```js
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