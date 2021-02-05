# Collection auth - ***Update : 5 Février 2021***

## Installation des dépendances

```zsh
npm install joi
```

## Création d'un model de collection d'utilisateur pour MongoDB

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

## Création d'un validateur

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
