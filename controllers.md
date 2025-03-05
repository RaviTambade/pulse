Below are some example **controller** files for common functionalities in the PlusCare Application, including **authentication**, **appointments**, and **prescriptions**. These controllers will handle the business logic for each route, interact with the MySQL database, and send appropriate responses to the client.

---

### **1. `authController.js`** - Handles user authentication (register and login)

```javascript
// controllers/authController.js
const bcrypt = require('bcrypt');
const jwt = require('jsonwebtoken');

// Register user
exports.register = (req, res) => {
  const { username, email, password } = req.body;

  // Check if the email already exists
  const checkEmailQuery = 'SELECT * FROM users WHERE email = ?';
  req.db.query(checkEmailQuery, [email], (err, results) => {
    if (err) {
      return res.status(500).json({ message: 'Database error' });
    }

    if (results.length > 0) {
      return res.status(400).json({ message: 'Email already exists' });
    }

    // Hash the password before saving to the database
    bcrypt.hash(password, 10, (err, hashedPassword) => {
      if (err) {
        return res.status(500).json({ message: 'Error hashing password' });
      }

      // Insert the new user into the database
      const insertQuery = 'INSERT INTO users (username, email, password) VALUES (?, ?, ?)';
      req.db.query(insertQuery, [username, email, hashedPassword], (err, results) => {
        if (err) {
          return res.status(500).json({ message: 'Database error' });
        }
        res.status(201).json({ message: 'User registered successfully' });
      });
    });
  });
};

// Login user
exports.login = (req, res) => {
  const { email, password } = req.body;

  // Check if the email exists
  const query = 'SELECT * FROM users WHERE email = ?';
  req.db.query(query, [email], (err, results) => {
    if (err) {
      return res.status(500).json({ message: 'Database error' });
    }

    if (results.length === 0) {
      return res.status(400).json({ message: 'Invalid credentials' });
    }

    const user = results[0];

    // Compare the provided password with the stored hashed password
    bcrypt.compare(password, user.password, (err, isMatch) => {
      if (err) {
        return res.status(500).json({ message: 'Error comparing passwords' });
      }

      if (!isMatch) {
        return res.status(400).json({ message: 'Invalid credentials' });
      }

      // Generate a JWT token
      const token = jwt.sign(
        { userId: user.id, username: user.username },
        process.env.JWT_SECRET,
        { expiresIn: '1h' }
      );

      res.status(200).json({ token });
    });
  });
};
```

---

### **2. `appointmentController.js`** - Handles appointment-related functionality

```javascript
// controllers/appointmentController.js

// Book an appointment
exports.bookAppointment = (req, res) => {
  const { userId, doctorId, appointmentDate, appointmentTime } = req.body;

  // Check if the appointment slot is available
  const checkSlotQuery = 'SELECT * FROM appointments WHERE doctor_id = ? AND appointment_date = ? AND appointment_time = ?';
  req.db.query(checkSlotQuery, [doctorId, appointmentDate, appointmentTime], (err, results) => {
    if (err) {
      return res.status(500).json({ message: 'Database error' });
    }

    if (results.length > 0) {
      return res.status(400).json({ message: 'The selected time slot is already booked' });
    }

    // Insert the appointment into the database
    const insertQuery = 'INSERT INTO appointments (user_id, doctor_id, appointment_date, appointment_time) VALUES (?, ?, ?, ?)';
    req.db.query(insertQuery, [userId, doctorId, appointmentDate, appointmentTime], (err, results) => {
      if (err) {
        return res.status(500).json({ message: 'Database error' });
      }
      res.status(201).json({ message: 'Appointment booked successfully' });
    });
  });
};

// Get appointments for a user
exports.getAppointments = (req, res) => {
  const userId = req.params.userId;

  // Fetch all appointments for a specific user
  const query = 'SELECT * FROM appointments WHERE user_id = ?';
  req.db.query(query, [userId], (err, results) => {
    if (err) {
      return res.status(500).json({ message: 'Database error' });
    }
    res.status(200).json({ appointments: results });
  });
};
```

---

### **3. `prescriptionController.js`** - Handles prescription-related functionality

```javascript
// controllers/prescriptionController.js

// Add a prescription for a patient
exports.addPrescription = (req, res) => {
  const { userId, doctorId, medication, dosage, prescriptionDate } = req.body;

  // Insert the prescription into the database
  const insertQuery = 'INSERT INTO prescriptions (user_id, doctor_id, medication, dosage, prescription_date) VALUES (?, ?, ?, ?, ?)';
  req.db.query(insertQuery, [userId, doctorId, medication, dosage, prescriptionDate], (err, results) => {
    if (err) {
      return res.status(500).json({ message: 'Database error' });
    }
    res.status(201).json({ message: 'Prescription added successfully' });
  });
};

// Get prescriptions for a user
exports.getPrescriptions = (req, res) => {
  const userId = req.params.userId;

  // Fetch all prescriptions for a specific user
  const query = 'SELECT * FROM prescriptions WHERE user_id = ?';
  req.db.query(query, [userId], (err, results) => {
    if (err) {
      return res.status(500).json({ message: 'Database error' });
    }
    res.status(200).json({ prescriptions: results });
  });
};
```

---

### **4. `userController.js`** - Handles user profile functionality

```javascript
// controllers/userController.js

// Get user profile details
exports.getUserProfile = (req, res) => {
  const userId = req.params.userId;

  // Fetch the user's profile information
  const query = 'SELECT id, username, email FROM users WHERE id = ?';
  req.db.query(query, [userId], (err, results) => {
    if (err) {
      return res.status(500).json({ message: 'Database error' });
    }

    if (results.length === 0) {
      return res.status(404).json({ message: 'User not found' });
    }

    res.status(200).json({ user: results[0] });
  });
};

// Update user profile details
exports.updateUserProfile = (req, res) => {
  const { userId, username, email } = req.body;

  // Update user profile information
  const updateQuery = 'UPDATE users SET username = ?, email = ? WHERE id = ?';
  req.db.query(updateQuery, [username, email, userId], (err, results) => {
    if (err) {
      return res.status(500).json({ message: 'Database error' });
    }

    if (results.affectedRows === 0) {
      return res.status(404).json({ message: 'User not found' });
    }

    res.status(200).json({ message: 'Profile updated successfully' });
  });
};
```

---

### **5. `feedbackController.js`** - Handles feedback submission functionality

```javascript
// controllers/feedbackController.js

// Submit user feedback
exports.submitFeedback = (req, res) => {
  const { userId, feedbackText, rating } = req.body;

  // Insert feedback into the database
  const insertQuery = 'INSERT INTO feedback (user_id, feedback_text, rating) VALUES (?, ?, ?)';
  req.db.query(insertQuery, [userId, feedbackText, rating], (err, results) => {
    if (err) {
      return res.status(500).json({ message: 'Database error' });
    }
    res.status(201).json({ message: 'Feedback submitted successfully' });
  });
};

// Get feedback for a doctor or service
exports.getFeedback = (req, res) => {
  const doctorId = req.params.doctorId;

  // Fetch all feedback for a specific doctor
  const query = 'SELECT * FROM feedback WHERE doctor_id = ?';
  req.db.query(query, [doctorId], (err, results) => {
    if (err) {
      return res.status(500).json({ message: 'Database error' });
    }
    res.status(200).json({ feedback: results });
  });
};
```

---

### **Explanation of Controller Code:**

1. **Database Interaction**:
   - Each controller uses `req.db.query()` to interact with the MySQL database. This method performs SQL queries like `SELECT`, `INSERT`, `UPDATE`, etc.
   - Error handling is performed inside each query callback function. If a query fails, the controller responds with an appropriate error message.
   
2. **Business Logic**:
   - Each controller implements the necessary business logic for the respective feature (e.g., user authentication, appointment booking, prescription management, etc.).
   - For example, before booking an appointment, the system checks if the time slot is available (`SELECT` query), and if not, it responds with a conflict message.

3. **Authentication**:
   - The `authController.js` handles user registration and login, including password hashing (`bcrypt`) and JWT generation for secure authentication.

4. **Controller Structure**:
   - Each controller corresponds to a specific resource (e.g., `appointments`, `prescriptions`, `feedback`), and it contains methods (functions) for interacting with that resource.
   - These controller functions are then linked to corresponding routes in your `routes` folder.

---

### **Conclusion:**

With these controller files, you can manage the core functionalities of the **PlusCare Application** efficiently using **MySQL**. The controllers interact with the database to retrieve, insert, update, and delete records, while ensuring business logic and error handling are properly managed.