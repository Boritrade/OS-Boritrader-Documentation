# Primary Database Table Summary
**Boritrade, LLC**  
**Updated Oct 21, 2024**  

### 1. **Users Table**
Stores basic user account information.

| **Field**      | **Data Type**     | **Description**                  |
|----------------|-------------------|----------------------------------|
| id (PK)        | `SERIAL`          | Unique identifier for each user. |
| username       | `VARCHAR(50)`     | User's display name.             |
| email          | `VARCHAR(100)`    | User's email address.            |
| password_hash  | `TEXT`            | Secure hashed password.          |
| created_at     | `TIMESTAMP`       | Account creation timestamp.      |

**Entries per user**: One.

**Constraints**:
- `username` and `email` must be unique.
- Deleting a user cascades deletions across related entities.

---

### 2. **UserProfiles Table**
Stores additional profile information for each user.

| **Field**      | **Data Type**     | **Description**                 |
|----------------|-------------------|----------------------------------|
| user (PK, FK)  | `INT`             | Links to the Users table.        |
| full_name      | `VARCHAR(100)`    | User's full name.                |
| date_of_birth  | `DATE`            | User's date of birth.            |
| address        | `TEXT`            | User's address.                  |

**Entries per user**: One.

**Constraints**:
- One-to-one relationship with the Users table.
- Deleting a user deletes their profile.

---

### 3. **ApiKeys Table**
Stores API keys for external services linked to each user.

| **Field**      | **Data Type**     | **Description**                   |
|----------------|-------------------|-----------------------------------|
| id (PK)        | `SERIAL`          | Unique identifier for API keys.   |
| user_id (FK)   | `INT`             | Links to the Users table.         |
| api_key        | `TEXT`            | API key for external service.     |
| api_secret     | `TEXT`            | API secret key.                   |
| api_tld        | `TEXT`            | Region code (e.g., 'us' for USA). |
| created_at     | `TIMESTAMP`       | API key creation timestamp.       |
| last_used      | `TIMESTAMP`       | Last usage timestamp.             |
| permissions    | `TEXT`            | Optional permissions.             |

**Entries per user**: Many.

**Constraints**:
- Foreign key to Users table.
- Deleting a user deletes their API keys.

---

### 4. **TradingPreferences Table**
Stores trading preferences and modes for each user.

| **Field**             | **Data Type**     | **Description**                   |
|-----------------------|-------------------|----------------------------------|
| user_id (PK, FK)      | `INT`             | Links to the Users table.        |
| auto_purchase_enabled | `BOOLEAN`         | Enable/disable auto purchases.   |
| notification_only_enabled | `BOOLEAN`     | Enable/disable notifications only. |
| paper_trading_enabled | `BOOLEAN`         | Enable/disable paper trading.    |
| live_trading_enabled  | `BOOLEAN`         | Enable/disable live trading.     |

**Entries per user**: One.

**Constraints**:
- One-to-one relationship with the Users table.
- `auto_purchase_enabled` and `notification_only_enabled` cannot both be `TRUE`.
- `paper_trading_enabled` and `live_trading_enabled` cannot both be `TRUE`.
- Deleting a user deletes their trading preferences.

---

### 5. **NotificationSettings Table**
Stores notification preferences for each user.

| **Field**      | **Data Type**     | **Description**                  |
|----------------|-------------------|----------------------------------|
| user_id (PK, FK) | `INT`          | Links to the Users table.        |
| email_enabled  | `BOOLEAN`         | Enable/disable email notifications. |
| sms_enabled    | `BOOLEAN`         | Enable/disable SMS alerts.       |

**Entries per user**: One.

**Constraints**:
- One-to-one relationship with the Users table.
- Deleting a user deletes their notification settings.

---

### 6. **UserAlgorithmSettings Table**
Stores algorithm-specific configurations for each user.

| **Field**            | **Data Type**     | **Description**                  |
|----------------------|-------------------|----------------------------------|
| id (PK)              | `SERIAL`          | Unique identifier.               |
| user_id (FK)         | `INT`             | Links to the Users table.        |
| algorithm_id (FK)    | `INT`             | Links to the Algorithms table.   |
| tickers              | `TEXT[]`          | List of tickers (e.g., BTC, ETH). |
| quantity             | `NUMERIC(10, 2)`  | Quantity to trade.               |
| execution_frequency  | `INTERVAL`        | Execution frequency for the algorithm. |

**Entries per user**: Many.

**Constraints**:
- Foreign key to Users and Algorithms tables.
- Unique combination of `user_id` and `algorithm_id`.
- Deleting a user deletes their algorithm settings.

---

#### 7. **Notifications Table**
This table stores notifications sent to users from their algorithms.

| **Field**          | **Data Type**     | **Description**                                                                 |
|--------------------|-------------------|---------------------------------------------------------------------------------|
| `notification_id` (PK) | `SERIAL`          | Unique identifier for each notification.                                         |
| `user_id` (FK)     | `INT`             | Links to the `AuthUser` table; indicates the recipient of the notification.      |
| `algorithm_id` (FK)| `INT` (nullable)  | Links to the `Algorithm` table; specifies the algorithm associated with the notification. |
| `message`          | `TEXT`            | Raw content of the notification.                                                |
| `ticker`           | `VARCHAR(50)` (nullable) | A short string representing a stock or financial ticker.                           |
| `status`           | `VARCHAR(20)` (nullable) | Current status of the notification, such as "Buy," "Sell," or "Do Nothing."         |
| `created_at`       | `TIMESTAMP`       | The timestamp when the notification was created. Automatically set on creation. |

#### **Entries per User**
- Each user can have multiple notifications.

#### **Constraints**
1. **Foreign Key: `user_id`**  
   - Links to the `AuthUser` table.  
   - Deleting a user will delete all their associated notifications (`CASCADE` behavior).

2. **Foreign Key: `algorithm_id`**  
   - Links to the `Algorithm` table.  
   - Nullable field, meaning not all notifications must be linked to an algorithm.

#### **Additional Notes**
- **Ordering**: Notifications are sorted in descending order of their `created_at` timestamp (`ordering = ['-created_at']`).
- **Custom Manager**: The table uses `NotificationManager` to extend query capabilities and enforce user entry limits.

---

### 8. **Orders Table**
Stores user-submitted orders.

| **Field**      | **Data Type**     | **Description**                  |
|----------------|-------------------|----------------------------------|
| id (PK)        | `SERIAL`          | Unique order identifier.         |
| user_id (FK)   | `INT`             | Links to the Users table.        |
| pair           | `VARCHAR(10)`     | Trading pair (e.g., BTC/USD).    |
| order_type     | `VARCHAR(10)`     | Type of order ('buy' or 'sell'). |
| amount         | `NUMERIC(18, 8)`  | Amount of the asset.             |
| price          | `NUMERIC(18, 8)`  | Order price.                     |
| status         | `VARCHAR(20)`     | Order status.                    |
| timestamp      | `TIMESTAMP`       | Order creation timestamp.        |

**Entries per user**: Many.

**Constraints**:
- Foreign key to Users table.
- `order_type` must be either 'buy' or 'sell'.
- Deleting a user deletes their orders.

---

### 9. **Portfolios Table**
Stores current portfolio entries for each user.

| **Field**      | **Data Type**     | **Description**                  |
|----------------|-------------------|----------------------------------|
| id (PK)        | `SERIAL`          | Unique portfolio entry identifier. |
| user_id (FK)   | `INT`             | Links to the Users table.        |
| asset          | `VARCHAR(10)`     | Asset in the portfolio.          |
| quantity       | `NUMERIC(18, 8)`  | Quantity held.                   |
| average_buy_price | `NUMERIC(18, 8)`| Average buy price.               |

**Entries per user**: Many.

**Constraints**:
- Foreign key to Users table.
- Deleting a user deletes their portfolio entries.

---

### 10. **Algorithms Table**
Stores algorithm definitions users can configure.

| **Field**      | **Data Type**     | **Description**                  |
|----------------|-------------------|----------------------------------|
| id (PK)        | `SERIAL`          | Unique algorithm identifier.     |
| display_name   | `VARCHAR(100)`    | Algorithm display name.          |
| internal_name  | `VARCHAR(100)`    | Internal algorithm name.         |
| description    | `TEXT`            | Details on how algorithm works.  |

**Entries per user**: Internally managed.

**Constraints**:
- `display_name` and `internal_name` must be unique.

---

### 11. **AdminUsers Table**
Stores admin user account information.

| **Field**      | **Data Type**     | **Description**                  |
|----------------|-------------------|----------------------------------|
| id (PK)        | `SERIAL`          | Unique admin identifier.         |
| username       | `VARCHAR(50)`     | Admin's username.                |
| password_hash  | `TEXT`            | Secure hashed password.          |
| role           | `VARCHAR(50)`     | Admin's role.                    |

**Entries per user**: One per admin.

**Constraints**:
- `username` must be unique.

---

### 12. **SystemLogs Table**
Stores system events and logs related to user actions.

| **Field**      | **Data Type**     | **Description**                  |
|----------------|-------------------|----------------------------------|
| id (PK)        | `SERIAL`          | Unique log identifier.           |
| event          | `VARCHAR(100)`    | Event description.               |
| description    | `TEXT`            | Detailed event description.      |
| timestamp      | `TIMESTAMP`       | Log timestamp.                   |
| user_id (FK)   | `INT`             | Links to the Users table.        |

**Entries per user**: Many.

**Constraints**:
- Logs remain even if the associated user is deleted.

---

### 13. **LoginAttempts Table**
Stores user login attempts.

| **Field**      | **Data Type**     | **Description**                  |
|----------------|-------------------|----------------------------------|
| id (PK)        | `SERIAL`          | Unique attempt identifier.       |
| user_id (FK)   | `INT`             | Links to the Users table.        |
| timestamp      | `TIMESTAMP`       | Attempt timestamp.               |
| ip_address     | `INET`            | IP address of the user.          |
| success        | `BOOLEAN`         | Whether the attempt was successful. |

**Entries per user**: Many.

**Constraints**:
- Logs remain even if the associated user is deleted.

---

### 14. **PasswordResets Table**
Stores password reset requests.

| **Field**      | **Data Type**     | **Description**                  |
|----------------|-------------------|----------------------------------|
| token (PK)     | `TEXT`            | Unique reset token.              |
| user_id (FK)   | `INT`             | Links to the Users table.        |
| created_at     | `TIMESTAMP`       | Token creation timestamp.        |
| expires_at     | `TIMESTAMP`       | Token expiration timestamp.      |
| used           | `BOOLEAN`         | Whether the token was used.      |

**Entries per user**: Many.

**Constraints**:
- Tokens remain even if the associated user is deleted.



