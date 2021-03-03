# Collection auth - ***Update : 5 Février 2021***

## Installation des dépendances

```zsh
npm install joi bcrypt @types/bcrypt jsonwebtoken @types/jsonwebtoken
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

## Création des helpers

Créer un fichier **src/helpers/auth.token.ts** et copier :

```ts
import JWT, { decode } from "jsonwebtoken";
import createError from "http-errors";
import { Request, Response, NextFunction } from "express";

// Decode userId at https://jwt.io/

export const signAccessToken = (userId: string) => {
  return new Promise((resolve, reject) => {
    const payload = { name: process.env.APP_NAME };
    const secret = process.env.ACCESS_TOKEN_SECRET;
    const options = {
      expiresIn: process.env.ACCESS_TOKEN_EXPIRES_IN,
      issuer: "jwt.io",
      audience: userId,
    };
    JWT.sign(payload, secret, options, (err, token) => {
      if (err) return reject(new createError.InternalServerError());
      resolve(token);
    });
  });
};

export const verifyAccessToken = (
  req: Request,
  res: Response,
  next: NextFunction
) => {
  const { authorization } = req.headers;
  if (!authorization) return next(new createError.Unauthorized());
  const token = authorization.split(" ")[1];
  JWT.verify(token, process.env.ACCESS_TOKEN_SECRET, (err, decoded) => {
    if (err) {
      const message =
        err.name === "JsonWebTokenError" ? "Unauthorized" : err.message;
      return next(new createError.Unauthorized(message));
    }
    req.body.decoded = decoded;
    next();
  });
};

export const signRefreshToken = (userId: string) => {
  return new Promise((resolve, reject) => {
    const payload = { name: process.env.APP_NAME };
    const secret = process.env.REFRESH_TOKEN_SECRET;
    const options = {
      expiresIn: process.env.REFRESH_TOKEN_EXPIRES_IN,
      issuer: "jwt.io",
      audience: userId,
    };
    JWT.sign(payload, secret, options, (err, token) => {
      if (err) return reject(new createError.InternalServerError());
      resolve(token);
    });
  });
};

export const verifyRefreshToken = (refreshToken: string) => {
  return new Promise((resolve, reject) => {
    JWT.verify(
      refreshToken,
      process.env.REFRESH_TOKEN_SECRET,
      (err, decoded) => {
        if (err) return reject(new createError.Unauthorized());
        // @ts-ignore
        resolve(decoded.aud);
      }
    );
  });
};
```

Créer un fichier **src/helpers/bcrypt.ts** et copier :

```ts
import bcrypt from "bcrypt";

export const encodePassword = async (password: string) => {
  const salt = await bcrypt.genSalt(10);
  const hashedPassword = await bcrypt.hash(password, salt);
  return hashedPassword;
};

export const decodePassword = (password: string, dbPassword: string) => {
  return bcrypt.compare(password, dbPassword);
};
```


## Création des routes utilisateur

Créer un fichier **src/routes/auth.routes.ts** et copier :

```ts
import { Router } from "express";
import createError from "http-errors";
import { User } from "../db";
import { authSchema } from "../validator";
import {
  encodePassword,
  decodePassword,
  signAccessToken,
  signRefreshToken,
  verifyRefreshToken,
} from "../helpers";

export const authRoutes = Router();

authRoutes.post("/register", async (req, res, next) => {
  try {
    const { email, password } = await authSchema.validateAsync(req.body);
    const user = await User.findOne({ email });
    if (user)
      throw new createError.Conflict(`${email} is already been registered`);
    const hashedPassword = await encodePassword(password);
    const newUser = new User({ email, password: hashedPassword });
    const savedUser = await newUser.save();
    const accessToken = await signAccessToken(savedUser.id);
    const refreshToken = await signRefreshToken(savedUser.id);
    return res.send({ accessToken, refreshToken });
  } catch (err) {
    next(err);
  }
});

authRoutes.post("/login", async (req, res, next) => {
  try {
    const { email, password } = await authSchema.validateAsync(req.body);
    const user = await User.findOne({ email });
    if (!user) throw new createError.NotFound("User not registered");
    const isValidPassword = await decodePassword(password, user.password);
    if (!isValidPassword)
      throw new createError.Unauthorized("email or password is not valid");
    const accessToken = await signAccessToken(user.id);
    const refreshToken = await signRefreshToken(user.id);
    return res.send({ accessToken, refreshToken });
  } catch (err) {
    if (err.isJoi)
      return next(new createError.BadRequest("Invalid email or password"));
    next(err);
  }
});

authRoutes.post("/refresh-token", async (req, res, next) => {
  try {
    const { refreshToken } = req.body;
    if (!refreshToken || typeof refreshToken !== "string")
      throw new createError.BadRequest();
    const userId = await verifyRefreshToken(refreshToken);
    if (typeof userId !== "string") throw new createError.BadRequest();
    const accessToken = await signAccessToken(userId);
    const newRefreshToken = await signRefreshToken(userId);
    return res.send({ accessToken, refreshToken: newRefreshToken });
  } catch (err) {
    next(err);
  }
});
```

## fichier .env

Ajouter les clés suivantes dans le fichier .env

```
ACCESS_TOKEN_SECRET=<YOUR_ACCESS_TOKEN_SECRET_KEY>
REFRESH_TOKEN_SECRET=<YOUR_REFRESH_TOKEN_SECRET_KEY>
ACCESS_TOKEN_EXPIRES_IN = 1d
REFRESH_TOKEN_EXPIRES_IN = 7d
```

## Création des tests

TO DO

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
