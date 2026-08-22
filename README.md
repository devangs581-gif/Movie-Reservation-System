# 🎬 Movie Reservation System

A RESTful backend API for a movie theater reservation platform, built with **Node.js**, **Express**, and **MongoDB**. It supports user authentication, movie and showtime management, seat reservations with live availability tracking, and role-based access for admins.

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Environment Variables](#environment-variables)
- [Default Admin Account](#default-admin-account)
- [API Reference](#api-reference)
- [Data Models](#data-models)
- [Testing the API](#testing-the-api)
- [License](#license)

## Overview

The Movie Reservation System exposes a JSON API for managing a cinema's core operations:

- Visitors can register/log in and browse movies and showtimes.
- Authenticated users can reserve seats for a showtime and manage their own bookings.
- Admins can manage the full catalog of movies and showtimes, and view all reservations across the system.

Seat availability is tracked per showtime and automatically adjusted whenever a reservation is created or cancelled, preventing overbooking.

## Features

- 🔐 JWT-based authentication with password hashing (bcrypt)
- 👤 Role-based access control (`user` / `admin`)
- 🎞️ Movie catalog management (create, read, update, delete)
- 🕒 Showtime scheduling tied to movies, with seat capacity tracking
- 🎟️ Seat reservation and cancellation with automatic seat count adjustment
- 🛡️ Protected routes via middleware for authentication and authorization
- ⚙️ Automatic bootstrap of a default admin account on server startup

## Tech Stack

| Layer          | Technology            |
|----------------|------------------------|
| Runtime        | Node.js                |
| Framework      | Express.js              |
| Database       | MongoDB (via Mongoose)  |
| Auth           | JSON Web Tokens (JWT)   |
| Password Hashing | bcryptjs              |
| Dev Tooling    | nodemon                 |

## Project Structure

```
Movie-Reservation-System-main/
├── src/
│   ├── controllers/
│   │   ├── authController.js
│   │   ├── movieController.js
│   │   ├── showtimeController.js
│   │   └── reservationController.js
│   ├── middlewares/
│   │   └── auth.js
│   ├── models/
│   │   ├── User.js
│   │   ├── Movie.js
│   │   ├── Showtime.js
│   │   └── Reservation.js
│   ├── routes/
│   │   ├── authRoutes.js
│   │   ├── movieRoutes.js
│   │   ├── showtimeRoutes.js
│   │   └── reservationRoutes.js
│   ├── utils/
│   │   └── initAdmin.js
│   └── server.js
├── package.json
└── LICENSE
```

## Getting Started

### Prerequisites

- Node.js (v16+ recommended)
- A running MongoDB instance (local or hosted, e.g. MongoDB Atlas)

### Installation

1. Clone the repository:
   ```bash
   git clone <repository-url>
   cd Movie-Reservation-System-main
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Create a `.env` file in the project root (see [Environment Variables](#environment-variables)).

4. Start the server:
   ```bash
   npm start
   ```
   Or, for auto-reload during development:
   ```bash
   npm run dev
   ```

The API will be available at `http://localhost:3000` (or the port set in your `.env` file).

## Environment Variables

Create a `.env` file with the following keys:

```env
PORT=3000
MONGODB_URI=mongodb://localhost:27017/movie_reservation_system
JWT_SECRET=your_jwt_secret_here
```

| Variable      | Description                                      |
|---------------|---------------------------------------------------|
| `PORT`        | Port the Express server listens on                |
| `MONGODB_URI` | MongoDB connection string                          |
| `JWT_SECRET`  | Secret key used to sign and verify JWT tokens      |

## Default Admin Account

On startup, `initAdmin.js` checks whether an admin user already exists. If not, it automatically creates one:

- **Username:** `admin`
- **Email:** `admin@example.com`
- **Password:** `adminpassword`

> ⚠️ **Security note:** Change this password immediately after first login in any non-local environment, or update `initAdmin.js` to source credentials from environment variables before deploying.

## API Reference

All request/response bodies are JSON. Protected routes require an `Authorization: Bearer <token>` header.

### Auth — `/api/auth`

| Method | Endpoint    | Access | Description               |
|--------|-------------|--------|----------------------------|
| POST   | `/register` | Public | Register a new user (`username`, `email`, `password`) |
| POST   | `/login`    | Public | Log in with `email` and `password`, returns a JWT      |

### Movies — `/api/movies`

| Method | Endpoint | Access       | Description               |
|--------|----------|--------------|----------------------------|
| GET    | `/`      | Public       | List all movies            |
| POST   | `/`      | Admin only   | Create a movie              |
| PUT    | `/:id`   | Admin only   | Update a movie              |
| DELETE | `/:id`   | Admin only   | Delete a movie              |

Movie fields: `title`, `description`, `posterImage`, `genre` (all required).

### Showtimes — `/api/showtimes`

| Method | Endpoint | Access       | Description                        |
|--------|----------|--------------|--------------------------------------|
| GET    | `/`      | Public       | List all showtimes (with movie populated) |
| POST   | `/`      | Admin only   | Create a showtime                    |
| PUT    | `/:id`   | Admin only   | Update a showtime                    |
| DELETE | `/:id`   | Admin only   | Delete a showtime                    |

Showtime fields: `movie` (Movie ID), `startTime`, `endTime`, `totalSeats`, `availableSeats`. `availableSeats` is automatically capped so it never exceeds `totalSeats`.

### Reservations — `/api/reservations`

| Method | Endpoint | Access          | Description                                  |
|--------|----------|-----------------|------------------------------------------------|
| POST   | `/`      | Authenticated   | Reserve seats (`showtime` ID, `seats` count)   |
| GET    | `/user`  | Authenticated   | Get the current user's reservations            |
| DELETE | `/:id`   | Authenticated   | Cancel one of your own reservations             |
| GET    | `/all`   | Admin only      | Get all reservations across all users           |

Creating a reservation checks that enough seats are available and decrements `availableSeats` on the showtime; cancelling a reservation restores those seats.

## Data Models

**User**
- `username`, `email` (unique) — required
- `password` — hashed with bcrypt before saving
- `role` — `user` (default) or `admin`

**Movie**
- `title`, `description`, `posterImage`, `genre` — required

**Showtime**
- `movie` — reference to `Movie`
- `startTime`, `endTime`
- `totalSeats`, `availableSeats` (auto-clamped so `availableSeats ≤ totalSeats`)

**Reservation**
- `user` — reference to `User`
- `showtime` — reference to `Showtime`
- `seats` — number of seats booked
- `createdAt` — timestamp (defaults to now)

## Testing the API

Use Postman, Insomnia, or `curl` to exercise the endpoints. For protected routes, include the JWT returned from `/api/auth/login` or `/api/auth/register`:

```
Authorization: Bearer <your_jwt_token>
```

Example flow:
1. `POST /api/auth/register` → get a token
2. `POST /api/movies` as admin → create a movie
3. `POST /api/showtimes` as admin → schedule a showtime for that movie
4. `POST /api/reservations` as a user → book seats

