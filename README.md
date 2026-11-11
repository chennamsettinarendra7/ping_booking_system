# Department Seminar Hall & Lab Slot Booking System

A full-stack reservation portal for college faculty members, department heads, and class representatives to reserve seminar halls, computing laboratories, and smart AV projector facilities with real-time conflict prevention.

---

## 1. Technology Stack
- **Frontend**: React.js (Vite, TypeScript, Tailwind CSS v4, Lucide Icons)
- **Backend**: Node.js & Express.js REST API
- **Database**: MongoDB with Mongoose ODM
- **Conflict Prevention**: Pre-save verification ensuring no overlapping bookings for the same hall/lab, date, and time slot.

---

## 2. Project Directory Structure
```
├── client/                      # Frontend React Application
│   ├── components/              # UI components (Navbar, Sidebar, StatCard, Toast, Modals)
│   ├── pages/                   # Pages (Dashboard, NewBooking, ViewBookings, ScheduleMatrix, Facilities)
│   ├── services/api.ts          # REST API communication service layer
│   └── types/index.ts           # TypeScript interfaces (Booking, Resource, Stats)
├── server/                      # Backend Node.js & Express Application
│   ├── config/db.ts             # MongoDB Mongoose connection handler
│   ├── controllers/             # Booking controller (CRUD, conflict checks, stats)
│   ├── models/Booking.ts        # Mongoose Schema & Model with conflict index
│   ├── routes/bookingRoutes.ts  # Express REST API routes
│   ├── app.ts                   # Express server configuration & middlewares
│   └── server.ts                # Standalone backend server entry point
├── src/                         # React Root Application (App.tsx, main.tsx, index.css)
├── .env.example                 # Environment variables template
├── server.ts                    # Unified full-stack server running Express + Vite
└── package.json                 # Dependencies and build scripts
```

---

## 3. Quick Start in VS Code

### Step 1: Clone or Open in VS Code
Open the project directory in Visual Studio Code.

### Step 2: Install Dependencies
Open the VS Code terminal (`Ctrl + ~` or `Cmd + ~`) and run:
```bash
npm install
```

### Step 3: Configure Environment Variables
Copy `.env.example` to `.env`:
```bash
cp .env.example .env
```
Open `.env` and set your `MONGODB_URI` (see Section 4 below).

### Step 4: Run the Application
Start the full-stack development server:
```bash
npm run dev
```
Open your browser at **`http://localhost:3000`**. The application runs with both React and Express REST APIs seamlessly unified on port 3000.

*(Optional)* To run only the backend server on a dedicated port:
```bash
npx tsx server/server.ts
```

---

## 4. Connecting MongoDB Atlas (Free Cloud Database)

1. **Sign Up / Log In**: Visit [mongodb.com/atlas](https://www.mongodb.com/atlas) and log into your account.
2. **Create a Free Cluster**: Click **Create Deployment**, select the **M0 Free Shared Tier**, choose your nearest region (e.g., AWS / Asia or US), and click **Create**.
3. **Configure Database Credentials**:
   - Go to **Security > Database Access**.
   - Click **Add New Database User**.
   - Create a user (e.g., username `college_admin` and set an auto-generated password).
4. **Configure IP Access**:
   - Go to **Security > Network Access**.
   - Click **Add IP Address**.
   - Choose **Allow Access from Anywhere (`0.0.0.0/0`)** and click Confirm.
5. **Copy Connection String**:
   - Go to **Database > Deployment > Connect**.
   - Select **Drivers (Node.js)**.
   - Copy the URI string:
     ```
     mongodb+srv://<username>:<password>@cluster0.mongodb.net/seminar_booking?retryWrites=true&w=majority
     ```
6. **Set in `.env`**:
   Replace `<username>` and `<password>` with your database user credentials in `.env`:
   ```env
   MONGODB_URI="mongodb+srv://college_admin:YourPassword123@cluster0.mongodb.net/seminar_booking?retryWrites=true&w=majority"
   ```
7. Restart your development server (`npm run dev`). The system will automatically connect to MongoDB Atlas and persist all bookings permanently.

---

## 5. Mongoose Booking Schema

```typescript
{
  facultyName: String,     // Required, trim, minlength: 2
  hallName: String,        // Required, Seminar Hall or Laboratory
  bookingDate: Date,       // Required, reservation calendar date
  timeSlot: String,        // Required, e.g. "10:00 AM - 11:00 AM"
  purpose: String,         // Required, minlength: 3
  department: String,      // e.g. "Computer Science & Engineering"
  contactNumber: String,   // Optional contact info
  equipmentNeeded: [String], // e.g. ["AV Projector & Screen", "Mic Array"]
  status: String,          // "Confirmed" | "Cancelled"
  createdAt: Date          // Default: Date.now
}
```

### Conflict Prevention Rule
Before saving any reservation, the backend checks for existing active bookings:
```typescript
const conflict = await Booking.findOne({
  hallName,
  bookingDate: { $gte: startOfDay, $lte: endOfDay },
  timeSlot,
  status: { $ne: 'Cancelled' }
});

if (conflict) {
  return res.status(409).json({
    success: false,
    message: "This time slot is already reserved. Please select another slot."
  });
}
```

---

## 6. REST API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/bookings` | Fetch all bookings with search, date, and hall filters |
| `POST` | `/api/bookings` | Create new reservation with validation & conflict prevention |
| `DELETE` | `/api/bookings/:id` | Cancel reservation and release slot |
| `GET` | `/api/bookings/stats` | Retrieve Total, Available, Today's, and Upcoming stats |
| `GET` | `/api/bookings/resources` | Fetch all department seminar halls, labs & standard slots |
| `GET` | `/api/bookings/availability` | Query live slot availability for a hall on a given date |
| `GET` | `/api/system/status` | Verify MongoDB connection and system status |
| `POST` | `/api/system/seed` | Seed initial demo data |

---

## 7. Key Features
- **Dashboard**: 4 KPI cards (Total Bookings, Available Halls/Labs, Today's Bookings, Upcoming Reservations), today's timeline, and recent booking activity.
- **New Booking Form**: Faculty name, Hall/Lab selector, Date picker, dynamic slot matrix with live free/occupied status, and equipment checklist.
- **Conflict Prevention**: Rejects duplicate bookings with the standard response: *"This time slot is already reserved. Please select another slot."*
- **Reservations Table**: Real-time search, date filter, hall filter, CSV export, and Cancel Booking with confirmation dialog.
- **Slot Matrix**: 2D grid of all 7 facilities vs 7 time slots for any date.
- **Facilities Directory**: Complete seating capacities, room numbers, and AV projector specifications.
