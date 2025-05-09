# Airbnb Clone – Backend Requirements

## 1. User Authentication

### Functional Requirements
- Allow users to register as guests or hosts.
- Users must log in using email/password or OAuth (Google/Facebook).
- Secure authentication with JWT (JSON Web Tokens).
- Passwords must be hashed and securely stored.

### API Endpoints

| Method | Endpoint         | Description              |
|--------|------------------|--------------------------|
| POST   | /api/register    | Register a new user      |
| POST   | /api/login       | Login user               |
| POST   | /api/oauth       | OAuth login (Google/Fb)  |

### Input / Output

- Input (JSON):  
  ```json
  {
    "email": "user@example.com",
    "password": "securePass123",
    "role": "guest",
    "first_name": "John",
    "last_name": "Doe"
  }
- Output (JSON):  
  ```json
  {
  "token": "jwt_token_here",
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "role": "guest"
  }
}

### Validation Rules
- Email must be unique and valid format.
- Password must be ≥ 8 characters.
- All required fields must be provided.

### Performance
- Token response time: < 300ms.
- Must handle 10,000 concurrent sessions/min.

## 2. Property Management

### Functional Requirements
- Hosts can add, update, and delete their property listings.
- Listings must contain: title, description, location, price per night, availability dates, and amenities.
- Images should be uploaded and linked to each listing.
- Listings are associated with the host who created them.
- Only the host who created a listing can modify or delete it.

---

### API Endpoints

| Method | Endpoint              | Description                   | Access        |
|--------|-----------------------|-------------------------------|---------------|
| POST   | /api/properties       | Create new listing            | Host          |
| GET    | /api/properties       | Retrieve all listings         | Public        |
| GET    | /api/properties/:id   | View specific property        | Public        |
| PUT    | /api/properties/:id   | Update a listing              | Host (owner)  |
| DELETE | /api/properties/:id   | Delete a listing              | Host (owner)  |

---

### Request Example (POST /api/properties)

- Input (JSON):
  ```json
  {
    "title": "Cozy 2 Bedroom Apartment",
    "description": "Quiet and safe neighborhood near city center.",
    "location": "Accra, Ghana",
    "price_per_night": 120,
    "availability_start": "2025-06-01",
    "availability_end": "2025-06-30",
    "amenities": ["wifi", "kitchen", "air_conditioning"],
    "images": ["image1.jpg", "image2.jpg"],
  }

- Output (JSON):
  ```json
  {
    "id": "prop_8f1a23cd",
    "title": "Cozy 2 Bedroom Apartment",
    "status": "created"
  }

### Validation Rules
- title, description, location, price, and availability are mandatory.
- price_per_night must be a positive float.
- availability_start must be before availability_end.
- Images must be uploaded using multipart/form-data or pre-signed URL.
- Only authenticated users with host role can create or modify listings.

### Performance Criteria
- GET /api/properties supports pagination and filtering by location, price, and amenities.
- Top 100 listings are cached using Redis.
- Response time for read endpoints must be under 500ms.
- Full-text search enabled for title and description fields.

## 3. Booking System

### Functional Requirements

- Guests can book available properties for specific date ranges.
- The system must prevent double bookings through date validation.
- Booking statuses include: pending, confirmed, canceled, and completed.
- Each booking is associated with a guest (user) and a property.

### API Endpoints

| Method | Endpoint           | Description                    |
|--------|--------------------|--------------------------------|
| POST   | /api/bookings      | Create a new booking           |
| GET    | /api/bookings      | Retrieve bookings for a user   |
| GET    | /api/bookings/:id  | Retrieve a specific booking    |
| PUT    | /api/bookings/:id  | Update booking status          |
| DELETE | /api/bookings/:id  | Cancel a booking               |

### Request/Response Examples

- Request (Create Booking):
  ```json
  {
    "property_id": "prop-uuid-123",
    "start_date": "2025-06-01",
    "end_date": "2025-06-05"
  }

- Response
  ```json
  {
    "booking_id": "book-uuid-456",
    "status": "pending",
    "total_price": 400.00
  }

### Validation Rules

- ✅ Booking dates must not overlap with existing bookings (status: "pending" or "confirmed") for the same property.
- ✅ Users cannot book properties they own.
- ✅ Start date must be earlier than end date.
- ✅ All fields (`property_id`, `start_date`, `end_date`) are required.
- ✅ Backend must validate input types and required constraints.
- ✅ Application must return appropriate error codes (e.g., 400 for bad request, 409 for conflict).

---

### Database Design – Bookings Table

| Field         | Type      | Constraints                                          |
|---------------|-----------|------------------------------------------------------|
| id            | UUID      | Primary Key                                          |
| user_id       | UUID      | Foreign Key → users(id), NOT NULL                   |
| property_id   | UUID      | Foreign Key → properties(id), NOT NULL              |
| start_date    | DATE      | NOT NULL                                            |
| end_date      | DATE      | NOT NULL                                            |
| total_price   | DECIMAL   | NOT NULL                                            |
| status        | ENUM      | Values: `pending`, `confirmed`, `canceled`, `completed` |
| created_at    | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP                           |
| updated_at    | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |

Indexes:

- 🔹 Index on `user_id`
- 🔹 Composite index on (`property_id`, `start_date`, `end_date`)
- 🔹 Index on `status` for fast filtering
