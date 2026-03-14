# 🛒 MarketNext

A full-stack MERN ecommerce platform with role-based access for Brands (Sellers) and Customers, featuring JWT authentication and cloud image storage.

🔗 **Live Demo:** 
      backend: [https://marketnext-backend.vercel.app/]
      frontend: [https://marketnext-frontend.vercel.app/]
      admin: [https://marketnext-admin.vercel.app/]
---

## 🧰 Tech Stack

| Layer | Technology | Purpose |
|---|---|---|
| Frontend | React.js | Component-based UI for both the storefront and brand dashboard |
| Backend | Node.js + Express.js | REST API server with clean route/controller/middleware structure |
| Database | MongoDB + Mongoose | Flexible document storage with schema validation |
| Auth | JWT (Access + Refresh Tokens) | Secure, stateless authentication with silent session renewal |
| Image Storage | Cloudinary / AWS S3 | Cloud-hosted image delivery so MongoDB only stores URLs |
| Deployment | Vercel | Zero-config CI/CD with GitHub integration |

---

## 📁 Project Structure


MarketNext/
├── backend/
│   ├── config/
│   │   ├── db.js                 # MongoDB connection setup
│   │   └── cloudinary.js         # Cloudinary SDK initialization
│   ├── controllers/
│   │   ├── authController.js     # Signup, login, logout, token refresh logic
│   │   └── productController.js  # Product CRUD, soft delete, dashboard stats
│   ├── middleware/
│   │   ├── authMiddleware.js     # Verifies JWT access token on protected routes
│   │   └── roleMiddleware.js     # Restricts routes by user role (brand/customer)
│   ├── models/
│   │   ├── User.js               # User schema — stores role, hashed password, refresh token
│   │   └── Product.js            # Product schema — supports draft, published, archived states
│   ├── routes/
│   │   ├── authRoutes.js         # Maps /api/auth endpoints to controllers
│   │   └── productRoutes.js      # Maps /api/products endpoints with middleware guards
│   └── server.js                 # App entry point — middleware, routes, DB connection
├── frontend/
│   ├── components/               # Reusable UI elements (navbar, cards, filters, pagination)
│   ├── pages/                    # Route-level views (marketplace, product detail)
│   ├── context/                  # Global auth state via React Context
│   └── App.jsx                   # Router setup and context providers
├── admin/
│   ├── components/               # Brand dashboard UI components
│   ├── pages/                    # Dashboard pages (product list, create, edit)
│   └── App.jsx                   # Admin router setup
├── .gitignore
├── package.json
└── package-lock.json


---

## ✨ Features

### 🔐 Authentication & Authorization
- Signup & Login with **role selection** (Brand or Customer)
- Passwords hashed with **bcrypt** before storage
- **JWT Access Token** — short-lived (15m), sent in response body
- **Refresh Token** — long-lived (7d), stored in `httpOnly` cookie to prevent JS access
- Silent token renewal via `/api/auth/refresh` — no re-login needed
- Logout clears the refresh token from DB, fully invalidating the session
- `authMiddleware` blocks unauthenticated requests with `401`
- `roleMiddleware` blocks wrong-role requests with `403`

### 🏪 Brand (Seller) Capabilities
- Access a dedicated **admin dashboard** separate from the customer app
- Create products as **draft** (hidden) or **published** (live in marketplace)
- Upload **multiple product images** — stored in Cloudinary, URLs saved in DB
- Edit only **their own products** — server verifies ownership on every mutation
- **Soft delete** sets status to `archived` — data preserved, hidden from marketplace
- Dashboard summary shows total, published, and archived product counts

### 🛍️ Customer Capabilities
- Browse all published products on the marketplace
- **Search** by product name and **filter** by category — processed server-side
- **Server-side pagination** — only the requested page is fetched, not the full catalog
- View full product detail pages with images, description, price, and category
- Strictly **read-only** — role middleware blocks all write operations at the API level

---

## 🚀 Getting Started

### Prerequisites
- Node.js v18+
- MongoDB (local or Atlas)
- Cloudinary or AWS S3 account

### 1. Clone the repository

bash
git clone https://github.com/madhu-ratnam/Ecommerce-app.git
cd Ecommerce-app


### 2. Set up environment variables

Create a `.env` file in the `backend/` directory:

env
# Server
PORT=5000
NODE_ENV=development

# MongoDB
MONGO_URI=your_mongodb_connection_string

# JWT
ACCESS_TOKEN_SECRET=your_access_token_secret
REFRESH_TOKEN_SECRET=your_refresh_token_secret
ACCESS_TOKEN_EXPIRY=15m
REFRESH_TOKEN_EXPIRY=7d

# Cloudinary
CLOUDINARY_CLOUD_NAME=your_cloud_name
CLOUDINARY_API_KEY=your_api_key
CLOUDINARY_API_SECRET=your_api_secret


### 3. Install dependencies

bash
# Backend
cd backend && npm install

# Frontend
cd ../frontend && npm install

# Admin
cd ../admin && npm install


### 4. Run the development servers

bash
# Backend
cd backend && npm run dev

# Frontend
cd frontend && npm run dev

# Admin
cd admin && npm run dev


---

## 🔌 API Overview

### Auth Routes — `/api/auth`

| Method | Endpoint | Description | Access |
|--------|----------|-------------|--------|
| POST | `/signup` | Register a new user with a role | Public |
| POST | `/login` | Authenticate and receive tokens | Public |
| POST | `/refresh` | Get a new access token via cookie | Public |
| POST | `/logout` | Invalidate refresh token and clear cookie | Auth |

> **`/signup`** — Hashes password and saves user with selected role; role cannot be changed later.
> **`/login`** — Returns access token in body and sets refresh token as `httpOnly` cookie.
> **`/refresh`** — Validates cookie refresh token against DB and issues a fresh access token.
> **`/logout`** — Clears refresh token from DB and expires the cookie on the client.

---

### Product Routes — `/api/products`

| Method | Endpoint | Description | Access |
|--------|----------|-------------|--------|
| GET | `/` | List published products with search, filter & pagination | Public |
| GET | `/:id` | Get a single product's full details | Public |
| POST | `/` | Create a new product with image uploads | Brand only |
| PUT | `/:id` | Update own product fields or status | Brand only |
| DELETE | `/:id` | Soft delete (archive) own product | Brand only |

> **`GET /`** — Accepts `search`, `category`, `page`, `limit` query params; returns only `published` products.
> **`GET /:id`** — Returns full product document; `404` if not found or archived.
> **`POST /`** — Uploads images to Cloudinary; attaches authenticated user as `owner` server-side.
> **`PUT /:id`** — Verifies `owner` matches authenticated user before applying updates; `403` if not.
> **`DELETE /:id`** — Sets `status: "archived"` instead of removing the document from DB.

---

### Dashboard Routes — `/api/dashboard`

| Method | Endpoint | Description | Access |
|--------|----------|-------------|--------|
| GET | /summary | Returns product counts grouped by status | Brand only |

> **GET /summary** — Queries only the authenticated brand's products and returns totalProducts, publishedCount, and archivedCount.

---

## 🗄️ MongoDB Schema Design

### User
js
{
  name: String,
  email: String (unique),
  password: String (hashed),
  role: { type: String, enum: ["brand", "customer"] },
  refreshToken: String,
  createdAt: Date
}


### Product
js
{
  title: String,
  description: String,
  category: String,
  price: Number,
  images: [String],         // Cloudinary/S3 URLs
  status: { type: String, enum: ["draft", "published", "archived"] },
  owner: { type: ObjectId, ref: "User" },
  createdAt: Date,
  updatedAt: Date
}


---

## 🛡️ Security Notes

- All secrets stored in **environment variables** — never hardcoded
- Refresh token in **`httpOnly` cookie** — inaccessible to browser JavaScript
- **Ownership check** on every product mutation — brands can't touch each other's listings
- **Role middleware** blocks customers from all write operations at the API level

---

## 📦 Deployment

1. Push code to GitHub
2. Connect repo to [Vercel](https://vercel.com)
3. Add all .env variables in the Vercel dashboard
4. Deploy — Vercel auto-detects the Node.js backend

---

## 🤝 Contributing

Pull requests are welcome. For major changes, open an issue first to discuss what you'd like to change.

---

## 👤 Author

**Madhu Ratnam**
- GitHub: [@madhu-ratnam](https://github.com/madhu-ratnam)
- Repo: [MarketNext](https://github.com/madhu-ratnam/Ecommerce-app)
