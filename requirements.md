# 🏡 Airbnb Clone Backend - Feature Specifications

This document outlines the technical and functional requirements for the key backend features of the Airbnb Clone project.  
It serves as the reference for API design, validation rules, and performance standards.

---

## ✅ 1. User Authentication

### API Endpoints
- `POST /users/register/`
- `POST /users/login/`
- `GET /users/{user_id}/`
- `PATCH /users/{user_id}/`
- `DELETE /users/{user_id}/`

### Input/Output Specifications

#### Register (POST `/users/register/`)
**Input:**
- `first_name`: string, required
- `last_name`: string, required
- `email`: string (must be unique), required
- `password`: string (min 8 chars), required
- `phone_number`: string, optional (format: 2547XXXXXXXX)

**Output:**
- `user_id`: UUID
- `first_name`, `last_name`, `email`
- `created_at`

#### Login (POST `/users/login/`)
**Input:**
- `email`: string, required
- `password`: string, required

**Output:**
- `token`: JWT or session token
- `user_id`, `email`, `role`

### Validation Rules
- `email` must be valid and unique.
- `password` must be at least 8 characters.
- `phone_number` format must match Kenyan pattern (e.g., 25472XXXXXXX).

### Performance Criteria
- Login and registration endpoints must respond within **300-500ms** under normal load.
- `email` field is indexed to support fast lookups.
- System should handle 10,000+ users without degraded performance.
- Login attempts should be rate-limited to prevent brute-force attacks.

---

## ✅ 2. Property Management

### API Endpoints
- `POST /properties/`
- `GET /properties/`
- `GET /properties/{property_id}/`
- `PATCH /properties/{property_id}/`
- `DELETE /properties/{property_id}/`

### Input/Output Specifications

#### Create Property (POST `/properties/`)
**Input:**
- `host_id`: UUID (must reference existing user)
- `title`: string, required
- `description`: string, optional
- `location`: string, required
- `price_per_night`: decimal, required
- `max_guests`: integer, required

**Output:**
- `property_id`: UUID
- `created_at`
- All property details

#### List Properties (GET `/properties/`)
**Output:**
- Array of properties, paginated (e.g., 10 per page)
- Each includes `property_id`, `title`, `price_per_night`, `location`

### Validation Rules
- `price_per_night` must be a positive decimal.
- `max_guests` must be an integer ≥ 1.
- `host_id` must exist in the `User` table.

### Performance Criteria
- Property listings should respond in **under 500ms** for paginated queries.
- Listings support filtering (by location, price) and sorting (e.g., newest first).
- Indexed fields: `location`, `price_per_night`.
- Backend should scale to efficiently handle 50,000+ property records.
- Create, update, and delete actions should complete within 1 second.

---

## ✅ 3. Booking System

### API Endpoints
- `POST /bookings/`
- `GET /bookings/`
- `GET /bookings/{booking_id}/`
- `PATCH /bookings/{booking_id}/`
- `DELETE /bookings/{booking_id}/`

### Input/Output Specifications

#### Create Booking (POST `/bookings/`)
**Input:**
- `user_id`: UUID (must reference existing user)
- `property_id`: UUID (must reference existing property)
- `start_date`: date, must be today or later
- `end_date`: date, must be after `start_date`
- `total_price`: decimal (computed, not user input)

**Output:**
- `booking_id`: UUID
- All booking details
- `status`: defaults to `'pending'`

#### List Bookings (GET `/bookings/`)
**Output:**
- Array of bookings with `booking_id`, `property_id`, `start_date`, `end_date`, `status`

### Validation Rules
- `start_date` must be today or a future date.
- `end_date` must be after `start_date`.
- Overlapping bookings for the same property are disallowed.
- `total_price` is computed based on nights × `price_per_night`.

### Performance Criteria
- Booking creation (with availability checks) must respond in **under 500ms**.
- Conflict detection uses indexed fields: `property_id`, `start_date`, `end_date`.
- System prevents concurrent double-bookings through transaction safety.
- Must handle **5,000+ bookings per day** with consistent response times.
- Booking listings should return paginated results within 400ms.

---
