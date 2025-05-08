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

## 2. Property Management

### Functional Requirements
- Hosts can add, update, and delete property listings.
- Listings include title, description, price, photos, and availability.
- Each listing is tied to a host (user).

## 3. Booking System

### Functional Requirements
- Guests can book available properties for date ranges.
- Double bookings are prevented using date validation.
- Booking status includes pending, confirmed, canceled.
