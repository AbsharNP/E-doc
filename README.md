## E-doc – Modern Medical SaaS

E-doc is a role-based medical appointment and records system built on Laravel + Vue 3 with a modern “medical SaaS” UI.

- **Patient**: Doctor discovery, booking wizard, slot-based session booking, upcoming visits, prescriptions.
- **Doctor**: Daily schedule management, session creation, appointment list, prescription entry.
- **Admin**: Department and doctor management, high-level analytics.

### Getting started

1. **Install PHP & Node dependencies**

```bash
composer install
npm install
```

2. **Environment & database**

- Copy `.env.example` to `.env` and configure your database connection.
- Run migrations:

```bash
php artisan migrate
```

3. **Seed default admin user**

Run the seeder to create an initial admin account:

```bash
php artisan db:seed --class=AdminUserSeeder
```

Default admin credentials (from `AdminUserSeeder`):

- Email: `admin@edoc.com`
- Password: `admin123`

4. **Run dev servers**

```bash
php artisan serve
npm run dev
```

Then open the app in your browser (typically `http://127.0.0.1:8000`).

### Frontend overview

- Vue 3 SPA under `resources/js` with role-specific dashboards and views.
- Router guards based on `userType` (`admin`, `doctor`, `patient`).
- Shared modern theme tokens in `resources/css/app.css` (cards, pills, teal/blue palette).

### Notes

- Doctor sessions are created from the doctor dashboard; patients then book specific time slots within those sessions.
- Prescriptions are created per appointment and stored server-side via the `/submit-prescription` endpoint.

