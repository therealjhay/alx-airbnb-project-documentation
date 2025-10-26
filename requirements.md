Airbnb Clone Backend – Requirements
1. User Authentication
Overview
Handles secure registration, login (JWT/OAuth), and role-based access for guests, hosts, and admins.

Endpoints
POST /api/auth/register

Input:
{ "first_name": String, "last_name": String, "email": String, "password": String, "role": "guest"|"host" }

Output:
201 Created – { "user_id": UUID, "token": JWT }
400 Bad Request – Validation or duplicate email

POST /api/auth/login

Input:
{ "email": String, "password": String }

Output:
200 OK – { "token": JWT, "role": String }
401 Unauthorized – Wrong credentials

POST /api/auth/oauth/<provider>

Input:
{ "access_token": String }

Output:
200 OK – { "token": JWT, ...userDetails }
400 Bad Request – Token invalid

Validation
Email unique, valid format; password minimum 8 chars; role must be “guest” or “host”.

Must hash passwords using strong algorithm (e.g., bcrypt).

JWT tokens expire after 1 hour.

Performance
Registration and login < 1.5 seconds under normal load.

Prevent brute-force attacks via rate limiting (max 5 requests/min per IP).

2. Property Management
Overview
Allows hosts to create, edit, view, and delete property listings with amenities, images, and pricing.

Endpoints
POST /api/properties

Input:
Auth JWT,
{ "name": String, "description": String, "location": String, "pricepernight": Decimal, "amenities": [String], "images": [File] }

Output:
201 Created – { property_id: UUID, ... }
400 Bad Request – Invalid/missing data

PATCH /api/properties/<property_id>

Input:
Auth JWT (host only),
Partial property fields for update

Output:
200 OK – Updated property
404 Not Found – If not owned by host

DELETE /api/properties/<property_id>

Input:
Auth JWT (host only)

Output:
204 No Content
404 Not Found – If not owned by host

GET /api/properties

Input:
Query params: location, price_min, price_max, guests, amenities

Output:
200 OK – [ ...propertyList ]

Validation
Name, location, price required; price positive decimal; at least one image.

Only authenticated hosts can add/edit/delete their listings.

Performance
Search/filter queries return ≤ 50 records/page, respond in < 2 seconds.

Image uploads constrain per property (max size/image count).

3. Booking System
Overview
Enables guests to create, view, and cancel bookings; enforces property availability and booking integrity.

Endpoints
POST /api/bookings

Input:
Auth JWT (guest),
{ "property_id": UUID, "start_date": ISODate, "end_date": ISODate, "guests": Integer }

Output:
201 Created – { booking_id, status: "pending", ... }
400 Bad Request – Availability conflict or invalid dates

GET /api/bookings/user

Input:
Auth JWT (user), pagination

Output:
200 OK – [ ...bookings ]

PATCH /api/bookings/<booking_id>/cancel

Input:
Auth JWT (guest or host)

Output:
200 OK – Updated status: "canceled"
403 Forbidden – Only guest, host, or admin can cancel

Validation
Dates must not overlap existing bookings; start_date < end_date; guests ≤ property capacity.

Only users who created the booking (or host of property) can cancel.

Performance
Booking actions < 2 seconds under typical load.

Atomic operations to avoid overbooking on concurrency. 