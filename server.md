If you're using **MySQL** as the backend database instead of MongoDB, you'll need to adjust your `server.js` file accordingly. Specifically, you will use the **`mysql2`** or **`mysql`** package (a MySQL client for Node.js) to interact with the MySQL database, and you won't use `mongoose` for database operations since `mongoose` is MongoDB-specific.

Here is an updated version of `server.js` for using **MySQL** as the database:

---

### **`server.js` Using MySQL:**

```javascript
// Import required packages
const express = require('express');
const cors = require('cors');
const bodyParser = require('body-parser');
const mysql = require('mysql2');
const dotenv = require('dotenv');
const path = require('path');

// Import routes (assuming you have separate route files for modular structure)
const authRoutes = require('./routes/auth');
const appointmentRoutes = require('./routes/appointments');
const prescriptionRoutes = require('./routes/prescriptions');
const userRoutes = require('./routes/users');
const doctorRoutes = require('./routes/doctors');
const paymentRoutes = require('./routes/payments');
const feedbackRoutes = require('./routes/feedback');
const adminRoutes = require('./routes/admin');
const chatbotRoutes = require('./routes/chatbot');

// Initialize express app
const app = express();

// Load environment variables
dotenv.config();

// Use middlewares
app.use(cors());  // Allow cross-origin requests
app.use(bodyParser.json());  // Parse incoming JSON requests
app.use(bodyParser.urlencoded({ extended: true }));  // For form data

// Create a MySQL connection pool
const pool = mysql.createPool({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  waitForConnections: true,
  connectionLimit: 10,
  queueLimit: 0
});

// Test database connection
pool.getConnection((err, connection) => {
  if (err) {
    console.error('Error connecting to MySQL:', err.stack);
    process.exit(1); // Exit if unable to connect to the DB
  } else {
    console.log('Connected to MySQL as id ' + connection.threadId);
    connection.release(); // Release the connection back to the pool
  }
});

// Middleware to handle database connection in requests
app.use((req, res, next) => {
  req.db = pool;  // Make the database pool available for all routes
  next();
});

// Use routes
app.use('/api/auth', authRoutes);
app.use('/api/appointments', appointmentRoutes);
app.use('/api/prescriptions', prescriptionRoutes);
app.use('/api/users', userRoutes);
app.use('/api/doctors', doctorRoutes);
app.use('/api/payments', paymentRoutes);
app.use('/api/feedback', feedbackRoutes);
app.use('/api/admin', adminRoutes);
app.use('/api/chatbot', chatbotRoutes);

// Serve static assets in production (for React front-end)
if (process.env.NODE_ENV === 'production') {
  app.use(express.static(path.join(__dirname, 'client/build')));

  // Serve index.html for any route not recognized by the API
  app.get('*', (req, res) => {
    res.sendFile(path.resolve(__dirname, 'client', 'build', 'index.html'));
  });
}

// Catch-all route for undefined API endpoints
app.use((req, res, next) => {
  res.status(404).json({ message: 'API endpoint not found' });
});

// Global error handler
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ message: 'Something went wrong!' });
});

// Start the server
const PORT = process.env.PORT || 5000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

---

### **Explanation of Changes:**

1. **MySQL Connection Setup**:
   - Instead of using `mongoose`, we use the **`mysql2`** package (or **`mysql`**) to connect to a **MySQL** database.
   - The **`mysql.createPool()`** function is used to create a **connection pool**. A connection pool is more efficient than repeatedly creating new connections because it keeps a pool of reusable connections to the database.
   - The connection pool configuration includes:
     - `host`: The host where the MySQL database is running (e.g., `localhost` or a cloud provider).
     - `user`: The MySQL username.
     - `password`: The MySQL password.
     - `database`: The name of the database to connect to.
     - `waitForConnections`, `connectionLimit`, and `queueLimit` are optional but help with managing the connection pool.

2. **Database Connection Test**:
   - Once the connection pool is created, we use **`pool.getConnection()`** to verify that we can connect to the MySQL database.
   - If the connection is successful, the connection is released back to the pool.

3. **Make Database Available in Routes**:
   - We make the `db` (connection pool) object available to all routes via middleware. This allows each route to easily access the database by referencing `req.db` (e.g., `req.db.query()`).
   - This approach eliminates the need to create separate database connections in each route, ensuring efficient reuse of connections.

4. **Handling MySQL Queries in Routes**:
   - In your route files (e.g., `auth.js`, `appointments.js`), you will query the MySQL database using `req.db.query()`.
   - Example of a query in a route handler:
     ```javascript
     // In routes/auth.js
     const express = require('express');
     const router = express.Router();

     // Login route
     router.post('/login', (req, res) => {
       const { email, password } = req.body;

       const query = 'SELECT * FROM users WHERE email = ?';
       req.db.query(query, [email], (err, results) => {
         if (err) {
           return res.status(500).json({ message: 'Database error' });
         }
         if (results.length === 0) {
           return res.status(400).json({ message: 'Invalid credentials' });
         }
         // Process user authentication (e.g., bcrypt for password hashing)
         // Send a JWT token or session here
         res.status(200).json({ message: 'Logged in successfully' });
       });
     });

     module.exports = router;
     ```
   - In this example, the `req.db.query()` method is used to execute a MySQL query to retrieve the user by email.

---

### **Directory Structure for MySQL Integration**:

```plaintext
/pluscare-backend
  ├── /controllers        # Business logic for each route
  ├── /models            # MySQL models (e.g., User, Appointment, Prescription) (Optional - for abstraction)
  ├── /routes            # Express route files (auth.js, appointments.js, etc.)
  ├── /middleware        # Middlewares (e.g., authentication, validation)
  ├── /utils             # Utility functions (e.g., validation, formatting)
  ├── /client            # React front-end (if it's in the same repo)
  ├── .env               # Environment variables (DB credentials, JWT secret, etc.)
  ├── server.js          # Main entry point for the server
  └── package.json       # Node.js dependencies and scripts
```

---

### **.env File Example**:

Your **`.env`** file should contain the following environment variables (and others as needed for your application):

```env
DB_HOST=localhost
DB_USER=root
DB_PASSWORD=yourpassword
DB_NAME=pluscare_db

JWT_SECRET=your_jwt_secret
PORT=5000
NODE_ENV=development
```

---

### **Conclusion:**

This updated `server.js` file is now configured to use **MySQL** with a **connection pool** for efficient handling of database operations. Routes can easily query the database using `req.db.query()`. By following this pattern, you can implement various business logic for authentication, appointment management, prescription handling, etc., while utilizing MySQL as the backend database.

