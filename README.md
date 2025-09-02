mern-boilerplate/

│

├── backend/

│ ├── models/

│ │ └── User.js

│ ├── routes/

│ │ └── auth.js

│ ├── server.js

│ └── .env

│

├── frontend/

│ ├── src/

│ │ ├── App.js

│ │ ├── api.js

│ │ ├── components/

│ │ │ ├── Login.js

│ │ │ └── Signup.js

│ │ └── index.js

│ └── package.json

│

└── README.md

🛠 Installation

1️⃣ Clone the repo

bash

Copy code

git clone https://github.com/your-username/mern-boilerplate.git

cd mern-boilerplate

2️⃣ Backend Setup

bash

Copy code

cd backend

npm install

Create a .env file in backend/:

env

Copy code

PORT=5000

MONGO_URI=mongodb://127.0.0.1:27017/mernboiler

JWT_SECRET=yourSecretKey

Run the backend:

bash

Copy code

npm start

3️⃣ Frontend Setup

bash

Copy code

cd frontend

npm install

Update src/api.js with your backend URL:

javascript

Copy code

import axios from "axios";

const API = axios.create({

baseURL: "http://localhost:5000",

withCredentials: true,

});

export default API;

Run the frontend:

bash

Copy code

npm start

📌 Backend Code

server.js

javascript

Copy code

const express = require("express");

const mongoose = require("mongoose");

const cors = require("cors");

require("dotenv").config();

const authRoutes = require("./routes/auth");

const app = express();

app.use(express.json());

app.use(cors());

app.use("/auth", authRoutes);

mongoose.connect(process.env.MONGO_URI)

.then(() => console.log("MongoDB Connected"))

.catch((err) => console.log(err));

app.listen(process.env.PORT, () =>

console.log(\`Server running on port ${process.env.PORT}\`)

);

models/User.js

javascript

Copy code

const mongoose = require("mongoose");

const userSchema = new mongoose.Schema({

name: String,

email: { type: String, unique: true },

password: String,

});

module.exports = mongoose.model("User", userSchema);

routes/auth.js

javascript

Copy code

const express = require("express");

const bcrypt = require("bcryptjs");

const jwt = require("jsonwebtoken");

const User = require("../models/User");

const router = express.Router();

// Signup

router.post("/signup", async (req, res) => {

const { name, email, password } = req.body;

try {

const hashedPassword = await bcrypt.hash(password, 10);

const newUser = new User({ name, email, password: hashedPassword });

await newUser.save();

res.json({ message: "User created successfully" });

} catch (err) {

res.status(400).json({ error: err.message });

}

});

// Login

router.post("/login", async (req, res) => {

const { email, password } = req.body;

try {

const user = await User.findOne({ email });

if (!user) return res.status(400).json({ error: "User not found" });

const isMatch = await bcrypt.compare(password, user.password);

if (!isMatch) return res.status(400).json({ error: "Invalid credentials" });

const token = jwt.sign({ id: user.\_id }, process.env.JWT_SECRET, {

expiresIn: "1h",

});

res.json({ token, user });

} catch (err) {

res.status(500).json({ error: err.message });

}

});

module.exports = router;

📌 Frontend Code

src/App.js

javascript

Copy code

import { useState } from "react";

import Signup from "./components/Signup";

import Login from "./components/Login";

function App() {

const \[user, setUser\] = useState(null);

return (

# MERN Boilerplate

{user ?

## Welcome {user.name}

: <>

}

);

}

export default App;

src/components/Signup.js

javascript

Copy code

import { useState } from "react";

import API from "../api";

export default function Signup({ setUser }) {

const \[form, setForm\] = useState({ name: "", email: "", password: "" });

const handleSubmit = async (e) => {

e.preventDefault();

await API.post("/auth/signup", form);

alert("Signup successful! Now login.");

};

return (

setForm({ ...form, name: e.target.value })} />

setForm({ ...form, email: e.target.value })} />

setForm({ ...form, password: e.target.value })} />

Signup

);

}

src/components/Login.js

javascript

Copy code

import { useState } from "react";

import API from "../api";

export default function Login({ setUser }) {

const \[form, setForm\] = useState({ email: "", password: "" });

const handleSubmit = async (e) => {

e.preventDefault();

const res = await API.post("/auth/login", form);

localStorage.setItem("token", res.data.token);

setUser(res.data.user);

};

return (

setForm({ ...form, email: e.target.value })} />

setForm({ ...form, password: e.target.value })} />

Login

);

}
