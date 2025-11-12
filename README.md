# mongo-devops-project
Design, deploy, and operate a highly-available, secure, observable MongoDB service for a sample web app using Infrastructure-as-Code, automation, CI/CD, backups, monitoring, and disaster recovery — all with reproducible artifacts and tests.

High-level architecture
Cloud: AWS (EC2 + VPC + Security Groups) — you can substitute any cloud or on-prem.
MongoDB deployment options: self-managed on VMs (systemd), Docker Compose (dev), and Kubernetes StatefulSet + PersistentVolumes (prod).
HA: Replica Set (3 nodes) + optional Sharding (for scale).
Automation: Terraform (infra) + Ansible (config) + Helm (K8s).
CI/CD: GitHub Actions (schemas/migrations, backup tests, deployment checks).
Monitoring: Prometheus + Grafana + MongoDB Exporter + Alertmanager.
Backup: mongodump (logical) + filesystem snapshots or mongodump to S3 + oplog-based PITR.
Security: TLS, SCRAM auth, least-privilege IAM (for cloud), network policies, key management (KMS).
Observability: logs shipped to ELK or Loki, metrics, dashboards, runbooks and SLA reporting.Milestone 0 — Prep (deliverables: Git repo skeleton, accounts)

Create a Git repo (e.g., mongo-devops-project) with folders: infra/, ansible/, k8s/, docker/, ci/, monitoring/, docs/.
Get AWS account or local VM environment (VirtualBox/Vagrant) for testing.
Install CLI tools locally: terraform, ansible, docker, kubectl, helm, mongo shell, awscli.
Deliverable: README with project overview and prerequisites.


🧩 Milestone 0 — Environment Setup (Detailed Guide)

Objective: Set up a fully working DevOps + MongoDB environment on your local or cloud machine, so you can start developing and deploying your app later.

PHASE 1 — System Preparation
🧠 Step 1: Decide where to work

You can use any of these environments:
Option A: Local system (Windows, macOS, or Linux)
Option B: Virtual Machine (e.g., Ubuntu on VirtualBox or VMware)
Option C: Cloud instance (AWS EC2, GCP VM, Azure VM)

👉 For best DevOps practice, choose Ubuntu 22.04 LTS (since it’s common in production).

🧰 Step 2: Update and Upgrade your system

Run this first to ensure your system is clean and up to date:

sudo apt update -y && sudo apt upgrade -y
sudo apt install curl wget git unzip -y

🧑‍💻 Step 3: Install Git (Version Control)

Git lets you manage code versions and collaborate.

sudo apt install git -y
git --version


Configure your identity:

git config --global user.name "YourName"
git config --global user.email "yourname@example.com"


✅ Verify:

git config --list

PHASE 2 — Install MongoDB
🍃 Step 4: Install MongoDB (Community Edition)
(A) Import the MongoDB GPG key:
curl -fsSL https://pgp.mongodb.com/server-7.0.asc | \
sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor

(B) Add the MongoDB repository:
echo "deb [signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg] \
https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | \
sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

(C) Install MongoDB:
sudo apt update
sudo apt install -y mongodb-org

(D) Start and enable MongoDB service:
sudo systemctl start mongod
sudo systemctl enable mongod
sudo systemctl status mongod


✅ You should see:

Active: active (running)

(E) Test MongoDB shell:
mongosh


Inside the shell, test it:

show dbs
use test
db.greetings.insertOne({message: "Hello MongoDB!"})
db.greetings.find()


Exit: exit

PHASE 3 — Install Node.js (Backend Runtime)

Your backend service will use Node.js to interact with MongoDB.

Step 5: Install Node.js + npm
sudo apt install -y nodejs npm
node -v
npm -v


If you want the latest Node.js:

curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs


✅ Verify:

node -v
npm -v

PHASE 4 — Install Docker (Containerization)

DevOps means containers and automation — Docker is key here.

🐳 Step 6: Install Docker Engine
sudo apt update
sudo apt install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg


Add repository:

echo \
  "deb [arch=$(dpkg --print-architecture) \
  signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null


Install Docker:

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin


Start & enable:

sudo systemctl start docker
sudo systemctl enable docker


Verify:

sudo docker run hello-world


✅ Expected output:

Hello from Docker!

Add your user to docker group (so you don’t need sudo every time):

sudo usermod -aG docker $USER


Then log out and back in.

PHASE 5 — Install VS Code (IDE)
🧩 Step 7: Install VS Code
sudo apt install wget gpg -y
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg
sudo install -o root -g root -m 644 packages.microsoft.gpg /usr/share/keyrings/
sudo sh -c 'echo "deb [arch=amd64 signed-by=/usr/share/keyrings/packages.microsoft.gpg] \
https://packages.microsoft.com/repos/code stable main" > /etc/apt/sources.list.d/vscode.list'
sudo apt install apt-transport-https
sudo apt update
sudo apt install code


✅ Launch:

code

PHASE 6 — Install Postman (API Testing)

You’ll need Postman to test your API endpoints.

sudo snap install postman


✅ Launch via:

postman &

PHASE 7 — Verify Everything Works Together
🧪 Step 8: Sanity Check

Run these to confirm all tools are ready:

Tool	Command	Expected
MongoDB	mongosh	connects successfully
Node.js	node -v	shows version
npm	npm -v	shows version
Git	git --version	version displayed
Docker	docker ps	no errors
VS Code	code	opens IDE
Postman	app opens	test API later
PHASE 8 — Optional: Setup GitHub Repository
Step 9: Create a GitHub repo for your project

Go to GitHub.com

Create a new repository → name it ecommerce-mongodb-devops

Copy the URL, e.g.
https://github.com/yourusername/ecommerce-mongodb-devops.git

In terminal:

mkdir ecommerce-mongodb-devops
cd ecommerce-mongodb-devops
git init
git remote add origin https://github.com/yourusername/ecommerce-mongodb-devops.git


You’ll use this later for CI/CD.🧩 Milestone 1 — Backend Application Development

Objective: Create a RESTful API backend for our “E-commerce Catalog Management System” that connects to MongoDB.
Stack: Node.js + Express.js + MongoDB (Mongoose ODM)

⚙️ PHASE 1 — Project Setup
🧠 Step 1: Create a project folder
cd ~
mkdir ecommerce-backend
cd ecommerce-backend

🧰 Step 2: Initialize Node.js project

This will create a package.json file that keeps track of dependencies and scripts.

npm init -y


✅ This creates a file:

package.json


Open it:

code .


You’ll see a JSON structure — it’s like metadata for your Node project.

📦 Step 3: Install Required Packages
Backend Core Dependencies:
npm install express mongoose dotenv cors

Dev Dependencies (for nodemon auto-reload):
npm install --save-dev nodemon

Package	Purpose
express	Web server framework for creating APIs
mongoose	ODM (Object Data Modeling) for MongoDB
dotenv	Loads environment variables from .env file
cors	Enables cross-origin requests
nodemon	Automatically restarts server when files change
⚙️ Step 4: Update package.json scripts

In your package.json, add this inside the "scripts" section:

"scripts": {
  "start": "node server.js",
  "dev": "nodemon server.js"
}


This lets you run:

npm run start → production mode

npm run dev → development mode (auto-reloads)

🧱 PHASE 2 — Folder Structure

Create this structure:

ecommerce-backend/
│
├── server.js
├── .env
├── package.json
│
├── config/
│   └── db.js
│
├── models/
│   └── Product.js
│
├── routes/
│   └── productRoutes.js
│
└── controllers/
    └── productController.js

📡 PHASE 3 — Connect to MongoDB
🧩 Step 5: Create DB Configuration

Create file:
config/db.js

import mongoose from "mongoose";

const connectDB = async () => {
  try {
    const conn = await mongoose.connect(process.env.MONGO_URI);
    console.log(`✅ MongoDB Connected: ${conn.connection.host}`);
  } catch (error) {
    console.error(`❌ Error: ${error.message}`);
    process.exit(1);
  }
};

export default connectDB;

🔐 Step 6: Setup Environment Variables

Create file:
.env

PORT=5000
MONGO_URI=mongodb://localhost:27017/ecommerce

🚀 Step 7: Create Server Entry File

File: server.js

import express from "express";
import dotenv from "dotenv";
import cors from "cors";
import connectDB from "./config/db.js";
import productRoutes from "./routes/productRoutes.js";

dotenv.config();  // Load .env
connectDB();       // Connect to MongoDB

const app = express();
app.use(cors());
app.use(express.json()); // Parse JSON body

app.get("/", (req, res) => {
  res.send("E-commerce Backend is running...");
});

app.use("/api/products", productRoutes);

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`🚀 Server running on port ${PORT}`));

🧩 PHASE 4 — Define MongoDB Model
🧱 Step 8: Create Product model

File: models/Product.js

import mongoose from "mongoose";

const productSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, "Product name is required"]
  },
  price: {
    type: Number,
    required: [true, "Product price is required"]
  },
  category: {
    type: String,
    required: true
  },
  inStock: {
    type: Boolean,
    default: true
  }
}, {
  timestamps: true
});

const Product = mongoose.model("Product", productSchema);
export default Product;

🧠 PHASE 5 — Create Controllers
Step 9: Create controllers/productController.js
import Product from "../models/Product.js";

// @desc Get all products
// @route GET /api/products
export const getProducts = async (req, res) => {
  try {
    const products = await Product.find();
    res.json(products);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

// @desc Create new product
// @route POST /api/products
export const createProduct = async (req, res) => {
  const { name, price, category } = req.body;

  try {
    const newProduct = new Product({ name, price, category });
    await newProduct.save();
    res.status(201).json(newProduct);
  } catch (error) {
    res.status(400).json({ message: error.message });
  }
};

🚏 PHASE 6 — Create Routes
Step 10: Create routes/productRoutes.js
import express from "express";
import { getProducts, createProduct } from "../controllers/productController.js";

const router = express.Router();

router.get("/", getProducts);
router.post("/", createProduct);

export default router;

🧪 PHASE 7 — Test the Backend
Step 11: Start Server
npm run dev


✅ Output:

✅ MongoDB Connected: localhost
🚀 Server running on port 5000

Step 12: Test with Postman
1️⃣ GET all products:

Method: GET

URL: http://localhost:5000/api/products

Response:

[]

2️⃣ POST new product:

Method: POST

URL: http://localhost:5000/api/products

Body → JSON:

{
  "name": "Wireless Mouse",
  "price": 799,
  "category": "Electronics"
}


Response:

{
  "_id": "67447a2434c912...",
  "name": "Wireless Mouse",
  "price": 799,
  "category": "Electronics",
  "inStock": true,
  "createdAt": "2025-11-12T08:30:00Z"
}

3️⃣ GET again:

Now it should return the product you added.

🧾 PHASE 8 — Version Control
Step 13: Commit and Push
git add .
git commit -m "Initial backend setup with MongoDB integration"
git push origin main

✅ Milestone 1 Summary

You’ve now built a fully functional backend service that:

Uses Node.js + Express to serve APIs

Stores and retrieves data from MongoDB

Has modular code (routes, controllers, models)

Runs in dev mode via nodemon


