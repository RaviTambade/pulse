The **Data Access Logic (DAL)** layer is responsible for abstracting the database operations from the business logic in the controller. It encapsulates all interactions with the database, allowing the rest of the application to interact with it in a simplified way.

In a **MERN stack** with **MySQL** as the backend database, your DAL layer would contain functions to interact with the database (e.g., `getUser`, `createAppointment`, `updatePrescription`, etc.). The controllers will then call these DAL functions to perform the database operations.

### **Data Access Layer (DAL) Code**

We'll create a set of DAL functions to handle common operations for the **authentication**, **appointments**, and **prescriptions**.

---

### **1. DAL for `auth.js` (Authentication)**

```javascript
// dal/authDAL.js
const bcrypt = require('bcrypt');

// Register user
exports.registerUser = (db, username, email, password) => {
  return new Promise((resolve, reject) => {
    // Check if the email already exists
    const checkEmailQuery = 'SELECT * FROM users WHERE email = ?';
    db.query(checkEmailQuery, [email], (err, results) => {
      if (err) {
        reject(new Error('Database error'));
      }
      if (results.length > 0) {
        reject(new Error('Email already exists'));
      }

      // Hash the password
      bcrypt.hash(password, 10, (err, hashedPassword) => {
        if (err) {
          reject(new Error('Error hashing password'));
        }

        // Insert the new user into the database
        const insertQuery = 'INSERT INTO users (username, email, password) VALUES (?, ?, ?)';
        db.query(insertQuery, [username, email, hashedPassword], (err, results) => {
          if (err) {
            reject(new Error('Error inserting user'));
          }
          resolve('User registered successfully');
        });
      });
    });
  });
};

// Find user by email
exports.findUserByEmail = (db, email) => {
  return new Promise((resolve, reject) => {
    const query = 'SELECT * FROM users WHERE email = ?';
    db.query(query, [email], (err, results) => {
      if (err) {
        reject(new Error('Database error'));
      }
      resolve(results);
    });
  });
};
```

---

### **2. DAL for `appointment.js` (Appointments)**

```javascript
// dal/appointmentDAL.js

// Book an appointment
exports.bookAppointment = (db, userId, doctorId, appointmentDate, appointmentTime) => {
  return new Promise((resolve, reject) => {
    // Check if the time slot is available
    const checkSlotQuery = 'SELECT * FROM appointments WHERE doctor_id = ? AND appointment_date = ? AND appointment_time = ?';
    db.query(checkSlotQuery, [doctorId, appointmentDate, appointmentTime], (err, results) => {
      if (err) {
        reject(new Error('Database error'));
      }
      if (results.length > 0) {
        reject(new Error('The selected time slot is already booked'));
      }

      // Insert the appointment into the database
      const insertQuery = 'INSERT INTO appointments (user_id, doctor_id, appointment_date, appointment_time) VALUES (?, ?, ?, ?)';
      db.query(insertQuery, [userId, doctorId, appointmentDate, appointmentTime], (err, results) => {
        if (err) {
          reject(new Error('Error inserting appointment'));
        }
        resolve('Appointment booked successfully');
      });
    });
  });
};

// Get appointments for a user
exports.getAppointmentsByUserId = (db, userId) => {
  return new Promise((resolve, reject) => {
    const query = 'SELECT * FROM appointments WHERE user_id = ?';
    db.query(query, [userId], (err, results) => {
      if (err) {
        reject(new Error('Database error'));
      }
      resolve(results);
    });
  });
};
```

---

### **3. DAL for `prescription.js` (Prescriptions)**

```javascript
// dal/prescriptionDAL.js

// Add a prescription
exports.addPrescription = (db, userId, doctorId, medication, dosage, prescriptionDate) => {
  return new Promise((resolve, reject) => {
    // Insert the prescription into the database
    const insertQuery = 'INSERT INTO prescriptions (user_id, doctor_id, medication, dosage, prescription_date) VALUES (?, ?, ?, ?, ?)';
    db.query(insertQuery, [userId, doctorId, medication, dosage, prescriptionDate], (err, results) => {
      if (err) {
        reject(new Error('Database error'));
      }
      resolve('Prescription added successfully');
    });
  });
};

// Get prescriptions for a user
exports.getPrescriptionsByUserId = (db, userId) => {
  return new Promise((resolve, reject) => {
    const query = 'SELECT * FROM prescriptions WHERE user_id = ?';
    db.query(query, [userId], (err, results) => {
      if (err) {
        reject(new Error('Database error'));
      }
      resolve(results);
    });
  });
};
```

---

### **4. DAL for `feedback.js` (Feedback)**

```javascript
// dal/feedbackDAL.js

// Submit user feedback
exports.submitFeedback = (db, userId, doctorId, feedbackText, rating) => {
  return new Promise((resolve, reject) => {
    // Insert feedback into the database
    const insertQuery = 'INSERT INTO feedback (user_id, doctor_id, feedback_text, rating) VALUES (?, ?, ?, ?)';
    db.query(insertQuery, [userId, doctorId, feedbackText, rating], (err, results) => {
      if (err) {
        reject(new Error('Database error'));
      }
      resolve('Feedback submitted successfully');
    });
  });

// Get feedback for a doctor
exports.getFeedbackByDoctorId = (db, doctorId) => {
  return new Promise((resolve, reject) => {
    const query = 'SELECT * FROM feedback WHERE doctor_id = ?';
    db.query(query, [doctorId], (err, results) => {
      if (err) {
        reject(new Error('Database error'));
      }
      resolve(results);
    });
  });
};
```

---

### **Explanation of DAL Code:**

1. **Abstracted Database Logic**:
   - The DAL functions handle all direct interactions with the database.
   - Each function returns a `Promise`, which either resolves when the query is successful or rejects when there’s an error. This allows you to handle the results or errors in the calling controller in a consistent manner.

2. **Error Handling**:
   - Every database query is wrapped with proper error handling. If a query fails (e.g., due to a duplicate entry or invalid syntax), an error message is thrown.

3. **Promises**:
   - Since database operations are asynchronous, we use **Promises** to manage the flow of data. This allows the caller (i.e., the controller) to handle the result of the query in a clean and synchronous way using `.then()` or `async/await`.

4. **Data Validation and Queries**:
   - For example, in `registerUser`, we first check if the email already exists in the database before proceeding with the insertion. Similarly, in `bookAppointment`, we check if the appointment slot is available before confirming the booking.

5. **Encapsulation**:
   - The DAL functions only focus on database operations (CRUD). The controllers should handle the logic related to business rules, validation, and response formatting.

---

### **Controller Integration Example:**

In the controller file, you can now use these DAL functions:

```javascript
// controllers/authController.js
const authDAL = require('../dal/authDAL');
const bcrypt = require('bcrypt');
const jwt = require('jsonwebtoken');

exports.register = async (req, res) => {
  const { username, email, password } = req.body;
  
  try {
    await authDAL.registerUser(req.db, username, email, password);
    res.status(201).json({ message: 'User registered successfully' });
  } catch (error) {
    res.status(400).json({ message: error.message });
  }
};

exports.login = async (req, res) => {
  const { email, password } = req.body;
  
  try {
    const users = await authDAL.findUserByEmail(req.db, email);
    
    if (users.length === 0) {
      return res.status(400).json({ message: 'Invalid credentials' });
    }
    
    const user = users[0];
    
    const isMatch = await bcrypt.compare(password, user.password);
    
    if (!isMatch) {
      return res.status(400).json({ message: 'Invalid credentials' });
    }
    
    const token = jwt.sign(
      { userId: user.id, username: user.username },
      process.env.JWT_SECRET,
      { expiresIn: '1h' }
    );
    
    res.status(200).json({ token });
    
  } catch (error) {
    res.status(500).json({ message: 'Database error' });
  }
};
```

---

### **Conclusion:**

The **Data Access Layer (DAL)** abstracts the raw database queries and encapsulates them into reusable functions. These functions are designed to handle all interactions with the MySQL database, returning promises that resolve when the operation is successful and reject when there is an error. The controller then invokes these functions to perform the required operations, which improves maintainability and separation of concerns in your application.