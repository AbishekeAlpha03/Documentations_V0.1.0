
# Database Schema Documentation

### 1. **Users Table**
| Column Name      | Data Type        | Constraints                                       | Description                                    |
|------------------|------------------|--------------------------------------------------|------------------------------------------------|
| user_id          | INT              | PRIMARY KEY, AUTO_INCREMENT                      | Unique identifier for each user.               |
| name             | VARCHAR(100)     | NOT NULL                                         | Full name of the user.                         |
| email            | VARCHAR(255)     | NOT NULL, UNIQUE, INDEX, encrypted               | User's email, stored securely.                 |
| password_hash    | VARCHAR(255)     | NOT NULL                                         | Securely hashed password.                      |
| phone_number     | VARCHAR(20)      | NOT NULL, UNIQUE                                 | User's phone number for contact.               |
| created_at       | TIMESTAMP        | DEFAULT CURRENT_TIMESTAMP                        | Record of when the user was created.           |
| last_login       | TIMESTAMP        | NULL                                             | Last login timestamp for tracking.             |
| user_type        | ENUM('rider', 'driver') | NOT NULL                                    | To differentiate between rider and driver.     |
| status           | ENUM('active', 'inactive', 'banned') | DEFAULT 'active'               | Account status control.                        |

### 2. **Drivers Table**
| Column Name      | Data Type        | Constraints                                       | Description                                    |
|------------------|------------------|--------------------------------------------------|------------------------------------------------|
| driver_id        | INT              | PRIMARY KEY, AUTO_INCREMENT, FOREIGN KEY(user_id)| Unique identifier for each driver, linked to users table. |
| license_number   | VARCHAR(100)     | NOT NULL, UNIQUE                                 | Driver's license number for verification.      |
| vehicle_id       | INT              | FOREIGN KEY(vehicle_id)                          | Reference to the driver's vehicle.             |
| availability     | BOOLEAN          | NOT NULL, DEFAULT 1                              | Driver availability status (online/offline).   |
| rating           | DECIMAL(2,1)     | NULL                                             | Driver's average rating.                       |
| created_at       | TIMESTAMP        | DEFAULT CURRENT_TIMESTAMP                        | When the driver registered.                    |

### 3. **Vehicles Table**
| Column Name      | Data Type        | Constraints                                       | Description                                    |
|------------------|------------------|--------------------------------------------------|------------------------------------------------|
| vehicle_id       | INT              | PRIMARY KEY, AUTO_INCREMENT                      | Unique identifier for each vehicle.            |
| driver_id        | INT              | FOREIGN KEY(driver_id)                           | Reference to the driver owning the vehicle.    |
| make             | VARCHAR(50)      | NOT NULL                                         | Vehicle make (e.g., Toyota, Ford).             |
| model            | VARCHAR(50)      | NOT NULL                                         | Vehicle model (e.g., Camry, Civic).            |
| license_plate    | VARCHAR(20)      | NOT NULL, UNIQUE                                 | Vehicle's license plate for identification.    |
| year             | YEAR             | NOT NULL                                         | Year of manufacture.                           |
| color            | VARCHAR(30)      | NOT NULL                                         | Vehicle color for identification.              |
| status           | ENUM('active', 'maintenance', 'inactive') | NOT NULL          | Vehicle operational status.                    |

### 4. **Trips Table:**

| Column Name           | Data Type                 | Constraints                                          | Description                                    |
|-----------------------|---------------------------|-----------------------------------------------------|------------------------------------------------|
| trip_id               | INT                       | PRIMARY KEY, AUTO_INCREMENT                         | Unique identifier for each trip.               |
| rider_id              | INT                       | FOREIGN KEY (users.user_id)                         | Reference to the rider from the users table.   |
| driver_id             | INT                       | FOREIGN KEY (drivers.driver_id)                     | Reference to the driver from the drivers table.|
| start_location        | POINT                     | NOT NULL                                            | Start location (latitude, longitude).          |
| end_location          | POINT                     | NOT NULL                                            | End location (latitude, longitude).            |
| distance              | DECIMAL(6,2)              | NOT NULL                                            | Distance traveled in kilometers.               |
| fare                  | DECIMAL(10,2)             | NOT NULL                                            | Fare for the trip.                             |
| status                | ENUM('requested', 'ongoing', 'completed', 'canceled') | NOT NULL        | Current trip status.                           |
| cancellation_reason   | VARCHAR(255)              | NULL                                                | Reason for cancellation, if canceled.          |
| canceled_by           | ENUM('driver', 'rider', 'system') | NULL                                        | Who canceled the trip.                         |
| start_time            | TIMESTAMP                 | NULL                                                | When the trip started.                         |
| end_time              | TIMESTAMP                 | NULL                                                | When the trip ended.                           |
| created_at            | TIMESTAMP                 | DEFAULT CURRENT_TIMESTAMP                           | When the trip was created.                     |

### 5. **Payments Table**
| Column Name      | Data Type        | Constraints                                       | Description                                    |
|------------------|------------------|--------------------------------------------------|------------------------------------------------|
| payment_id       | INT              | PRIMARY KEY, AUTO_INCREMENT                      | Unique identifier for each payment.            |
| trip_id          | INT              | FOREIGN KEY(trip_id)                             | Reference to the trip (from trips table).      |
| rider_id         | INT              | FOREIGN KEY(rider_id)                            | Reference to the rider making the payment.     |
| amount           | DECIMAL(10,2)    | NOT NULL                                         | Total amount paid for the trip.                |
| payment_method   | ENUM('credit_card', 'paypal', 'cash') | NOT NULL                    | Payment method used.                           |
| status           | ENUM('pending', 'completed', 'failed') | NOT NULL               | Status of the payment.                         |
| payment_date     | TIMESTAMP        | DEFAULT CURRENT_TIMESTAMP                        | Date when payment was made.                    |

### 6. **Ratings Table**
| Column Name      | Data Type        | Constraints                                       | Description                                    |
|------------------|------------------|--------------------------------------------------|------------------------------------------------|
| rating_id        | INT              | PRIMARY KEY, AUTO_INCREMENT                      | Unique identifier for each rating.             |
| trip_id          | INT              | FOREIGN KEY(trip_id)                             | Reference to the trip being rated.             |
| user_id          | INT              | FOREIGN KEY(user_id)                             | Reference to the user giving the rating.       |
| rating           | INT              | NOT NULL, CHECK (rating >= 1 AND rating <= 5)    | Rating given (1-5 stars).                     |
| comments         | TEXT             | NULL                                             | Additional feedback.                           |
| created_at       | TIMESTAMP        | DEFAULT CURRENT_TIMESTAMP                        | When the rating was created.                   |

### 7. **Locations Table (Tracking the Vehicle):**
This table records each location pinged from the driver's phone during a trip. You can track the vehicle's movement by correlating these records with the `trip_id`.

| Column Name      | Data Type        | Constraints                                       | Description                                    |
|------------------|------------------|--------------------------------------------------|------------------------------------------------|
| location_id      | INT              | PRIMARY KEY, AUTO_INCREMENT                      | Unique identifier for each location.           |
| trip_id          | INT              | FOREIGN KEY (trips.trip_id)                      | Reference to the trip being tracked.           |
| driver_id        | INT              | FOREIGN KEY (drivers.driver_id)                  | Reference to the driver being tracked.         |
| latitude         | DECIMAL(9,6)     | NOT NULL                                         | Latitude of the vehicle at a certain time.     |
| longitude        | DECIMAL(9,6)     | NOT NULL                                         | Longitude of the vehicle at a certain time.    |
| timestamp        | TIMESTAMP        | NOT NULL                                         | Time of location capture (track when location is recorded). |

#### **Tracking the Vehicle Based on Trip Details:**
- When a trip is started, the **driver’s location** is tracked continuously or at regular intervals and recorded in the **Locations Table**.
- Each location update is tied to a specific `trip_id` and `driver_id`, so you can reconstruct the vehicle's movement during the trip.
- **Geospatial Data**: The columns `latitude` and `longitude` can be used to plot the driver's path on a map. You can visualize the real-time location or trip route by querying the **Locations Table** using the `trip_id`.

---

### **Table Relationships and How They Are Interconnected:**

| **Table**        | **Foreign Key**                | **Linked To**                                    | **Relationship**                               |
|------------------|-------------------------------|-------------------------------------------------|------------------------------------------------|
| Users            | -                             | -                                               | Stores general user data for both riders and drivers. |
| Drivers          | user_id (foreign key)          | Users.user_id                                   | Each driver is also a user, linked via `user_id`.|
| Vehicles         | driver_id (foreign key)        | Drivers.driver_id                               | Each vehicle is assigned to a driver.          |
| Trips            | rider_id (foreign key)         | Users.user_id                                   | The rider who requested the trip is linked via `rider_id`. |
| Trips            | driver_id (foreign key)        | Drivers.driver_id                               | The driver assigned to the trip is linked via `driver_id`. |
| Trips            | cancellation_reason, canceled_by | -                                               | Stores cancellation details if trip is canceled. |
| Locations        | trip_id (foreign key)          | Trips.trip_id                                   | Each location update during a trip is linked to a trip. |
| Locations        | driver_id (foreign key)        | Drivers.driver_id                               | Each location update is associated with the driver moving. |
| Payments         | trip_id (foreign key)          | Trips.trip_id                                   | Payments are made for a specific trip.         |
| Ratings          | trip_id (foreign key)          | Trips.trip_id                                   | Ratings are given for a specific trip.         |
| Ratings          | user_id (foreign key)          | Users.user_id                                   | The user (either rider or driver) giving the rating. |

#### **Summary of Relationships:**
- **Users** are generalized to represent both **riders** and **drivers**. 
- **Drivers** have their own details (e.g., license, availability) and are linked to **vehicles**.
- **Trips** connect the **rider** and the **driver**. If a trip is canceled, a reason and who canceled it are recorded.
- **Locations** track the movement of a driver during a trip using geospatial data.
- **Payments** and **Ratings** are tied to specific trips, ensuring traceability of financial transactions and feedback.