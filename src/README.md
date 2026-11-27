# Mergington High School Activities API

A super simple FastAPI application that allows students to view and sign up for extracurricular activities.

## Features

- View all available extracurricular activities
- View registered students without logging in
- Allow teachers to sign up or unregister students after logging in

## Getting Started

1. Install the dependencies:

   ```
   pip install fastapi uvicorn
   ```

2. Configure teacher authentication. Generate a salted PBKDF2 password hash
   using Python; enter the password only at the prompt:

   ```
   python -c 'import getpass, hashlib, secrets; salt=secrets.token_bytes(16); password=getpass.getpass("Teacher password: "); digest=hashlib.pbkdf2_hmac("sha256", password.encode(), salt, 310000); print(salt.hex() + ":" + digest.hex())'
   ```

   Configure each teacher as a username mapped to their generated
   `salt:hash`, then set the JSON object and a session-signing secret in the
   environment before starting the server:

   ```
   export TEACHER_CREDENTIALS='{"teacher":"paste-salt:paste-hash"}'
   export SESSION_SECRET='paste-at-least-32-random-bytes-here'
   ```

   Generate a session secret with
   `python -c 'import secrets; print(secrets.token_urlsafe(32))'` and keep it
   private and stable across server restarts. Do not commit credentials or
   secrets. Set `COOKIE_SECURE=true` when the app is served over HTTPS; leave
   it unset for local HTTP development.

3. Run the application:

   ```
   uvicorn src.app:app --reload
   ```

4. Open your browser and go to:
   - API documentation: http://localhost:8000/docs
   - Alternative documentation: http://localhost:8000/redoc

## API Endpoints

| Method | Endpoint                                                          | Description                                                         |
| ------ | ----------------------------------------------------------------- | ------------------------------------------------------------------- |
| GET    | `/activities`                                                     | Get all activities and registered students                          |
| GET    | `/auth/status`                                                    | Check whether the current browser session is authenticated           |
| POST   | `/auth/login`                                                     | Log in with the configured teacher username and password             |
| POST   | `/auth/logout`                                                    | End the current teacher session                                     |
| POST   | `/activities/{activity_name}/signup?email=student@mergington.edu` | Teacher-only: sign up a student                                      |
| DELETE | `/activities/{activity_name}/unregister?email=student@mergington.edu` | Teacher-only: unregister a student                                 |

Teacher passwords are configured as salted PBKDF2 hashes rather than checked-in
credentials. Add one username/hash pair per teacher to `TEACHER_CREDENTIALS`.
Successful logins use an eight-hour, signed, HttpOnly, SameSite=Strict cookie.
Activity data remains in memory and resets when the server restarts.

## Data Model

The application uses a simple data model with meaningful identifiers:

1. **Activities** - Uses activity name as identifier:

   - Description
   - Schedule
   - Maximum number of participants allowed
   - List of student emails who are signed up

2. **Students** - Uses email as identifier:
   - Name
   - Grade level

All data is stored in memory, which means data will be reset when the server restarts.
