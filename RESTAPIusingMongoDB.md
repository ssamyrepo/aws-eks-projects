Here’s a **step-by-step guide** to implement the REST API project  on an **Amazon EC2 instance**. This guide includes all the actual steps to set up the environment, deploy the application, and test the CRUD operations.

---

## Prerequisites

1. **AWS Account**: You need an AWS account to create an EC2 instance.
2. **EC2 Instance**: Launch an EC2 instance with Ubuntu or Amazon Linux AMI.
3. **MongoDB Atlas**: Sign up for a free MongoDB Atlas account to host your database in the cloud.
4. **SSH Access**: Ensure you have SSH access to the EC2 instance.

---

## Step 1: Set Up MongoDB Atlas

1. **Sign Up for MongoDB Atlas**:
   - Go to [MongoDB Atlas](https://www.mongodb.com/cloud/atlas) and create a free account.
2. **Create a Cluster**:
   - Create a free-tier cluster.
3. **Whitelist IP Address**:
   - Whitelist the public IP of your EC2 instance in MongoDB Atlas to allow access.
4. **Get Connection String**:
   - Get the connection string for your MongoDB cluster. It will look like this:
     ```
     mongodb+srv://<username>:<password>@<cluster-url>/<dbname>?retryWrites=true&w=majority
     ```

---

## Step 2: Set Up the EC2 Instance

1. **Connect to the EC2 Instance**:
   ```bash
   ssh -i /path/to/your-key.pem ec2-user@<public-ip-of-ec2>
   ```

2. **Install Node.js and npm**:
   - For Ubuntu:
     ```bash
     sudo apt update
     sudo apt install nodejs npm
     ```
   - For Amazon Linux:
     ```bash
     sudo yum update
     sudo yum install nodejs npm
     ```

3. **Install MongoDB Driver**:
   - Install the MongoDB Node.js driver globally:
     ```bash
     sudo npm install -g mongodb
     ```

---

## Step 3: Create the REST API Application

1. **Create a Project Directory**:
   ```bash
   mkdir rest-api
   cd rest-api
   ```

2. **Initialize a Node.js Project**:
   ```bash
   npm init -y
   ```

3. **Install Required Packages**:
   ```bash
   npm install express mongodb body-parser
   ```

4. **Create the Application File**:
   - Create a file named `app.js`:
     ```bash
     touch app.js
     ```

5. **Write the REST API Code**:
   - Open `app.js` and add the following code:

```javascript
const express = require('express');
const { MongoClient } = require('mongodb');
const bodyParser = require('body-parser');

const app = express();
const port = 5000;

// MongoDB connection string
const uri = "mongodb+srv://<username>:<password>@<cluster-url>/<dbname>?retryWrites=true&w=majority";
const client = new MongoClient(uri, { useNewUrlParser: true, useUnifiedTopology: true });

// Middleware to parse JSON
app.use(bodyParser.json());

// Connect to MongoDB
async function connectToDB() {
  try {
    await client.connect();
    console.log("Connected to MongoDB");
  } catch (err) {
    console.error("Error connecting to MongoDB:", err);
  }
}

connectToDB();

// Create a new document
app.post('/api/books', async (req, res) => {
  const db = client.db('library');
  const collection = db.collection('books');
  const result = await collection.insertOne(req.body);
  res.status(201).json({ message: "Book created", id: result.insertedId });
});

// Read all documents
app.get('/api/books', async (req, res) => {
  const db = client.db('library');
  const collection = db.collection('books');
  const books = await collection.find({}).toArray();
  res.status(200).json(books);
});

// Read a single document by ID
app.get('/api/books/:id', async (req, res) => {
  const db = client.db('library');
  const collection = db.collection('books');
  const book = await collection.findOne({ _id: req.params.id });
  if (book) {
    res.status(200).json(book);
  } else {
    res.status(404).json({ message: "Book not found" });
  }
});

// Update a document by ID
app.put('/api/books/:id', async (req, res) => {
  const db = client.db('library');
  const collection = db.collection('books');
  const result = await collection.updateOne({ _id: req.params.id }, { $set: req.body });
  if (result.modifiedCount > 0) {
    res.status(200).json({ message: "Book updated" });
  } else {
    res.status(404).json({ message: "Book not found" });
  }
});

// Delete a document by ID
app.delete('/api/books/:id', async (req, res) => {
  const db = client.db('library');
  const collection = db.collection('books');
  const result = await collection.deleteOne({ _id: req.params.id });
  if (result.deletedCount > 0) {
    res.status(200).json({ message: "Book deleted" });
  } else {
    res.status(404).json({ message: "Book not found" });
  }
});

// Start the server
app.listen(port, () => {
  console.log(`Server is running on http://localhost:${port}`);
});
```

---

## Step 4: Run the Application

1. **Start the Server**:
   ```bash
   node app.js
   ```

2. **Test the API**:
   - Use `curl` or Postman to test the API endpoints.

---

## Step 5: Test CRUD Operations

### 1. **Create a Book**
   ```bash
   curl -X POST http://<public-ip-of-ec2>:5000/api/books \
     -H "Content-Type: application/json" \
     -d '{"title": "Introduction to RESTful APIs", "author": "John Doe", "pages": 120}'
   ```

### 2. **Read All Books**
   ```bash
   curl http://<public-ip-of-ec2>:5000/api/books
   ```

### 3. **Read a Single Book**
   ```bash
   curl http://<public-ip-of-ec2>:5000/api/books/<book-id>
   ```

### 4. **Update a Book**
   ```bash
   curl -X PUT http://<public-ip-of-ec2>:5000/api/books/<book-id> \
     -H "Content-Type: application/json" \
     -d '{"title": "Updated Title"}'
   ```

### 5. **Delete a Book**
   ```bash
   curl -X DELETE http://<public-ip-of-ec2>:5000/api/books/<book-id>
   ```

---

## Step 6: Clean Up

1. **Stop the Application**:
   - Press `Ctrl+C` to stop the Node.js server.

2. **Terminate the EC2 Instance**:
   - Go to the AWS EC2 console and terminate the instance if no longer needed.

