# Hospital Appointment API

A Spring Boot + Maven REST API that lets patients book, cancel, and reschedule
hospital appointments. Built with a layered architecture: **Model → Repository
→ Service → Controller**, backed by MySQL.

## Project structure

```
src/main/java/com/hospital/appointment/
├── model/            Entities: Patient, Doctor, Appointment, AppointmentStatus
├── repository/        Spring Data JPA repositories
├── service/            Service interfaces
│   └── impl/           Service implementations (business logic)
├── controller/         REST controllers
├── dto/                Request/response DTOs
└── exception/          Custom exceptions + global exception handler
```

## Prerequisites

- Java 17+
- Maven 3.8+ (or use the included wrapper if you generate one)
- MySQL 8+ running locally
- VS Code with the "Extension Pack for Java" and "Spring Boot Extension Pack"
  (recommended, not required)

## 1. Set up MySQL

You don't need to manually create the database — `createDatabaseIfNotExist=true`
in the config handles it. You just need a MySQL server running and a user with
access.

## 2. Configure the connection

Edit `src/main/resources/application.properties`:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/hospital_appointments?createDatabaseIfNotExist=true&useSSL=false&serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=your_mysql_password
```

Replace the username/password with your own MySQL credentials.

## 3. Run the app

From the project root:

```bash
mvn spring-boot:run
```

Or in VS Code: open the folder, let it import as a Maven project, then run/debug
`HospitalAppointmentApiApplication.java` directly (right-click → "Run Java").

The API starts on `http://localhost:8080`.

Hibernate will auto-create the `patients`, `doctors`, and `appointments` tables
on first run (`spring.jpa.hibernate.ddl-auto=update`).

## 4. Try it out

### Create a doctor
```bash
curl -X POST http://localhost:8080/api/doctors \
  -H "Content-Type: application/json" \
  -d '{"fullName": "Dr. Jane Smith", "specialization": "Cardiology"}'
```

### Create a patient
```bash
curl -X POST http://localhost:8080/api/patients \
  -H "Content-Type: application/json" \
  -d '{"fullName": "John Doe", "email": "john@example.com", "phoneNumber": "555-1234"}'
```

### Book an appointment
```bash
curl -X POST http://localhost:8080/api/appointments \
  -H "Content-Type: application/json" \
  -d '{
        "patientId": 1,
        "doctorId": 1,
        "appointmentDateTime": "2026-10-15T10:00:00",
        "reasonForVisit": "Annual checkup"
      }'
```

### Cancel an appointment
```bash
curl -X PATCH http://localhost:8080/api/appointments/1/cancel
```

### Reschedule an appointment
```bash
curl -X PATCH http://localhost:8080/api/appointments/1/reschedule \
  -H "Content-Type: application/json" \
  -d '{"newAppointmentDateTime": "2026-10-16T14:30:00"}'
```

### View appointments
```bash
curl http://localhost:8080/api/appointments
curl http://localhost:8080/api/appointments/patient/1
curl http://localhost:8080/api/appointments/doctor/1
```

## API summary

| Method | Endpoint                                | Description                     |
|--------|------------------------------------------|----------------------------------|
| POST   | `/api/patients`                          | Create a patient                 |
| GET    | `/api/patients`                          | List patients                    |
| GET    | `/api/patients/{id}`                     | Get a patient                    |
| DELETE | `/api/patients/{id}`                     | Delete a patient                 |
| POST   | `/api/doctors`                           | Create a doctor                  |
| GET    | `/api/doctors`                           | List doctors                     |
| GET    | `/api/doctors/{id}`                      | Get a doctor                     |
| DELETE | `/api/doctors/{id}`                      | Delete a doctor                  |
| POST   | `/api/appointments`                      | Book an appointment              |
| GET    | `/api/appointments`                      | List all appointments            |
| GET    | `/api/appointments/{id}`                 | Get one appointment              |
| PATCH  | `/api/appointments/{id}/cancel`          | Cancel an appointment            |
| PATCH  | `/api/appointments/{id}/reschedule`      | Reschedule an appointment        |
| GET    | `/api/appointments/patient/{patientId}`  | List a patient's appointments    |
| GET    | `/api/appointments/doctor/{doctorId}`    | List a doctor's appointments     |

## Notes / things worth knowing

- Booking and rescheduling both check that the doctor isn't already booked
  at that exact date/time (excluding cancelled appointments), and reject with
  `409 Conflict` if so.
- `appointmentDateTime` must be in the future (validated via `@Future`).
- Validation errors return `400` with a `fieldErrors` map.
- Not-found resources return `404`; scheduling conflicts return `409`.
- This is a starting point, not a production-hardened service — there's no
  authentication/authorization layer, so you'll want to add Spring Security
  (or similar) before exposing this beyond local development.
