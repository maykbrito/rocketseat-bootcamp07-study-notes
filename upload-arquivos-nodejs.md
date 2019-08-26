# Upload Files with Nodejs

<p align="center">*</p>

**Multer**

```sh
yarn add multer
```

<p align="center">*</p>

**src/config/multer.js**

```js
import multer from "multer";
import crypto from "crypto";
import { extname, resolve } from "path";

export default {
  storage: multer.diskStorage({
    destination: resolve(__dirname, "..", "..", "tmp", "uploads"),
    filename: (req, file, cb) => {
      crypto.randomBytes(16, (err, res) => {
        if (err) return cb(err);

        return cb(null, res.toString("hex") + extname(file.originalname));
      });
    }
  })
};
```

<p align="center">*</p>

**create migration**

```sh
yarn sequelize migration:create --name=create-files
```

<p align="center">*</p>

**src/database/migrations/create-files.js**

```js
module.exports = {
  up: (queryInterface, Sequelize) =>
    queryInterface.createTable("files", {
      id: {
        type: Sequelize.INTEGER,
        allowNull: false,
        autoIncrement: true,
        primaryKey: true
      },
      name: {
        type: Sequelize.STRING,
        allowNull: false
      },
      path: {
        type: Sequelize.STRING,
        allowNull: false,
        unique: true
      },
      created_at: {
        type: Sequelize.DATE,
        allowNull: false
      },
      updated_at: {
        type: Sequelize.DATE,
        allowNull: false
      }
    }),

  down: queryInterface => queryInterface.dropTable("files")
};
```

<p align="center">*</p>

**migrate database**

```sh
yarn sequelize db:migrate
```

<p align="center">*</p>

**src/app/models/File**

```js
import Sequelize, { Model } from "sequelize";

class File extends Model {
  static init(sequelize) {
    super.init(
      {
        name: Sequelize.STRING,
        path: Sequelize.STRING,
        url: {
          type: Sequelize.VIRTUAL,
          get() {
            return `${process.env.APP_URL}/files/${this.path}`;
          }
        }
      },
      {
        sequelize
      }
    );

    return this;
  }
}

export default File;
```

<p align="center">*</p>

**src/app/controllers/FileController.js**

```js
import File from "../models/File";

class FileController {
  async store(req, res) {
    const { originalname: name, filename: path } = req.file;
    const file = await File.create({
      name,
      path
    });

    return res.json(file);
  }
}

export default new FileController();
```

<p align="center">*</p>

**src/routes.js**

```js
//...
import multer from "multer";
import multerConfig from "./config/multer";
import FileController from "./app/controllers/FileController";

const upload = multer(multerConfig);
//...
routes.post("/files", upload.single("file"), FileController.store);
//...
```

<p align="center">*</p>

**Create Relationship**

# Reference file in User.

```sh
yarn sequelize migration:create --name=add-avatar-field-to-users
```

<p align="center">*</p>

**src/database/migrations/add-avatar-field-to-users.js**

```js
module.exports = {
  up: (queryInterface, Sequelize) =>
    queryInterface.addColumn("users", "avatar_id", {
      type: Sequelize.INTEGER,
      references: { model: "files", key: "id" },
      onUpdate: "CASCADE",
      onDelete: "SET NULL",
      allowNull: true
    }),

  down: queryInterface => queryInterface.removeColumn("users", "avatar_id")
};
```

<p align="center">*</p>

**src/app/models/User.js**

```js
class User extends Model {
  //...
  static associate(models) {
    this.belongsTo(models.File, { foreignKey: "avatar_id", as: "avatar" });
  }
  //...
}
```

<p align="center">*</p>

**src/database/index.js**

```js
import Sequelize from "sequelize";

import User from "../app/models/User";
import File from "../app/models/File";

import databaseConfig from "../config/database";

const models = [User, File];

class Database {
  constructor() {
    this.init();
    this.mongo();
  }

  init() {
    this.connection = new Sequelize(databaseConfig);
    models
      .map(model => model.init(this.connection))
      .map(model => model.associate && model.associate(this.connection.models));
  }
}

export default new Database();
```
