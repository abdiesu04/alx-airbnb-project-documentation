
---

## **1. User Registration with Roles (Guest, Host, Admin)**

### **Feature Overview:**
The **User Registration** process allows users to create accounts with roles: **Guest**, **Host**, and **Admin**. Each role determines the access and actions that a user can perform. The default role is **Guest**, and only an **Admin** can assign the **Admin** role.

---

### **API Endpoints:**

#### **POST /auth/register**
**Description:** Registers a new user with a specific role (`Guest`, `Host`, or `Admin`).  
- **Input:** User credentials (name, email, password, and role).  
- **Output:** Success message or relevant error message with validation feedback.

---

### **Input Specification:**

```json
{
  "name": "John Doe",
  "email": "johndoe@example.com",
  "password": "Password@123",
  "role": "Host"
}
```

---

### **Input Validation Rules:**

1. **Name:**  
   - Required, must be a string.
   - Minimum length: 3 characters, maximum length: 50 characters.
   - Cannot contain special characters or numbers.

2. **Email:**  
   - Required, must be a valid email format (`example@domain.com`).
   - Must be unique in the database.
   - Use regex validation to check for valid format.
   
3. **Password:**  
   - Required.
   - Minimum length: 8 characters.
   - Must include at least:
     - One uppercase letter.
     - One lowercase letter.
     - One special character (e.g., `@, #, $, etc.`).
     - One digit.
   - Password should be securely hashed before storing.

4. **Role:**  
   - Must be one of the following: `Guest`, `Host`, or `Admin`.
   - Default is `Guest` if no role is provided.
   - **Important:** Only an **Admin** can assign the `Admin` role.

---

### **Output Specifications:**

#### **1. Success:**

**Condition:** User successfully registered.  
```json
{
  "message": "Registration successful. Please verify your email.",
  "userId": "12345",
  "role": "Host"
}
```

#### **2. Error Scenarios:**

- **Email already exists:**  
  **Condition:** Email already exists in the database.  
  ```json
  {
    "error": "Email already registered. Please login or reset your password."
  }
  ```

- **Invalid email format:**  
  **Condition:** The email format is invalid.  
  ```json
  {
    "error": "Invalid email format. Please use a valid email address."
  }
  ```

- **Weak password:**  
  **Condition:** Password doesn't meet the complexity requirements.  
  ```json
  {
    "error": "Password must include at least 8 characters, one uppercase letter, one special character, and one number."
  }
  ```

- **Invalid role:**  
  **Condition:** Role is not one of the valid roles (`Guest`, `Host`, `Admin`).  
  ```json
  {
    "error": "Invalid role. Allowed roles are 'Guest', 'Host', and 'Admin'."
  }
  ```

- **Unauthorized role assignment:**  
  **Condition:** A non-Admin user attempts to register as an `Admin`.  
  ```json
  {
    "error": "You do not have permission to assign the Admin role."
  }
  ```

- **Missing required fields:**  
  **Condition:** Missing required fields (e.g., `name`, `email`, `password`).  
  ```json
  {
    "error": "Name, email, and password are required."
  }
  ```

---

### **Database Schema for User Registration:**

```sql
CREATE TABLE Users (
    id UUID PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    role ENUM('Guest', 'Host', 'Admin') DEFAULT 'Guest',
    is_verified BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

### **Role-Based Access Control:**

- **Guest:**  
   - Default role for users who simply register to browse, book properties, and manage their profile.
   - Permissions: View properties, make bookings, update profile.

- **Host:**  
   - Can list properties, manage their own property bookings, and view income reports.
   - Permissions: List properties, manage bookings, view host-specific analytics.

- **Admin:**  
   - Can manage all users, properties, and bookings across the platform.
   - Permissions: Create/edit users, assign roles (including Admin), manage all properties and bookings.

---

### **API for Role Assignment by Admin (POST /auth/assign-role)**

**Description:** Allows an **Admin** to assign roles to other users (e.g., granting Host or Admin roles).  
- **Input:** User ID and the role to assign.  
- **Output:** Confirmation message or error based on the role assignment.

#### **Input Example:**

```json
{
  "userId": "12345",
  "role": "Host"
}
```

#### **Output Example (Success):**
```json
{
  "message": "Role successfully updated to Host for user 12345."
}
```

#### **Output Example (Error):**

- **Non-Admin attempting role assignment:**  
  ```json
  {
    "error": "Only an Admin can assign roles."
  }
  ```

- **Invalid role:**  
  ```json
  {
    "error": "Invalid role. Allowed roles are 'Guest', 'Host', and 'Admin'."
  }
  ```

---

### **Performance Criteria:**

- **Response time for registration:**  
   - Successful registration: Should complete within **300ms** for up to **1000 concurrent requests**.
   - Email verification: Must be triggered within **2 seconds** of successful registration.
   
- **Role assignment response time:**  
   - Assigning or updating a role for an existing user should be completed within **200ms** for up to **1000 concurrent requests**.

---

### **Edge Cases & Error Handling:**

- **Duplicate Email Registration:**  
   - The system should handle duplicate email attempts by providing a helpful error message (`"Email already registered. Please login or reset your password."`).
   
- **Password Complexity:**  
   - If the password doesn’t meet the specified complexity requirements, the system should reject the registration request and inform the user with the error message `"Password must include at least 8 characters, one uppercase, one special character, and one number."`

- **Role-based Restrictions:**  
   - **Unauthorized Role Assignment:** If a non-Admin user attempts to assign the `Admin` role, the system should respond with `"You do not have permission to assign the Admin role."`
   - **Admin Role:** The `Admin` role can only be assigned via the Admin interface. Only the existing **Admin** can assign this role.

- **Email Verification Failure:**  
   - If the email verification link expires or is invalid, return an error message:  
     ```json
     {
       "error": "Email verification failed or link expired. Please request a new verification email."
     }
     ```

---

---

## **2. Property Management**

### **Feature Overview:**
The **Property Management** feature allows **Hosts** to manage their listed properties, including creating, updating, deleting, and viewing property details. **Admin** users can view, edit, or delete any property, while **Hosts** can only manage their own properties. **Guests** can view properties but cannot modify them.

---

### **API Endpoints:**

#### **POST /properties**
**Description:** Allows a **Host** to add a new property.  
- **Input:** Property details (title, description, location, price, photos, etc.).  
- **Output:** Success message with property details or error message.

---

#### **GET /properties/{propertyId}**
**Description:** Allows any user (Guest, Host, Admin) to view property details.  
- **Input:** Property ID.  
- **Output:** Property details or error message if the property doesn't exist.

---

#### **PUT /properties/{propertyId}**
**Description:** Allows a **Host** to update an existing property they own.  
- **Input:** Updated property details.  
- **Output:** Success message or error message if unauthorized or invalid data.

---

#### **DELETE /properties/{propertyId}**
**Description:** Allows a **Host** to delete a property they own or an **Admin** to delete any property.  
- **Input:** Property ID.  
- **Output:** Success message or error message if unauthorized.

---

### **Input Specification for Property Creation:**

```json
{
  "title": "Beachfront Villa",
  "description": "A beautiful villa by the sea with an infinity pool.",
  "location": {
    "address": "123 Beach Road",
    "city": "Miami",
    "state": "FL",
    "zip": "33101",
    "country": "USA"
  },
  "price_per_night": 250,
  "available_dates": ["2024-12-01", "2024-12-02", "2024-12-03"],
  "photos": ["photo1.jpg", "photo2.jpg"]
}
```

---

### **Input Validation Rules:**

1. **Title:**  
   - Required.
   - Minimum length: 3 characters, maximum length: 100 characters.
   - Cannot contain special characters that may cause rendering issues.

2. **Description:**  
   - Required.
   - Minimum length: 10 characters, maximum length: 500 characters.
   - Should not contain inappropriate language.

3. **Location:**  
   - Required.
   - **Address:** A valid address string, cannot be empty.
   - **City, State, Country:** Required, should be valid string entries.
   - **Zip code:** Optional, if provided, must match the country format.

4. **Price per Night:**  
   - Required.
   - Must be a positive number greater than zero (e.g., `250`).
   - Should allow decimal values (e.g., `99.99`).

5. **Available Dates:**  
   - Optional, can be empty if property is available at all times.
   - Should be an array of valid date strings in `YYYY-MM-DD` format.

6. **Photos:**  
   - Optional.
   - Should be an array of valid photo URLs or file names.
   - At least one photo must be provided if the property is being listed.

---

### **Output Specifications:**

#### **1. Success (Property Creation):**

**Condition:** A new property is successfully created by a **Host**.

```json
{
  "message": "Property created successfully.",
  "propertyId": "abcd1234",
  "title": "Beachfront Villa",
  "price_per_night": 250,
  "location": {
    "address": "123 Beach Road",
    "city": "Miami",
    "state": "FL",
    "zip": "33101",
    "country": "USA"
  },
  "available_dates": ["2024-12-01", "2024-12-02", "2024-12-03"],
  "photos": ["photo1.jpg", "photo2.jpg"]
}
```

#### **2. Error Scenarios:**

- **Unauthorized Access (Not Host or Admin):**  
  **Condition:** A **Guest** attempts to create or manage a property.  
  ```json
  {
    "error": "Only Hosts can create and manage properties."
  }
  ```

- **Missing or Invalid Input Fields:**  
  **Condition:** Required fields are missing or have invalid values (e.g., price is not a positive number).  
  ```json
  {
    "error": "Title, description, price, and location are required fields. Ensure price is a positive number."
  }
  ```

- **Invalid Photo URL/Format:**  
  **Condition:** Photo URLs are invalid or incorrectly formatted.  
  ```json
  {
    "error": "Invalid photo URLs. Ensure the URLs are valid or the filenames are correct."
  }
  ```

---

### **Database Schema for Property Management:**

```sql
CREATE TABLE Properties (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL,  -- Foreign key referencing the Host's User ID
    title VARCHAR(100) NOT NULL,
    description TEXT NOT NULL,
    location JSONB NOT NULL,
    price_per_night DECIMAL(10, 2) NOT NULL,
    available_dates DATE[],
    photos TEXT[] NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES Users(id)
);
```

---

### **Role-Based Access Control for Property Management:**

- **Guest:**  
   - Can only view properties.  
   - Cannot create, update, or delete properties.

- **Host:**  
   - Can create, update, and delete their own properties.
   - Cannot edit or delete properties listed by other users.

- **Admin:**  
   - Can view, edit, and delete any property on the platform.

---

### **Performance Criteria:**

- **Response time for property creation, update, or deletion:**  
   - Should complete within **500ms** for up to **1000 concurrent requests**.
   
- **Property search and listing performance:**  
   - Searching or filtering properties by location or price should return results in **<500ms** for up to **10,000 properties**.

- **Image upload and handling:**  
   - Image uploads for properties should be completed in **<2 seconds** for images under 5MB.

---

### **Edge Cases & Error Handling:**

- **Duplicate Property Listing:**  
   - The system should ensure that there are no duplicate properties with the same title and location for the same **Host**. If detected, return the error:
     ```json
     {
       "error": "You already have a property listed with this title and location."
     }
     ```

- **Non-Existent Property Update or Delete Attempt:**  
   - If a **Host** attempts to update or delete a property they do not own, return the error:  
     ```json
     {
       "error": "You cannot modify a property that you do not own."
     }
     ```

- **Unauthorized Property Deletion:**  
   - If a **Guest** or non-Admin user attempts to delete a property, return the error:  
     ```json
     {
       "error": "Only the property owner or an Admin can delete the property."
     }
     ```

- **Invalid Property Update Data:**  
   - If the data provided for updating the property is invalid (e.g., missing required fields or incorrect formats), return an error with the specific issue:
     ```json
     {
       "error": "Missing or invalid data. Please provide valid title, description, price, and location."
     }
     ```

---


---

## **3. Booking System**

### **Feature Overview:**
The **Booking System** allows **Guests** to book available properties for a specified duration. A **Host** can manage the availability and price for their property. **Admin** users can view, approve, or cancel any booking. The system ensures that double bookings do not occur and that all guests are properly notified upon booking confirmation.

---

### **API Endpoints:**

#### **POST /bookings**
**Description:** Allows a **Guest** to make a booking for a property.  
- **Input:** Booking details (guest ID, property ID, check-in/check-out dates, payment details).  
- **Output:** Booking confirmation or error message.

---

#### **GET /bookings/{bookingId}**
**Description:** Allows a **Guest** or **Admin** to view booking details.  
- **Input:** Booking ID.  
- **Output:** Booking details (property, guest, dates, status, etc.) or error message if the booking doesn't exist.

---

#### **PUT /bookings/{bookingId}/status**
**Description:** Allows an **Admin** or **Host** to update the status of a booking (e.g., approve, cancel, complete).  
- **Input:** Booking ID and new status.  
- **Output:** Success message or error message if unauthorized or invalid status.

---

#### **DELETE /bookings/{bookingId}**
**Description:** Allows a **Guest** to cancel their own booking, or **Host**/**Admin** to cancel a booking for any guest.  
- **Input:** Booking ID.  
- **Output:** Success message or error message if unauthorized or invalid operation.

---

### **Input Specification for Booking Creation:**

```json
{
  "propertyId": "abcd1234",
  "guestId": "guest1234",
  "checkInDate": "2024-12-01",
  "checkOutDate": "2024-12-05",
  "paymentDetails": {
    "paymentMethod": "credit_card",
    "amount": 1000,
    "paymentStatus": "completed"
  }
}
```

---

### **Input Validation Rules:**

1. **Property ID:**  
   - Required.
   - Must exist in the database and be available for the selected dates.
   
2. **Guest ID:**  
   - Required.
   - Must be a valid **Guest** user ID.

3. **Check-In/Check-Out Dates:**  
   - Required.
   - Check-In date must be in the future and before the Check-Out date.
   - Dates must not overlap with existing bookings for the selected property.

4. **Payment Details:**  
   - Required.
   - **Payment Method:** Should be one of "credit_card", "paypal", or "bank_transfer."
   - **Amount:** Should match the property's booking cost for the specified dates.
   - **Payment Status:** Should be "completed" for successful transactions.

---

### **Output Specifications:**

#### **1. Success (Booking Creation):**

**Condition:** A new booking is successfully created.

```json
{
  "message": "Booking created successfully.",
  "bookingId": "booking1234",
  "propertyId": "abcd1234",
  "guestId": "guest1234",
  "checkInDate": "2024-12-01",
  "checkOutDate": "2024-12-05",
  "paymentStatus": "completed",
  "totalAmount": 1000
}
```

#### **2. Error Scenarios:**

- **Property Already Booked:**  
  **Condition:** The property is already booked for the selected dates.  
  ```json
  {
    "error": "The selected property is already booked for the given dates. Please choose different dates."
  }
  ```

- **Invalid Dates (Past Date or Overlapping):**  
  **Condition:** The check-in or check-out date is in the past or the booking overlaps with another reservation.  
  ```json
  {
    "error": "The check-in date must be in the future and should not overlap with existing bookings."
  }
  ```

- **Invalid Payment Details:**  
  **Condition:** The payment details are incorrect or the payment status is not marked as completed.  
  ```json
  {
    "error": "Invalid payment details. Please ensure payment is completed before booking."
  }
  ```

- **Unauthorized Booking Attempt (Guest Trying to Book Without Payment):**  
  **Condition:** A **Guest** attempts to make a booking without providing valid payment details.  
  ```json
  {
    "error": "Booking cannot be created without valid payment details."
  }
  ```

- **Unauthorized Booking Cancellation (Non-Guest Trying to Cancel):**  
  **Condition:** A user who is not the guest cannot cancel the booking.  
  ```json
  {
    "error": "Only the guest who made the booking can cancel it."
  }
  ```

---

### **Database Schema for Booking Management:**

```sql
CREATE TABLE Bookings (
    id UUID PRIMARY KEY,
    property_id UUID NOT NULL,  -- Foreign key referencing the Property ID
    guest_id UUID NOT NULL,  -- Foreign key referencing the Guest's User ID
    check_in_date DATE NOT NULL,
    check_out_date DATE NOT NULL,
    payment_status VARCHAR(50) NOT NULL,  -- 'completed', 'pending', 'failed'
    amount DECIMAL(10, 2) NOT NULL,  -- Total booking amount
    status VARCHAR(50) DEFAULT 'pending',  -- 'pending', 'approved', 'completed', 'cancelled'
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (property_id) REFERENCES Properties(id),
    FOREIGN KEY (guest_id) REFERENCES Users(id)
);
```

---

### **Role-Based Access Control for Booking Management:**

- **Guest:**  
   - Can create a booking for a property they are interested in.
   - Can view and cancel their own bookings.
   - Cannot cancel bookings made by others.

- **Host:**  
   - Can view all bookings made for their own properties.
   - Can cancel bookings for their properties.
   - Cannot create or cancel bookings for other Hosts' properties.

- **Admin:**  
   - Can view, approve, cancel, or complete bookings for any property.
   - Can update booking statuses (approve, cancel, complete).

---

### **Performance Criteria:**

- **Response time for booking creation:**  
   - Should complete within **500ms** for up to **1000 concurrent booking requests**.

- **Search and booking performance:**  
   - Searching and booking for properties should be handled in **<500ms** for up to **10,000 properties**.

- **Payment processing time:**  
   - Payment completion (via integrated payment service) should not exceed **5 seconds**.

---

### **Edge Cases & Error Handling:**

- **Booking Overlap:**  
   - If a guest tries to book a property on dates that have already been booked by another guest, return the error:  
   ```json
   {
     "error": "The selected dates are already taken. Please choose different dates."
   }
   ```

- **Unauthorized Attempt to Modify Booking:**  
   - If a non-Admin user or non-owner user attempts to modify or cancel a booking, return an error:  
   ```json
   {
     "error": "You do not have permission to modify or cancel this booking."
   }
   ```

- **Booking Confirmation Failure:**  
   - If an error occurs during booking confirmation (e.g., payment failure or system error), return an appropriate error message.  
   ```json
   {
     "error": "Booking confirmation failed. Please try again later."
   }
   ```

---
