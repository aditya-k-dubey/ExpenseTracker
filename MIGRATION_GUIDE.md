# 🍃 MongoDB Migration Guide

Because this application was built using a **Layered Architecture** with a decoupled **Repository Layer**, migrating the storage engine from in-memory JavaScript arrays to **MongoDB** requires minimal changes. The rest of the server (routes, controllers, and services) and the entire frontend client do not need any modifications.

Here is the step-by-step guide to migrate the app.

---

## Step 1: Install Mongoose & Dotenv

Navigate into your `server/` folder and install the required packages:

```bash
cd server
npm install mongoose dotenv
```

- **Mongoose**: A library that provides schema validation and queries for MongoDB.
- **Dotenv**: Loads configuration/connection settings from a `.env` file into `process.env`.

---

## Step 2: Configure Environment Variables

Create a file named `.env` in the root of your `server/` directory:

```env
PORT=5000
MONGODB_URI=mongodb://localhost:27017/expensetracker
```

*(Note: Replace the URI with your MongoDB Atlas connection string when deploying to production).*

---

## Step 3: Define the Mongoose Schema

Create a new file named `expenseModel.js` in `server/src/models/`:

```javascript
import mongoose from "mongoose";

const ExpenseSchema = new mongoose.Schema(
  {
    title: {
      type: String,
      required: [true, "Title is required"],
      trim: true,
      maxlength: [100, "Title must be 100 characters or less"],
    },
    description: {
      type: String,
      trim: true,
      maxlength: [500, "Description must be 500 characters or less"],
      default: "",
    },
    amount: {
      type: Number,
      required: [true, "Amount is required"],
      min: [0.01, "Amount must be a positive number"],
    },
    category: {
      type: String,
      required: [true, "Category is required"],
      enum: {
        values: ["Food", "Travel", "Shopping", "Bills", "Entertainment", "Other"],
        message: "{VALUE} is not a valid category",
      },
    },
    date: {
      type: String,
      required: [true, "Date is required"],
    },
  },
  {
    timestamps: true, // Automatically manages createdAt and updatedAt fields
    toJSON: {
      virtuals: true,
      transform: (doc, ret) => {
        ret.id = ret._id.toString(); // Map _id to id so frontend matches
        delete ret._id;
        delete ret.__v;
        return ret;
      },
    },
  }
);

export default mongoose.model("Expense", ExpenseSchema);
```

---

## Step 4: Update the Repository Layer

Replace the code in `server/src/repositories/expenseRepository.js` with this Mongoose version:

```javascript
import Expense from "../models/expenseModel.js";

/**
 * Retrieves all expenses from MongoDB.
 */
export async function findAll() {
  return await Expense.find().sort({ date: -1 });
}

/**
 * Finds a single expense by ID in MongoDB.
 */
export async function findById(id) {
  return await Expense.findById(id);
}

/**
 * Creates a new expense document.
 */
export async function create(expenseData) {
  const expense = new Expense(expenseData);
  return await expense.save();
}

/**
 * Updates an existing expense in MongoDB.
 */
export async function update(id, updateData) {
  return await Expense.findByIdAndUpdate(
    id,
    { $set: updateData },
    { new: true, runValidators: true }
  );
}

/**
 * Removes an expense from MongoDB by ID.
 */
export async function remove(id) {
  return await Expense.findByIdAndDelete(id);
}
```

---

## Step 5: Adjust the Service Layer (Add `async/await`)

Since MongoDB queries are asynchronous, they return Promises. You will need to add the `async` keyword to your service functions in `server/src/services/expenseService.js` and use `await` when calling repository methods.

For example, update `getExpenseById`:

```javascript
export async function getExpenseById(id) {
  const expense = await expenseRepository.findById(id); // Added await

  if (!expense) {
    const error = new Error(`Expense with ID "${id}" not found`);
    error.statusCode = 404;
    throw error;
  }

  return expense;
}
```

Do the same in the Controller layer (`expenseController.js`) to `await` calls to the Service layer!

---

## Step 6: Connect to MongoDB on Startup

In `server/server.js`, import `dotenv` and establish the connection:

```javascript
import dotenv from "dotenv";
import mongoose from "mongoose";
import app from "./app.js";

// Load configuration
dotenv.config();

const PORT = process.env.PORT || 5000;
const MONGODB_URI = process.env.MONGODB_URI;

// Connect to MongoDB
mongoose
  .connect(MONGODB_URI)
  .then(() => {
    console.log("🍃 Connected to MongoDB successfully.");
    
    // Start listening only after DB connection succeeds
    app.listen(PORT, () => {
      console.log(`💰 Server is running on port ${PORT}`);
    });
  })
  .catch((err) => {
    console.error("✕ MongoDB connection error:", err.message);
    process.exit(1);
  });
```
