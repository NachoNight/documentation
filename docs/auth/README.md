# Authentication API

### Resources:

- GitHub Repository: [https://github.com/NachoNight/auth-api](https://github.com/NachoNight/auth-api)
- Sequelize: [https://sequelize.org/](https://sequelize.org/)
- JSON Web Token: [https://jwt.io/](https://jwt.io/)
- OAuth: [https://oauth.net/](https://oauth.net/)
- node-cache: [https://npmjs.com/package/node-cache/](https://npmjs.com/package/node-cache/)
- Nodemailer: [https://npmjs.com/package/nodemailer/](https://npmjs.com/package/nodemailer/)
- Mailgun: [https://www.mailgun.com/](https://www.mailgun.com/)
- EJS: [https://ejs.co/](https://ejs.co/)
- Axios: [https://www.npmjs.com/package/axios](https://www.npmjs.com/package/axios)

---

### Navigation:

- [Authentication API](#authentication-api)
    - [Resources:](#resources)
    - [Navigation:](#navigation)
- [Getting Started:](#getting-started)
- [Scripts:](#scripts)
- [Environmental Variables:](#environmental-variables)
- [Dependencies:](#dependencies)
- [Routes / Endpoints:](#routes--endpoints)
- [Configuration:](#configuration)
    - [Server:](#server)
    - [Database:](#database)
    - [Mail:](#mail)
    - [OAuth:](#oauth)
        - [Google and Discord:](#google-and-discord)
        - [Twitter:](#twitter)
        - [Shared:](#shared)
- [Database:](#database-1)
    - [Models:](#models)
- [Authentication:](#authentication)
    - [OAuth providers:](#oauth-providers)
    - [Usage:](#usage)
- [Caching:](#caching)
- [Mailing system:](#mailing-system)
    - [Templates:](#templates)
- [Functions:](#functions)
- [Middleware:](#middleware)
- [Utilities:](#utilities)
- [Input validation:](#input-validation)
- [Requests:](#requests)
    - [Root](#root)
    - [Register](#register)
    - [Send verification email](#send-verification-email)
    - [Verify account](#verify-account)
    - [Get current user](#get-current-user)
    - [Delete current user](#delete-current-user)
    - [Password recovery](#password-recovery)
    - [Restore password](#restore-password)
    - [Change email](#change-email)
    - [Verify email change](#verify-email-change)
    - [Change password](#change-password)
    - [Add email to collection](#add-email-to-collection)
    - [Remove email from collection](#remove-email-from-collection)

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

`/app/config/index.js`

```js
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

To interact with the database, we are using [Sequelize](https://sequelize.org/).

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

### Models:

At this time, only two models are in use:

- `email-address.model.js` - Email address model
- `user.model.js` - User model

These models are used for communicating with the database rather than having to use SQL queries.

You can find them in `/app/db/models`.

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

---

# Caching:

Our current cache setup runs in server memory using the [node-cache](https://www.npmjs.com/package/node-cache) module, rather than in a Redis instance or in any other solution which would be decoupled from the server. Currently, this is working fine, but in the future, we might change this due to data persistence if the sever crashes, etc.

The main purpose of the caching system is to temporarily hold password recovery and email verification tokens. This is a better solution that to actually store the tokens in the database itself.

The standard TTL (time to live) for keys in the cache is one hour (3600 seconds), and an automated check to remove dead keys is ran every 5 minutes (300 seconds).

---

# Mailing system:

For sending out emails, we are using [Nodemailer](https://www.npmjs.com/package/nodemailer).

This solution was a no-brainer since the package is really easy to use, and it is a good performer.

You will need to provide your own SMTP credentials in the `.env` file for it to work.

The service we are using for testing is [Mailgun](https://www.mailgun.com/email-api), which has an amazing free tier.

That said, you do need to provide a list of authorized recipients when using a free account, otherwise you will be getting errors.

`/app/mail/index.js`

```js
const { createTransport } = require("nodemailer");
const { renderFile } = require("ejs");
const { join } = require("path");
const { host, port, user, pass, sender } = require("../config").mail;

const transport = createTransport({
  host,
  port,
  secure: false,
  auth: {
    user,
    pass
  }
});

module.exports = (to, subject, text, template = "sample") => {
  const config = {
    from: sender,
    to,
    subject
  };
  renderFile(
    join(__dirname, `/templates/${template}.ejs`),
    { ...config, body: text },
    async (err, data) => {
      if (err) throw err;
      await transport.sendMail(
        {
          ...config,
          html: data
        },
        error => {
          if (error) {
            console.error(`Email could not be sent to ${to}`);
          } else {
            console.log(`Email sent to ${to}`);
          }
        }
      );
    }
  );
};
```

### Templates:

Templates are written in [EJS](https://ejs.co/) which get rendered as HTML and sent over to the recipient.

`/app/mail/templates/sample.ejs`

```js
<html>
  <body>
    <h1>NachoNight</h1>
    <p>From: <%= from %></p>
    <p>To: <%= to %></p>
    <p>Subject: <%= subject %></p>
    <div>
      <%= body %>
    </div>
  </body>
</html>
```

---

# Functions:

There are a few functions that have been written to avoid repetition. They are located in the `/app/functions` folder.

- `addStrategy` - Adds a strategy to Passport
- `determineCredentials` - Determines which credentials are required for which path
- `generateRandomBytes` - Generates a 256 bit long string
- `generateToken` - Generates and signs a JSON web token
- `pathRequiresUser` - Checks if the current endpoint requires authentication
- `registerUser` - Registers a new user

---

# Middleware:

The server and most endpoints run middleware.

The server middleware application is bootstrapped into a single function exported from `/app/middleware/index.js`.

`/app/middleware/index.js`

```js
const { json, urlencoded } = require("body-parser");
const session = require("express-session");
const { secret } = require("../config").server;
const logger = require("./logger");

module.exports = app => {
  app.use(json());
  app.use(urlencoded({ extended: true }));
  app.use(session({ secret, resave: true, saveUninitialized: true }));
  logger(app);
};
```

The other middleware is located in the same folder. Here's a list:

- `checkForUser` - Checks if the user making the request exists
- `checkIfBanned` - Checks if the user making the request is banned
- `logger` - Initializes the logging middleware
- `validateInput` - Input validation handler

---

# Utilities:

Currently, we are only running one utility function which is used for testing endpoints in our test suite.

`apiTestingUtility`, located in `/app/utils`, is an [Axios](https://www.npmjs.com/package/axios) instance which calls the various API routes in the `test.test.js` file located in `/app/test`.

`/app/utils/apiTestingUtility.js`

```js
const axios = require("axios");
const { port } = require("../config").server;

module.exports = async (method, endpoint, data = null, token = "") => {
  axios.defaults.headers.common.Authorization = token;
  const res = await axios[method](`http://localhost:${port}${endpoint}`, data);
  return {
    status: res.status,
    data: res.data
  };
};
```

---

# Input validation:

Located in `/app/validation` are the validation methods and module we use for checking provided inputs and sanitization.

The method naming convention tries to resemble the route endpoint names as much as possible to ensure that the `validateInput` middleware can call it.

We will not cover the methods here, so if you are interested to see how they work, you can take a look in the [GitHub repository](https://github.com/NachoNight/auth-api).

---

# Requests:

### Root

`GET /`

Input: None

Requires authentication: false

Response: NachoNight Authentication API

---

### Register

`POST /register`

Example input:

```json
{
  "email": "test@example.com",
  "password": "test12345",
  "confirmPassword": "test12345"
}
```

Requires authentication: false

Example response:

```json
{
  "verified": false,
  "banned": false,
  "created": "2019-10-06T11:49:06.189Z",
  "clientID": "d0a8a9a4-af9a-4e0c-9dbd-80209ce48267",
  "id": 1,
  "email": "test@example.com",
  "password": "$2b$14$2PGckwtaM9NL3LLPdSZgrOAXJ.hQ5D0wLapO9iAGbXHTydgV.3yYi",
  "updatedAt": "2019-10-06T11:49:06.190Z",
  "createdAt": "2019-10-06T11:49:06.190Z"
}
```

---

### Send verification email

`GET /send-verification`

Input: None

Requires authentication: true

Example response:

```json
{
  "verified": false,
  "banned": false,
  "created": "2019-10-06T11:49:06.189Z",
  "clientID": "d0a8a9a4-af9a-4e0c-9dbd-80209ce48267",
  "id": 1,
  "email": "test@example.com",
  "password": "$2b$14$2PGckwtaM9NL3LLPdSZgrOAXJ.hQ5D0wLapO9iAGbXHTydgV.3yYi",
  "updatedAt": "2019-10-06T11:49:06.190Z",
  "createdAt": "2019-10-06T11:49:06.190Z"
}
```

---

### Verify account

`GET /verify-account/:token`

Input: none

Requires authentication: false

Example response:

```json
{
  "verified": true,
  "banned": false,
  "created": "2019-10-06T11:49:06.189Z",
  "clientID": "d0a8a9a4-af9a-4e0c-9dbd-80209ce48267",
  "id": 1,
  "email": "test@example.com",
  "password": "$2b$14$2PGckwtaM9NL3LLPdSZgrOAXJ.hQ5D0wLapO9iAGbXHTydgV.3yYi",
  "updatedAt": "2019-10-06T11:49:06.190Z",
  "createdAt": "2019-10-06T11:49:06.190Z"
}
```

---

### Get current user

`GET /current`

Input: none

Requires authentication: true

Example response:

```json
{
  "id": 1,
  "email": "test@example.com",
  "verified": false,
  "banned": false,
  "clientID": "d0a8a9a4-af9a-4e0c-9dbd-80209ce48267",
  "created": "2019-10-06T11:49:06.189Z"
}
```

---

### Delete current user

`DELETE /delete`

Input: none

Requires authentication: true

Example response:

```json
{
  "deleted": true,
  "timestamp": 1570363123168
}
```

---

### Password recovery

`PATCH /forgot`

Example input:

```json
{
  "email": "test@example.com"
}
```

Requires authentication: false

Response:

```json
{
  "initiatedPasswordRecovery": true
}
```

---

### Restore password

`PATCH /recover/:token`

Input: none

Requires authentication: false

Example response:

```json
{
  "verified": true,
  "banned": false,
  "created": "2019-10-06T11:49:06.189Z",
  "clientID": "d0a8a9a4-af9a-4e0c-9dbd-80209ce48267",
  "id": 1,
  "email": "test@example.com",
  "password": "$2b$14$2PGckwtaM9NL3dfasfAwOLpaHHfglasgrOAXJ.hQ5D0wLapO9iAGbXHTydgV.3yYi",
  "updatedAt": "2019-10-06T11:49:06.190Z",
  "createdAt": "2019-10-06T11:49:06.190Z"
}
```

---

### Change email

`PUT /change-email`

Example input:

```json
{
  "email": "edited_email@example.com"
}
```

Requires authentication: true

Example response:

```json
{
  "initiatedEmailChange": true
}
```

---

### Verify email change

`GET /verify-email-change/:token`

Input: none

Requires authentication: false

Example response:

```json
{
  "verified": true,
  "banned": false,
  "created": "2019-10-06T11:49:06.189Z",
  "clientID": "d0a8a9a4-af9a-4e0c-9dbd-80209ce48267",
  "id": 1,
  "email": "edited_email@example.com",
  "password": "$2b$14$2PGckwtaM9NL3dfasfAwOLpaHHfglasgrOAXJ.hQ5D0wLapO9iAGbXHTydgV.3yYi",
  "updatedAt": "2019-10-06T11:49:06.190Z",
  "createdAt": "2019-10-06T11:49:06.190Z"
}
```

---

### Change password

`PUT /change-password`

Example input:

```json
{
  "password": "12345test",
  "confirmPassword": "12345test"
}
```

Requires authentication: true

Example response:

```json
{
  "verified": true,
  "banned": false,
  "created": "2019-10-06T11:49:06.189Z",
  "clientID": "d0a8a9a4-af9a-4e0c-9dbd-80209ce48267",
  "id": 1,
  "email": "edited_email@example.com",
  "password": "$2b$14$2PGcasch124HGFHazorGprOAXJ.hQ5D0wLapO9iAGbXHTydgV.a4Y5",
  "updatedAt": "2019-10-06T11:49:06.190Z",
  "createdAt": "2019-10-06T11:49:06.190Z"
}
```

---

### Add email to collection

`POST /add-address`

Example input:

```json
{
  "email": "account@test.com"
}
```

Requires authentication: false

Output:

```json
{
  "action": "created"
}
```

---

### Remove email from collection

`DELETE /remove-address`

Example input:

```json
{
  "email": "account@test.com"
}
```

Requires authentication: false

Output:

```json
{
  "action": "deleted"
}
```

---
