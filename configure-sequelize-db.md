# Sequelize and Database

```sh
mkdir src/config
mkdir src/database/migrations
mkdir src/app/controllers
mkdir src/app/models

touch src/config/database.js
touch src/database/index.js
touch .sequelizerc
touch .env

yarn add sequelize dotenv pg pg-hstore
yarn add sequelize-cli -D

docker run --name database -e POSTGRES_PASSWORD=docker -p 5432:5432 -d postgres
```

$$*$$

**.sequelizerc**

```js
const { resolve } = require("path");

module.exports = {
  config: resolve(__dirname, "src", "config", "database.js"),
  "models-path": resolve(__dirname, "src", "app", "models"),
  "migrations-path": resolve(__dirname, "src", "database", "migrations"),
  "seeders-path": resolve(__dirname, "src", "database", "seeds")
};
```

$$*$$

**dotenv**

```env
# Database
DB_HOST=localhost
DB_NAME=gobarber
DB_USER=postgres
DB_PASS=docker
```

$$*$$

**src/config/database.js**

```js
require("dotenv/config");

module.exports = {
  dialect: "postgres",
  host: process.env.DB_HOST,
  username: process.env.DB_USER,
  password: process.env.DB_PASS,
  database: process.env.DB_NAME,
  define: {
    timestamps: true,
    underscored: true,
    underscoredAll: true
  }
};
```