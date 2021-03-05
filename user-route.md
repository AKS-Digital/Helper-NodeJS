# Collection (User) - Update : 4 Mars 2021

## Model Mongoose

Création ou mise à jour du model

```ts
import mongoose, { Schema, Document } from "mongoose";

export interface IUser extends Document {
  email: string;
  password: string;
  birthdate: Date;
  gender: "male" | "female";
  createdAt: Date;
  emailVerified: boolean;
}

const UserSchema: Schema = new Schema({
  email: { type: String, required: true, lowercase: true, unique: true },
  password: { type: String, required: true },
  birthdate: { type: Date },
  gender: { type: String },
  createdAt: { type: Date, default: new Date() },
  emailVerified: { type: Boolean, default: false },
});

export const User = mongoose.model<IUser>("User", UserSchema);
```

## Validator

Schema de validation de l'utilisateur

```ts
import Joi from "joi";

export const userSchema = Joi.object({
  birthdate: Joi.date().timestamp().iso(),
  gender: Joi.string().valid("male", "female"),
}).options({ allowUnknown: true });
```

## Routes express

Intégration des routes utilisateur à express

```ts
import { Router } from "express";
import createError from "http-errors";
import { IUser, User } from "../db";
import { userSchema } from "../validator";
import { verifyAccessToken } from "../helpers";

export const userRoutes = Router();

userRoutes.get("/me", verifyAccessToken, async (req, res, next) => {
  try {
    const userId: string = req.body.decoded.aud;
    if (!userId) throw new createError.BadRequest();
    const user = await User.findById(userId, { password: 0 });
    if (!user) throw new createError.NotFound();
    return res.send(user);
  } catch (err) {
    next(err);
  }
});

userRoutes.patch("/update", verifyAccessToken, async (req, res, next) => {
  try {
    const userId: string = req.body.decoded.aud;
    if (!userId) throw new createError.BadRequest();
    const { birthdate, gender } = await userSchema.validateAsync(req.body);
    const user = await User.findById(userId, { password: 0 });
    if (!user) throw new createError.NotFound();
    user.birthdate = birthdate ?? user.birthdate;
    user.gender = gender ?? user.gender;
    await user.save();
    return res.send(user);
  } catch (err) {
    next(err);
  }
});

userRoutes.delete("/delete", verifyAccessToken, async (req, res, next) => {
  try {
    const userId: string = req.body.decoded.aud;
    if (!userId) throw new createError.BadRequest();
    const user = await User.findById(userId);
    if (!user) throw new createError.NotFound();
    await user.remove();
    return res.send({ message: "user removed" });
  } catch (err) {
    next(err);
  }
});
```
