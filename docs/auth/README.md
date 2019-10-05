# Authentication API

---

Resources:

* GitHub Repository: [https://github.com/NachoNight/auth-api](https://github.com/NachoNight/auth-api)

# Getting Started:

Let's get started, shall we!

To run the application, the dependencies listed [here](#dependencies) have to be installed.

Be sure to check the version numbers, to avoid any version-mismatch related errors.

# Scripts:

* `npm start` - Starts the Node server.
* `npm run dev` - Starts the Node server using Nodemon, which enables hot reloading on save.
* `npm test` - Runs the test suite.

# Environmental Variables:

An `.env` file is required in the root directory of the file structure which will contain all the necessary metadata to run the application.

`.env.example`

```env
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

# Dependencies:

* Node.js (v. 10.16.3)
* npm (v. 6.9.0)
* PostgreSQL (v. 11.5)
* Docker (v. 19.03.2)
* Docker-compose (v. 1.24.1)

