# SmartEvent - Event Discovery & Ticket Booking System

SmartEvent is a full-stack event discovery and ticket booking application built with **FastAPI, SQLAlchemy, SQLite, React, Vite, Axios, JWT authentication, bcrypt, and QR code generation**.

The project brings into one application: event discovery and ticket booking, role-based dashboards, simulated payments and coupons, reviews and notifications, live availability, recommendations, wishlist, profile management, event announcements, and a built-in rule-based chatbot. Payments are simulated; the application does not charge real money or require OpenAI.

## 1. Project Setup

### Technologies

    Technology        Purpose

---

    Python            Backend programming
    FastAPI           REST API and WebSocket endpoint
    Uvicorn           Runs the backend server
    SQLAlchemy        Database models and queries
    SQLite            Default local database
    Pydantic          Request validation
    JWT               Authentication for protected endpoints
    bcrypt            Password hashing
    python-dotenv     Loads backend environment variables
    React             Frontend user interface
    Vite              Frontend development server and build
    Axios             Frontend API requests
    qrcode / Pillow   Generates ticket QR images

### Prerequisites

- Python 3.10 or a compatible newer version.
- Node.js and npm.
- VS Code or another code editor.
- A modern web browser.

### Project structure

```text
SmartEvent/
 ── backend/
   ── main.py
   ── requirements.txt
   ── .env
── frontend/
   ── index.html
   ── package.json
   ── package-lock.json
   ── vite.config.mjs
   ── src/
       ── App.jsx
       ── api.js
       ── index.css
       ── main.jsx
       ── components/
       ── pages/
── .gitignore
```

## 2. Database Configuration

SQLite is the default database. The application creates its tables when the backend starts and includes a small migration routine for certain older SQLite databases.

Default setting:

```env
DATABASE_URL=sqlite:///./smartevent.db
```

The database file is created relative to the backend directory when this default URL is used. Do not delete an existing database just to restart the application; it may contain user accounts, bookings, event dates, payments, or tickets.

### Database tables

---

    Table                 Purpose

---

`users` Login identity, hashed password, and role

`events` Event details, category, location, date, price, capacity, organizer, and status

`bookings` User bookings, ticket quantity, subtotal, discount, final amount, and status

`tickets` Unique ticket code and QR ticket reference

`notifications` User notifications and read status

`payments` Simulated payment method, status, transaction reference, amount, and timestamp

`coupons` Discount rules, expiry, active status, and usage count

`reviews` User event ratings and review text

`wishlist` Events saved by users

`event_updates` Announcements attached to events

`event_views` Viewed-event activity used for recommendations

`user_profiles` Optional profile location and profile picture path

---

### Main relationships

```text
User 1 ────< Booking >──── 1 Event
Booking 1 ────< Ticket
Booking 1 ────< Payment
User 1 ────< Notification
User 1 ────< Review >──── 1 Event
User 1 ────< Wishlist >─── 1 Event
Event 1 ────< EventUpdate
User 1 ────< EventView >── 1 Event
User 1 ──── 1 UserProfile
```

## 3. Environment Variables

Create `backend/.env`. Use your own private secret; do not copy a secret from a public example or share it.

Example development configuration:

```env
SECRET_KEY=replace with a long random secret
DATABASE_URL=sqlite:///./smartevent.db
CORS_ORIGINS=http://localhost:5173
ADMIN_EMAILS=saptha@gmail.com
```

Generate a secret in the VS Code terminal with:

```powershell
python -c "import secrets; print(secrets.token_hex(32))"
```

Copy the generated value after `SECRET_KEY=` in `backend/.env`.

---

    Variable             Purpose

---

`SECRET_KEY` Signs JWT access tokens

`DATABASE_URL` Database connection

`CORS_ORIGINS` Allowed frontend origins, separated by commas if needed

`ADMIN_EMAILS` Email addresses that should be assigned the `ADMIN` role when those accounts exist

The backend prints a warning and uses a temporary random key if `SECRET_KEY` is missing. Configure a persistent key to avoid invalidating login sessions when the backend restarts.

The frontend API client defaults to `http://localhost:8000`. If the frontend needs another backend URL, configure `VITE_API_URL` in `frontend/.env`:

```env
VITE_API_URL=http://localhost:8000
```

Do not commit `.env` files, real secrets, or database files to source control.

## 4. Install Dependencies

Run commands from the project root unless a step says otherwise.

### Backend

```powershell
cd backend
python -m pip install -r requirements.txt
```

### Frontend

Open a second terminal:

```powershell
cd frontend
npm install
npm.cmd install
```

## 5. Run the Application

### Terminal 1 - FastAPI backend

```powershell
cd backend
python -m uvicorn main:app --reload
```

Backend URL:

```text
http://localhost:8000
```

Interactive API documentation:

```text
http://localhost:8000/docs
```

### Terminal 2 -- React frontend

```powershell
cd frontend
npm run dev
```

If PowerShell blocks npm scripts:

```powershell
npm.cmd run dev
```

Open the local URL printed by Vite, normally:

```text
http://localhost:5173
```

Keep both terminals running while using the application.

## 6. Authentication, Event Discovery and Booking

### Registration and login

Users register with a username, email, and password. The backend validates the input, hashes the password, and returns a JWT after successful login. The frontend stores the token and attaches it to protected API requests.

Main endpoints:

```text
POST /auth/register
POST /auth/login
GET  /auth/profile
PUT  /auth/profile
```

### Event discovery

Users can browse event cards, open an event's details, and search/filter available events.

```text
GET /events
GET /events/{event_id}
GET /events/categories/list
```

Supported query filters include category, text search, location, date range, minimum price, maximum price, and sorting. The homepage provides the event discovery interface.

### Booking workflow

```text
Event details
    ↓
Choose ticket quantity
    ↓
POST /bookings
    ↓
Validate user, event, quantity, and ticket availability
    ↓
Create pending booking and payment record
    ↓
Payment checkout
    ↓
Simulated payment result
    ↓
Confirmation and digital ticket after success
```

The backend checks ticket quantity and available stock. The maximum quantity in a single booking is 10. A booking belongs to the authenticated user; users cannot retrieve other users' personal booking history through the normal user endpoint.

```text
POST /bookings
GET  /bookings
POST /bookings/{booking_id}/cancel
```

### Digital tickets and QR codes

Successful payment produces a ticket with a unique ticket code. The QR endpoint generates the QR image for display or download.

```text
GET /tickets
GET /tickets/{ticket_code}/qr
GET /tickets/verify/{ticket_code}
```

The Tickets page lists the current user's tickets. The verification endpoint returns ticket verification information; it should be exposed to trusted event staff in a real production deployment.

### Notifications and reminders

Users can view their notification history, see unread notifications in the navigation dropdown, and mark their own notifications as read.

```text
GET   /notifications
PATCH /notifications/{notification_id}/read
```

Booking and payment actions create notifications. The reminder logic checks for upcoming booked events when the relevant notification endpoint is requested; it is not a continuously running background scheduler.

## 7. Roles, Organizer Tools and Admin Analytics

SmartEvent uses three roles:

---

    Role            Main responsibilities

---

`USER` Discover events, book tickets, pay through simulation, view tickets, manage wishlist/profile, and review eligible events

`ORGANIZER` Create and manage owned events, review bookings for those events, post event updates, and view event analytics

`ADMIN` View platform users, events, bookings, manage roles and coupons, and view platform analytics

### Organizer endpoints

```text
POST /events
GET  /organizer/events
PUT  /events/{event_id}
POST /events/{event_id}/cancel
GET  /organizer/bookings
GET  /organizer/events/{event_id}/bookings
GET  /organizer/analytics
```

Organizer routes check the user's role and verify event ownership before allowing changes.

### Admin endpoints

```text
GET   /admin/users
PATCH /admin/users/{user_id}/role
GET   /admin/events
GET   /admin/bookings
GET   /admin/analytics
GET   /admin/coupons
POST  /admin/coupons
```

The admin dashboard presents platform totals such as users, events, tickets sold, bookings, and revenue. It also displays daily ticket sales, monthly booking trends, popular events, and top-revenue events. Date filters can be applied to analytics where supported.

## 8. Simulated Payments and Coupons

### Payment simulation

SmartEvent uses a local simulated payment flow.
Payment records contain a booking reference, payment method, status, transaction reference, amount, and creation time. The checkout page shows the booking summary and coupon breakdown. The confirmation page displays the payment result and offers retry for eligible failed payments.

```text
GET  /payments
GET  /payments/{payment_id}
POST /payments/{payment_id}/simulate
```

The simulation accepts a result handled by the backend. A successful result updates the booking and issues the ticket; a failed result must not be presented as a successful payment.

### Coupons and discounts

Coupon validation checks the code, active state, expiry, minimum booking amount, usage limit, and discount rules. Percentage and fixed-amount discounts are supported.

```text
POST /coupons/validate
GET  /admin/coupons
POST /admin/coupons
POST /payments/{payment_id}/coupon
```

Seeded development coupon examples:

    Coupon           Discount   Minimum subtotal

---

    `WELCOME10`           10%               ₹100
    `SAVE50`        ₹50 fixed               ₹250

Coupon availability depends on the current database values and usage limits. The checkout page shows the subtotal, discount, and final amount. Do not assume a seeded coupon will remain valid after its usage limit or expiry rules change.

### Reviews and ratings

Users can submit a rating from 1 to 5 and optional review text for eligible events.Only after the event completed the user can submit a rating. The backend checks booking eligibility and ownership for review updates/deletions.

```text
GET    /events/{event_id}/reviews
POST   /events/{event_id}/reviews
PUT    /reviews/{review_id}
DELETE /reviews/{review_id}
```

The event detail page displays reviews and rating information. Review submission is intended for users who have attended/completed the relevant event.

### Notifications

Notification types include `EVENT`, `BOOKING`, `PAYMENT`, and `SYSTEM`. Notifications can be generated by booking and payment actions and event changes. The user interface includes an unread-count indicator, dropdown, history page, and mark-as-read action.

### Event search and UI

The discovery page supports text search and filters for category, location, date range, and price range, with sorting options for date, popularity, and price. The React interface includes event cards, event details, checkout, confirmation, ticket and booking pages, loading/error messages, and responsive styling.

## 9. Real-Time Availability and Personalization

### Live ticket availability

The backend provides an availability endpoint and WebSocket endpoint:

```text
GET       /events/{event_id}/availability
WebSocket /ws/events/{event_id}/availability
```

The frontend uses availability updates to refresh the displayed ticket count and disable booking when no tickets remain. Booking logic also checks stock on the backend so the frontend is not the only protection against overselling.

### Recommendations and activity tracking

```text
POST /events/{event_id}/view
GET  /recommendations
```

The application records event views and uses activity, booked categories, location preference, and event popularity to suggest relevant events. The homepage presents recommendations and trending events where data is available.

### Wishlist

```text
GET    /wishlist
POST   /wishlist/{event_id}
DELETE /wishlist/{event_id}
```

Users can save events from event cards, open their wishlist, remove saved events, and navigate from a saved event to its detail page to continue booking. Wishlist data is scoped to the authenticated user.

### Live event updates

```text
GET  /events/{event_id}/updates
POST /events/{event_id}/updates
```

Organizers and administrators can post event announcements. Users can view updates on the event detail page, helping them find schedule or event information without searching through unrelated pages.

### Profile management

```text
GET  /auth/profile
PUT  /auth/profile
GET  /auth/profile/summary
POST /auth/profile/picture
```

Users can update their username and location, optionally upload a profile picture, and view a summary of bookings, tickets, and saved events. Profile operations are tied to the authenticated user.

### Built-in chatbot assistant

```text
POST /assistant/chat
```

The floating SmartEvent Assistant uses built-in rule-based logic; no OpenAI API key or external AI service is required. It can help with supported requests such as finding upcoming events by category/location, listing trending events, showing the user's bookings, and providing basic ticket/payment guidance. Responses are limited to the implemented rules and the information stored in SmartEvent.

## 10. Frontend Pages and Components

The React application uses a single-page navigation flow.

Main pages:

- Register and Login.
- Home / Event Discovery.
- Event Details.
- Payment Checkout.
- Booking Confirmation.
- Booking History.
- Tickets and QR codes.
- Notifications.
- Organizer Dashboard.
- Admin Dashboard.
- Wishlist.
- User Profile.

Reusable components include the navigation bar, event cards, ticket cards, protected-route wrapper, notification dropdown, and floating chatbot assistant.

The API client in `frontend/src/api.js` uses `VITE_API_URL` or defaults to `http://localhost:8000`. Authenticated calls include the JWT bearer token.

## 11. Security and Validation

- Passwords are stored as hashes rather than plain-text passwords.
- JWT authentication protects user-specific operations.
- Role checks protect organizer and admin endpoints.
- Organizer ownership is checked before editing or cancelling owned events.
- Booking, payment, coupon, review, wishlist, and profile operations validate relevant ownership or rules.
- Pydantic models validate API request data.
- Ticket quantities and review ratings have defined input limits.
- CORS origins are configurable through the backend environment.
- Keep `SECRET_KEY` and `.env` private; never put secrets in React code.
- The simulated payment flow is for development and learning, not a real payment processor.

## 12. Quick Verification

After starting both servers, test registration/login, event discovery and filters, booking and simulated payment, QR tickets, notifications, coupon validation, reviews, and wishlist/profile pages. Then sign in with organizer and admin accounts to verify their dashboards and permissions. For API errors, inspect the response in `http://localhost:8000/docs` and the browser developer console.

Run basic syntax/build checks:

```powershell
cd backend
python -m py_compile main.py
cd ../frontend
npm run build
```

These checks do not replace full end-to-end browser or concurrent-load testing.

## 13. Sample Development Data

On startup, the backend adds the following events only when an event with the same title does not already exist:

    Event                   Category   Location                   Price   Initial ticket capacity

---

    Hyderabad Tech Meetup   Tech       HITEX, Hyderabad            ₹999                       100
    Live Music Night        Music      Gachibowli, Hyderabad     ₹1,499                       200
    Business Conference     Business   HICC, Hyderabad           ₹1,999                       150
    Sports Meet             Sports     LB Stadium, Hyderabad       ₹799                       300

The dates and ticket settings of existing records are preserved. The actual values displayed by the app come from the current database.

## 14. Typical Startup Order

1.  Configure `backend/.env`.
2.  Install backend requirements.
3.  Start FastAPI in the first terminal.
4.  Install frontend dependencies if this is the first run.
5.  Start Vite in the second terminal.
6.  Open the frontend URL.
7.  Register or log in, then test features using the role appropriate to each workflow.

## Author

Saptharishi reddy D
