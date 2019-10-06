# Authentication API

Resources:

- GitHub Repository: [https://github.com/NachoNight/auth-api](https://github.com/NachoNight/auth-api)

---

# Getting Started:

Let's get started, shall we!

To run the application, the dependencies listed [here](#dependencies) have to be installed.

Be sure to check the version numbers, to avoid any version-mismatch related errors.

---

# Scripts:

- `npm start` - Starts the Node server.
- `npm run dev` - Starts the Node server using Nodemon, which enables hot reloading on save.
- `npm test` - Runs the test suite.

---

# Environmental Variables:

An `.env` file is required in the root directory of the file structure which will contain all the necessary metadata to run the application.

There is an example file included in the repository.

`/.env.example`

```
NODE_ENV=
DB_NAME=
DB_USER=
DB_PASSWORD=
DB_HOST=
DB_DIALECT=
DB_PORT=
SMTP_HOSTNAME=
MAIL_PORT=
MAIL_USER=
MAIL_PASSWORD=
MAIL_SENDER=
SECRET=
GOOGLE_OAUTH_CLIENT_ID=
GOOGLE_OAUTH_CLIENT_SECRET=
TWITTER_OAUTH_CONSUMER_KEY=
TWITTER_OAUTH_CONSUMER_SECRET=
DISCORD_OAUTH_CLIENT_ID=
DISCORD_OAUTH_CLIENT_SECRET=
```

---

# Dependencies:

- Node.js (v. 10.16.3)
- npm (v. 6.9.0)
- PostgreSQL (v. 11.5)
- Docker (v. 19.03.2)
- Docker-compose (v. 1.24.1)

---

# Routes / Endpoints:

- Location: `/app/router.js`

- `GET /` - Loads the root page which should render 'NachoNight Authentication API'
- `POST /register` - Registers a new user
- `GET /send-verification` - Sends out a verification email
- `GET /verify-account/:token` - Account verification handler
- `POST /login` - Login endpoint
- `GET /current` - Get the current user data
- `DELETE /delete` - Delete the account of the current user
- `PATCH /forgot` - Sends out an email for password recovery
- `PUT /recover/:token` - Password recovery handler
- `PUT /change-email` - Requests an email change and sends out an email to the new address
- `GET /verify-email-change/:token` - Email change handler
- `PUT /change-password` - Password change handler
- `POST /add-address` - Email collection handler
- `DELETE /remove-address` - Removes email from our collection
- `GET /auth/google` - Google OAuth endpoint
- `GET /auth/google/callback` - Google OAuth handler
- `GET /auth/discord` - Discord OAuth endpoint
- `GET /auth/discord/callback` - Discord OAuth handler

---

# Configuration:

- Location: `/app/config/index.js`

`/app/config/index.js`

```js
// Load .env
require("dotenv").config();

module.exports = {
  server: {
    port: process.env.SERVER_PORT || 5000,
    environment: process.env.NODE_ENV || "development",
    secret: process.env.SECRET
  },
  database: {
    name: process.env.DB_NAME,
    username: process.env.DB_USER,
    password: process.env.DB_PASSWORD,
    host: process.env.DB_HOST,
    dialect: process.env.DB_DIALECT || "postgres",
    port: process.env.DB_PORT || 5432
  },
  mail: {
    host: process.env.SMTP_HOSTNAME,
    port: process.env.MAIL_PORT,
    user: process.env.MAIL_USER,
    pass: process.env.MAIL_PASSWORD,
    sender: process.env.MAIL_SENDER
  },
  oauth: {
    google: {
      clientID: process.env.GOOGLE_OAUTH_CLIENT_ID,
      clientSecret: process.env.GOOGLE_OAUTH_CLIENT_SECRET,
      callbackURL: "/auth/google/callback"
    },
    discord: {
      clientID: process.env.DISCORD_OAUTH_CLIENT_ID,
      clientSecret: process.env.DISCORD_OAUTH_CLIENT_SECRET,
      callbackURL: "/auth/discord/callback"
    },
    twitter: {
      consumerKey: process.env.TWITTER_OAUTH_CONSUMER_KEY,
      consumerSecret: process.env.TWITTER_OAUTH_CONSUMER_SECRET,
      callbackURL: "/auth/twitter/callback"
    }
  }
};
```

### Server:

- `Port` - Server Port
- `environment` - Current Node environment
- `secret` - Secret

### Database:

- `name` - Table name
- `username` - Database username
- `password` - Password of the database user
- `host` - Current hostname of the server hosting the database
- `dialect` - Name of the DBMS. By default it falls back to Postgres if nothing is provided
- `port` - Port on which the database is running on

### Mail:

- `host` - SMTP hostname
- `port` - SMTP port. The default port should be 587
- `user` - Mailgun user credentials
- `pass` - Password of the mailgun user
- `sender` - The email which will be used to send emails to users

### OAuth:

##### Google and Discord:

- `clientID` - OAuth client ID
- `clientSecret` - OAuth client secret

##### Twitter:

- `consumerKey` - Twitter API consumer key
- `consumerSecret` - Twitter API consumer secret

##### Shared:

- `callbackURL` - OAuth registration/login handler

---

# Database:

To interact with the database, we are using the Sequelize package.

The `Sequelize` constructor requires a database user and password, table name and a port to establish a connection.

All the information is provided by the [configuration](#configuration) which is constructed by the [environmental variables](#environmental-variables).

The connection module responds to changes in the `NODE_ENV` variable, which by default is set to `development`.

Also remember to change your database related environmental variables to match the environment your database is running in.

`/app/db/index.js`

```js
const Sequelize = require("sequelize");
const { resolve } = require("path");
const { readFileSync } = require("fs");
const { database, server } = require("../config");

const connect = () => {
  const { name, username, password, host, dialect, port } = database;
  const config = {
    host,
    dialect,
    port,
    logging: false
  };
  if (server.environment !== "development") {
    config.dialectOptions = {
      ssl: true,
      ca: readFileSync(resolve(__dirname, "../keys", "certificate.crt"))
    };
  }
  return new Sequelize(name, username, password, config);
};

module.exports = connect();
```

---

# Authentication:

Currently, we have two authentication strategies in place:

- [JSON Web Token](https://jwt.io/introduction/)
- [OAuth](https://oauth.net/about/introduction/)

### OAuth providers:

The application is using the OAuth APIs provided by Google and Discord.

In a future implementation, Twitter OAuth should be added to our list of providers.

You must generate your own OAuth credentials (Client IDs and secrets) and provide them in the `.env` file.

Make sure to set valid callback URLs, for example, if you are running the application locally: `http://localhost:<YOUR_PORT>/auth/<GOOGLE_OR_DISCORD>/callback`.

### Usage:

All authentication strategies work more or less in the same way.

The JWT strategy checks if a user exists within our database. If it does, you are succesfully validated, and the request object gets mutated with a new key, `user`, which is used for convenient and easy access to user data.
If there's no user in the database, the strategy just errors out and an 404 is sent out.

OAuth, on the other hand, finds or creates a new user. If the user exists, it will trigger a redirect to the previously mentioned callback URL. Otherwise, it will just create a new user.

You can take a look in how the authentication system works by looking at the `addStrategy` function located in `/app/functions/`.
