Bitbasket Backend
1. Overview
The backend is a RESTful API built with Node.js and Express. It uses MongoDB (via Mongoose) for data storage. Main responsibilities:

Serve JSON data for products, users, and orders.
Handle user authentication (registration/login with JWT).
Enforce access control for protected and admin routes.
Process orders and payments (if integrated later).
Provide data to the admin panel.
2. Setup
Create the backend folder and initialize npm:

mkdir backend && cd backend && npm init -y

Install dependencies: npm install express mongoose dotenv jsonwebtoken bcryptjs cors express-async-handler.

express: Web server framework.
mongoose: MongoDB ODM (defining schemas/models).
dotenv: Load environment variables from .env.
jsonwebtoken & bcryptjs: For auth (tokens and password hashing).
cors: Middleware to allow cross-origin requests (for production).
express-async-handler: To simplify async error handling in routes.
3. Folder Structure
Organize the backend code:

backend/
├── config/        # Configuration files (e.g., db.js for DB connection)
├── controllers/   # Route handler functions (for users, products, orders)
├── models/        # Mongoose schemas/models (User, Product, Order)
├── routes/        # Express routers for each resource (/api/users, /api/products, /api/orders)
├── middleware/    # Custom middleware (auth checks, error handlers)
├── server.js      # App entry point (sets up Express, middleware, routes)
Example: models/Product.js defines the Product schema, controllers/productController.js implements logic for fetching products, and routes/productRoutes.js maps HTTP verbs to controller functions.

4. Models
Define Mongoose schemas for main entities:

User: Fields might include userName, email, password (hashed), and role (Boolean). Add a method to the model to verify password (with bcrypt) and to generate JWT.
Product: Fields include name, price, brand, category, countInStock, image URL, and description. Store necessary details for listing and cart.
Order: Fields include a reference to the user (user: { type: mongoose.Schema.Types.ObjectId, ref: 'User' }), an array of items (with product, qty, price), shipping address, totalPrice, order status, and timestamps for created/paid/shipped dates.
Cart: Fields include a reference to the user (user: { type: mongoose.Schema.Types.ObjectId, ref: 'User' }), an array of items (with product, qty, price), and calculated totalPrice.
Use Mongoose schema options (like timestamps: true) to auto-track creation dates. Ensure to index or optimize queries as needed for performance (for example, indexing product names for search).

5. Authentication
Implement routes for Register and Login:

POST /api/users/register: Create a new user. Hash the password with bcrypt before saving. Respond with user info and a JWT token.
POST /api/users/login: Verify email and password. If valid, respond with user data and JWT token.
JWT is signed with a secret (e.g. from process.env.JWT_SECRET) and includes the user ID. Store the token on the client to authorize protected requests. Use Express middleware to protect routes: e.g. a function that extracts the token from the Authorization header, verifies it, and attaches req.user to the request object. A separate middleware can check req.user.isAdmin for admin routes.

6. API Routes
Implement RESTful endpoints (conventionally under /api) such as:

Users:

POST /api/users/login – Authenticate and get token.
POST /api/users/register – Create a new user.
GET /api/users/profile – Get logged-in user’s profile (protected).
PUT /api/users/profile – Update profile (protected).
Admin-only: GET /api/users – List all users; DELETE /api/users/:id – Delete user; GET /api/users/:id – Get user by ID for admin.
Products:

GET /api/products – Get all products (with optional filters or pagination).
GET /api/products/:id – Get single product by ID.
Admin-only: POST /api/products – Create new product; PUT /api/products/:id – Update product; DELETE /api/products/:id – Delete product.
Orders:

POST /api/orders – Create a new order (protected).
GET /api/orders/myorders – Get orders of logged-in user.
GET /api/orders/:id – Get order by ID (protected; allow only owner or admin).
PUT /api/orders/:id/pay – Mark order as paid (e.g. after payment).
Admin-only: GET /api/orders – Get all orders; PUT /api/orders/:id/deliver – Mark as delivered.
Controllers handle the logic (database queries and business rules) and return JSON. Use Express routers to connect these endpoints. Return proper HTTP status codes (200/201 for success, 400 for bad request, 401 for auth error, 404 for not found, etc.).

7. Error Handling
Set up middleware to catch errors:

404 Not Found: If a route is not matched, send a 404 response (e.g. a middleware at the end of routers).
General Error Handler: A middleware with four arguments (err, req, res, next) can format errors into JSON responses. You can check for Mongoose validation errors or JWT errors and set res.status accordingly.
Using a package like express-async-handler in routes lets you throw errors from async controllers, and the error-handling middleware will catch them. Always return meaningful error messages so the frontend can show them to the user if needed.

8. Env Variables (.env)
Create a .env file to store secrets (do NOT commit this). Typical variables:

PORT=5000
MONGO_URI=mongodb+srv://<user>:<pass>@cluster.mongodb.net/yourDB
JWT_SECRET=YourJWTSecret
Load these in server.js using require('dotenv').config(). Use process.env.PORT and process.env.MONGO_URI when connecting Express and Mongoose. This keeps sensitive info (like database credentials) out of code.
