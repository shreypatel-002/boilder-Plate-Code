1️⃣ Backend (Node + Express + MongoDB)

📂 backend/

npm init -y
npm install express mongoose cors bcryptjs jsonwebtoken dotenv

server.js

import express from "express";
import mongoose from "mongoose";
import dotenv from "dotenv";
import cors from "cors";

dotenv.config();
const app = express();

// Middlewares
app.use(express.json());
app.use(cors());

// MongoDB connect
mongoose.connect(process.env.MONGO_URL)
.then(() => console.log("✅ MongoDB connected"))
.catch(err => console.log(err));

// Routes
import authRoutes from "./routes/auth.js";
import crudRoutes from "./routes/crud.js";

app.use("/api/auth", authRoutes);
app.use("/api/items", crudRoutes);

app.listen(5000, () => console.log("🚀 Server running on port 5000"));

models/User.js

import mongoose from "mongoose";

const UserSchema = new mongoose.Schema({
firstName: String,
lastName: String,
email: { type: String, unique: true },
phone: String,
password: String,
avatar: String
});

export default mongoose.model("User", UserSchema);

models/Item.js (generic CRUD model)

import mongoose from "mongoose";

const ItemSchema = new mongoose.Schema({
title: String,
description: String,
createdBy: { type: mongoose.Schema.Types.ObjectId, ref: "User" }
}, { timestamps: true });

export default mongoose.model("Item", ItemSchema);

routes/auth.js

import express from "express";
import bcrypt from "bcryptjs";
import jwt from "jsonwebtoken";
import User from "../models/User.js";

const router = express.Router();

// Signup
router.post("/signup", async (req, res) => {
try {
const { firstName, lastName, email, phone, password } = req.body;
const hash = await bcrypt.hash(password, 10);
const newUser = await User.create({ firstName, lastName, email, phone, password: hash });
res.json(newUser);
} catch (err) {
res.status(500).json(err.message);
}
});

// Login
router.post("/login", async (req, res) => {
try {
const { email, password } = req.body;
const user = await User.findOne({ email });
if (!user) return res.status(400).json("User not found");

    const isMatch = await bcrypt.compare(password, user.password);
    if (!isMatch) return res.status(400).json("Invalid credentials");

    const token = jwt.sign({ id: user._id }, process.env.JWT_SECRET, { expiresIn: "1h" });
    res.json({ user, token });

} catch (err) {
res.status(500).json(err.message);
}
});

export default router;

routes/crud.js

import express from "express";
import Item from "../models/Item.js";

const router = express.Router();

// Create
router.post("/", async (req, res) => {
try {
const item = await Item.create(req.body);
res.json(item);
} catch (err) {
res.status(500).json(err.message);
}
});

// Read all
router.get("/", async (req, res) => {
try {
const items = await Item.find();
res.json(items);
} catch (err) {
res.status(500).json(err.message);
}
});

// Update
router.put("/:id", async (req, res) => {
try {
const item = await Item.findByIdAndUpdate(req.params.id, req.body, { new: true });
res.json(item);
} catch (err) {
res.status(500).json(err.message);
}
});

// Delete
router.delete("/:id", async (req, res) => {
try {
await Item.findByIdAndDelete(req.params.id);
res.json("Deleted");
} catch (err) {
res.status(500).json(err.message);
}
});

export default router;

📂 .env

MONGO_URL=mongodb://127.0.0.1:27017/practical_round
JWT_SECRET=supersecret

2️⃣ Frontend (React)

📂 frontend/

npx create-react-app frontend
cd frontend
npm install axios react-router-dom

src/App.js

import { BrowserRouter, Routes, Route } from "react-router-dom";
import Signup from "./pages/Signup";
import Login from "./pages/Login";
import Dashboard from "./pages/Dashboard";

function App() {
return (
<BrowserRouter>
<Routes>
<Route path="/signup" element={<Signup />} />
<Route path="/login" element={<Login />} />
<Route path="/dashboard" element={<Dashboard />} />
</Routes>
</BrowserRouter>
);
}

export default App;

src/pages/Signup.js

import { useState } from "react";
import axios from "axios";

export default function Signup() {
const [form, setForm] = useState({ firstName:"", lastName:"", email:"", phone:"", password:"" });

const handleChange = e => setForm({ ...form, [e.target.name]: e.target.value });

const handleSubmit = async () => {
await axios.post("http://localhost:5000/api/auth/signup", form);
alert("User Registered");
};

return (
<div>
<h2>Signup</h2>
<input name="firstName" placeholder="First Name" onChange={handleChange} />
<input name="lastName" placeholder="Last Name" onChange={handleChange} />
<input name="email" placeholder="Email" onChange={handleChange} />
<input name="phone" placeholder="Phone" onChange={handleChange} />
<input name="password" type="password" placeholder="Password" onChange={handleChange} />
<button onClick={handleSubmit}>Register</button>
</div>
);
}

src/pages/Login.js

import { useState } from "react";
import axios from "axios";

export default function Login() {
const [form, setForm] = useState({ email:"", password:"" });

const handleChange = e => setForm({ ...form, [e.target.name]: e.target.value });

const handleSubmit = async () => {
const res = await axios.post("http://localhost:5000/api/auth/login", form);
localStorage.setItem("user", JSON.stringify(res.data.user));
localStorage.setItem("token", res.data.token);
alert("Login Success");
window.location.href = "/dashboard";
};

return (
<div>
<h2>Login</h2>
<input name="email" placeholder="Email" onChange={handleChange} />
<input name="password" type="password" placeholder="Password" onChange={handleChange} />
<button onClick={handleSubmit}>Login</button>
</div>
);
}

src/pages/Dashboard.js

import { useEffect, useState } from "react";
import axios from "axios";

export default function Dashboard() {
const [items, setItems] = useState([]);
const [title, setTitle] = useState("");

const fetchItems = async () => {
const res = await axios.get("http://localhost:5000/api/items");
setItems(res.data);
};

const addItem = async () => {
await axios.post("http://localhost:5000/api/items", { title, description:"test" });
fetchItems();
};

useEffect(() => { fetchItems(); }, []);

return (
<div>
<h2>Dashboard</h2>
<input value={title} onChange={e => setTitle(e.target.value)} placeholder="New Item" />
<button onClick={addItem}>Add</button>
<ul>
{items.map(i => <li key={i._id}>{i.title}</li>)}
</ul>
</div>
);
}
