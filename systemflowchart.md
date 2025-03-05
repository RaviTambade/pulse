### **System Flowchart for PlusCare Application**

A **flowchart** is a diagram that represents the workflow or process of the system. It is useful for understanding how the system functions and how different components interact with each other.

The **PlusCare Application** consists of several key modules, including **User Registration/Login**, **Appointment Booking**, **Prescription Scanning**, **Location Services**, **Recommendations**, and the **AI Chatbot**. Below is a flowchart that illustrates the flow of the system across these functionalities.

---

### **System Flowchart**

1. **Start**
    - The process begins when the user interacts with the application.

---

#### **User Registration/Login**:
- **User** opens the app:
    - **Is user already registered?**
        - **Yes**: Go to **Login** process.
        - **No**: Go to **Registration** process.
  
    **Registration Process**:
    - User enters **Name**, **Email**, **Password**, **Role** (Patient/Doctor/Admin), and other necessary details.
    - User clicks **Register**.
    - The system validates the details (e.g., Email format, password strength).
    - If valid, **Store user details** in the **Database** (MySQL).
    - Display **Success Message**: "Registration Successful".
    - Redirect to **Login**.

    **Login Process**:
    - User enters **Email** and **Password**.
    - The system validates credentials:
        - **Valid credentials**: Proceed to the **Dashboard**.
        - **Invalid credentials**: Show an error message: "Invalid credentials".

---

#### **Dashboard/Homepage** (After successful login):
- User is presented with multiple options:
  - **Book Appointment**
  - **View Prescriptions**
  - **Location-based Healthcare Providers**
  - **AI Chatbot**
  - **Profile Settings**

---

#### **Booking an Appointment**:
1. User selects **Book Appointment**.
2. Choose a **Doctor** from the list.
3. **Check Doctor’s Availability** (Time slots for the selected doctor):
   - Available Time Slot → Proceed to **Confirm Appointment**.
   - No available time → **Notify user**: "Doctor not available for selected time."
4. User selects the **Date** and **Time**.
5. Confirm appointment details.
6. **Store Appointment details** in the **Database** (MySQL).
7. **Appointment Confirmation** message is shown.
8. **End Appointment Booking** process.

---

#### **Prescription Scanning**:
1. User selects **Upload Prescription**.
2. User uploads **Scanned Prescription Image**.
3. **Run OCR (Optical Character Recognition)** to extract text from the image.
4. **Extracted Prescription Details** (Medicines, Dosage, etc.) are displayed.
5. User confirms and clicks **Save**.
6. Store **Prescription Details** in the **Database** (MySQL).
7. Display **Prescription Upload Success** message.
8. **End Prescription Upload** process.

---

#### **Location-Based Services**:
1. User selects **Find Nearby Healthcare Providers**.
2. **Get User Location** via **GPS**.
3. Search for healthcare providers in proximity (Doctors, Pharmacies, Clinics).
4. **Display Healthcare Providers** list on **Map**.
5. User can view **Providers on Map** or **Search Results**.
6. **End Location-Based Service** process.

---

#### **AI Chatbot**:
1. User selects **Chat with AI Chatbot**.
2. **Chatbot Interface** opens.
3. User types a **Query** (e.g., "What are my upcoming appointments?", "Find a doctor").
4. The system checks the **Database** for required information (appointments, doctor details, etc.).
5. **AI Chatbot** generates a response based on user query.
6. Display the response to the user.
7. User can continue the conversation or **End Chat**.

---

#### **Profile Settings**:
1. User selects **Profile Settings**.
2. User can **Update Personal Information** (Phone number, Address, etc.).
3. System validates and updates **Database** with new details.
4. Display **Profile Update Success** message.
5. **End Profile Settings** process.

---

### **System Flowchart Representation:**

```plaintext
+-----------------------+
|       Start           |
+-----------------------+
           |
           v
+-----------------------+     No    +---------------------------+
| Is User Registered?   +--------->| Registration Process      |
+-----------------------+           +---------------------------+
           |                         | Enter details (Name, Email, etc.)
           v                         | Validate and store data
+-----------------------+            | Success message
|      Login Process    +------------+ Redirect to Login
+-----------------------+            |
           |                         v
           v                 +---------------------------+
+---------------------------| Login Process            |
|      Dashboard/Homepage   |                        |
+---------------------------+------------------------>|
|  - Book Appointment       |                        |
|  - View Prescriptions     |                        |
|  - Location Services      |                        |
|  - AI Chatbot             |                        |
|  - Profile Settings       |                        |
+---------------------------+                        |
           |                                            |
           |                                            |
           |      +-----------------------------------+
           v      v                                   v
+---------------------+         +--------------------------+        +------------------+
| Book Appointment    |         |  Prescription Scanning   |        | Location-based   |
+---------------------+         +--------------------------+        | Services         |
| - Choose Doctor     |         | - Upload Prescription    |        +------------------+
| - Select Time Slot |         | - OCR Extraction         |        | - Get Location   |
| - Confirm Details  |         | - Save Prescription      |        | - Search Nearby  |
+---------------------+         +--------------------------+        | Providers        |
| - Appointment Saved |         | - Display Success        |        +------------------+
+---------------------+         +--------------------------+
           |                                            |
           |                                            |
           |                                            v
           v                           +-------------------------+
+-------------------------+            |  AI Chatbot             |
| Profile Settings        |            +-------------------------+
+-------------------------+            | - Start Chat            |
| - Update Profile Info   |            | - Send Query           |
| - Save Changes          |            | - Display Response     |
+-------------------------+            | - Continue or End Chat |
           |                           +-------------------------+
           v
+-----------------------+
|        End            |
+-----------------------+
```

---

### **Explanation of the Flowchart:**

1. **Start**:
   - The process begins when a user opens the application.

2. **User Registration/Login**:
   - The system checks if the user is already registered or not. If they are registered, they proceed to login. If not, they go through the registration process, where their information is validated and stored.

3. **Dashboard/Homepage**:
   - Once logged in, users are directed to the main dashboard, where they can select different actions like booking an appointment, uploading prescriptions, finding healthcare providers, chatting with AI, or editing their profile.

4. **Appointment Booking**:
   - Users can select a doctor, choose a time slot, and confirm the appointment. The system checks if the doctor is available and confirms the booking by saving the appointment data.

5. **Prescription Scanning**:
   - Users can upload scanned prescriptions. The system uses OCR to extract the text from the scanned image and saves the extracted data in the database.

6. **Location-Based Services**:
   - The system uses the user’s GPS location to find nearby healthcare providers such as doctors, pharmacies, and clinics, which are displayed on the map or in a list.

7. **AI Chatbot**:
   - The chatbot allows users to interact and ask questions. It responds to queries regarding appointments, doctors, and more by fetching data from the system.

8. **Profile Settings**:
   - Users can update their profile information, which is validated and saved in the database.

---

### **Conclusion:**
This **system flowchart** provides a high-level overview of how different features of the **PlusCare Application** interact with each other. It gives a visual representation of the user journey through the system, from registration to booking appointments, uploading prescriptions, chatting with the AI, and managing their profile settings. This helps developers and stakeholders understand the overall flow of the application.