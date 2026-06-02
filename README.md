# 💰 Full-Stack Expense Tracker Dashboard

A premium, resume-worthy Expense Tracker Dashboard application built with a **React (Vite)** frontend and an **Express.js** REST API backend. It features a layered backend architecture designed for seamless MongoDB migration and a fully responsive glassmorphism dark-mode interface.

---

## 🏗️ Project Architecture

This application is built with a strong focus on **Separation of Concerns** and **Scalability**.

### Frontend (client/)
- **State Management**: Built-in React Context (`ExpenseContext`, `NotificationContext`) combined with custom hooks (`useExpenses`, `useDebounce`) to manage global application state.
- **Service Layer**: Dedicated API clients (`api.js`, `expenseService.js`) using Axios. Page components never call Axios directly.
- **Routing**: Client-side routing managed by React Router v6.
- **Design & Styling**: Custom Vanilla CSS with Design Tokens (`index.css`), providing fluid micro-animations, glassmorphism cards, color-coded categories, and responsive grids.

### Backend (server/)
- **Routes Layer** (`expenseRoutes.js`): Pure declarative mapping of endpoints to controller actions.
- **Controller Layer** (`expenseController.js`): Translates requests and responses, handles HTTP codes, and delegates to the Service.
- **Service Layer** (`expenseService.js`): Contains validation logic, dashboard calculations, and search/filtering/sorting algorithms.
- **Repository Layer** (`expenseRepository.js`): Abstracted data access. The *only* layer aware of *how* and *where* data is stored.
- **Data Store** (`expenseStore.js`): In-memory data store using mutable arrays. This isolated structure makes database migration extremely clean.
- **Middleware**: Built-in HTTP log tracker and global centralized exception catching handler.

---

## 🛠️ Tech Stack & Dependencies

### Frontend (Client)
- **React**: Functional components and hooks.
- **Vite**: Fast development HMR and production bundle builds.
- **React Router DOM**: Client-side page navigation.
- **Axios**: Promised-based client for HTTP communication.

### Backend (Server)
- **Node.js & Express.js**: Server engine and API router framework.
- **CORS**: Cross-origin policy sharing headers.
- **ES Modules**: Standardized import/export syntax.

---

## 🚀 Getting Started

### Prerequisites
- Node.js (v18 or higher recommended)
- npm (Node Package Manager)

### Installation
1. Clone or navigate into the workspace directory:
   ```bash
   cd ExpenseTracker
   ```

2. Install dependencies for the server:
   ```bash
   cd server
   npm install
   ```

3. Install dependencies for the client:
   ```bash
   cd ../client
   npm install
   ```

### Running the Application

To run the application, you will need to open two terminal windows (one for the backend and one for the frontend).

**1. Start the API Server (Backend)**
```bash
cd server
npm run dev
```
The server will start on [http://localhost:5000](http://localhost:5000). You can check its health at `/api/health`.

**2. Start the Vite App (Frontend)**
```bash
cd client
npm run dev
```
The client will start on [http://localhost:5173](http://localhost:5173). The Vite dev server will automatically proxy requests to the backend server.

---

## 📂 Project Structure

```
ExpenseTracker/
├── client/                          # React frontend
│   ├── src/
│   │   ├── components/
│   │   │   ├── common/              # LoadingSpinner, EmptyState
│   │   │   ├── dashboard/           # StatCard, CategoryChart, SpendingOverview
│   │   │   └── layout/              # Navbar, PageLayout
│   │   ├── context/                 # ExpenseContext, NotificationContext
│   │   ├── hooks/                   # useExpenses, useDebounce
│   │   ├── pages/                   # DashboardPage, ExpensesPage, Add/Edit Pages
│   │   ├── router/                  # AppRouter configuration
│   │   ├── services/                # api Axios configs, expense API calls
│   │   └── utils/                   # constants, formatters, validators
│   └── package.json
│
├── server/                          # Express.js backend
│   ├── src/
│   │   ├── routes/                  # Express route definitions
│   │   ├── controllers/             # Controller handlers
│   │   ├── services/                # Business logic & validations
│   │   ├── repositories/            # Data access abstractions
│   │   ├── data/                    # In-memory arrays (M1/M2 database)
│   │   └── middleware/              # logger, errorHandler
│   ├── app.js                   # Application middleware configurations
│   ├── server.js                # Port listening script
│   └── package.json
```

---

## 📝 Expense Data Schema

Each expense item consists of:
- `id` (String): Prefix-based unique identifier (`exp_...`).
- `title` (String): Required, 1-100 characters.
- `description` (String): Optional, max 500 characters.
- `amount` (Number): Required, positive float.
- `category` (String): Must be one of: `Food`, `Travel`, `Shopping`, `Bills`, `Entertainment`, `Other`.
- `date` (String): ISO date string (`YYYY-MM-DD`).
- `createdAt` (String): ISO timestamp when logged.
- `updatedAt` (String): ISO timestamp when modified.
