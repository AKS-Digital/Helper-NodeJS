# Collection auth - ***Update : 5 Février 2021***

## Installation des dépendances

```zsh
npm install joi
```

## Création du model mongoDB

Créer un fichier **src/db/models/user.model.ts** et copier :

```ts
import mongoose, { Schema, Document } from "mongoose";

export interface IUser extends Document {
  email: string;
  password: string;
}

const UserSchema: Schema = new Schema({
  email: { type: String, required: true, lowercase: true, unique: true },
  password: { type: String, required: true },
});

export const User = mongoose.model<IUser>("User", UserSchema);
```

## Création des validateurs

Créer un fichier **src/validator/auth.validator.ts** et copier :

```ts
import Joi from "joi";

//email and password is required.
//password. This regex will enforce these rules:
// At least one upper case English letter, (?=.*?[A-Z])
// At least one lower case English letter, (?=.*?[a-z])
// At least one digit, (?=.*?[0-9])
// At least one special character, (?=.*?[#?!@$%^&*-])
// Minimum eight in length .{8,} (with the anchors)

export const authSchema = Joi.object({
  email: Joi.string().email().lowercase().required(),
  password: Joi.string()
    .pattern(
      new RegExp(
        "^(?=.*?[A-Z])(?=.*?[a-z])(?=.*?[0-9])(?=.*?[#?!@$%^&*-]).{8,}$"
      )
    )
    .required(),
});
```

## Création des routes utilisateur

Créer un fichier **src/routes/auth.routes.ts** et copier :

```ts
import { Router } from "express";
import createError from "http-errors";
import { authSchema } from "../validator";
import { User } from "../db";

const router = Router();

router.post("/register", async (req, res, next) => {
  try {
    const { email, password } = await authSchema.validateAsync(req.fields);
    const user = await User.findOne({ email });
    if (user) throw new createError.Conflict(`${email} is already exists`);
    const newUser = new User({ email, password });
    const savedUser = await newUser.save();
    return res.send(savedUser);
  } catch (err) {
    next(err);
  }
});

export default router;
```

## Création des tests

Créer un fichier **src/\_test\_/auth.test.ts** et copier :

```ts
import dotenv from "dotenv";
dotenv.config();
import supertest from "supertest";
import { app } from "../app";
import { mongoDB } from "../db";

const request = supertest(app);

beforeAll(async () => {
  await mongoDB.connect();
  await mongoDB.dropDatabase();
});

afterAll(async () => {
  await mongoDB.close();
});

describe("Auth", () => {
  const badEmail = "my@g.c";
  const badPassword = "azerty";
  const correctEmail = "test@test.com";
  const correctPassword = "&Test123";

  it("should test email validator", async (done) => {
    const response = await request.post("/register").send({
      email: badEmail,
      password: correctPassword,
    });
    expect(response.status).toBe(422);
    expect(response.body).toStrictEqual({
      message: '"email" must be a valid email',
    });
    done();
  });

  it("should test password validator", async (done) => {
    const response = await request.post("/register").send({
      email: correctEmail,
      password: badPassword,
    });
    expect(response.status).toBe(422);
    expect(response.body.message).toContain(
      "fails to match the required pattern"
    );
    done();
  });

  it("should create user", async (done) => {
    const response = await request.post("/register").send({
      email: correctEmail,
      password: correctPassword,
    });
    expect(response.status).toBe(200);
    expect(response.body._id).toBeDefined();
    expect(response.body.email).toBeDefined();
    expect(response.body.password).toBeDefined();
    done();
  });

  it("should return conflict when user already exists", async (done) => {
    const response = await request.post("/register").send({
      email: correctEmail,
      password: correctPassword,
    });
    expect(response.status).toBe(409);
    expect(response.body.message).toBe(`${correctEmail} is already exists`);
    done();
  });
});
```
