# Global Class Offering Booking System

A REST API for creating and booking live online classes across time zones. Teachers can create courses, offerings, and class sessions. Parents can browse offerings in their local time zone and book them.

## Features

- Teacher, parent, and course creation
- Class offering and session management
- Timezone-aware scheduling: session times are entered in the offering's timezone and stored in UTC
- Offering and booking retrieval
- Booking conflict detection for overlapping sessions
- Transactional booking logic with pessimistic locking to protect against concurrent booking conflicts
- Request validation and centralized error handling
- Swagger UI and a Postman collection

## Technology

- Java 21
- Spring Boot 3.4
- Spring Web
- Spring Data JPA / Hibernate
- PostgreSQL 16
- Maven
- Docker Compose

## Prerequisites

Install the following:

- JDK 21
- Maven 3.9 or later
- Docker Desktop (or another Docker Compose-compatible runtime)

Check that the tools are available:

```shell
java -version
mvn -version
docker compose version
```

## Run locally

1. Clone the repository and enter its directory:

   ```shell
   git clone https://github.com/Balaji2004-bmn/Global-Class-Booking-System.git
   cd Global-Class-Booking-System
   ```

2. Start PostgreSQL:

   ```shell
   docker compose up -d
   docker compose ps
   ```

   Docker Compose starts PostgreSQL on port `5432` with these local development settings:

   | Setting | Value |
   | --- | --- |
   | Database | `class_booking` |
   | Username | `booking_app` |
   | Password | `booking_app` |
   | Host port | `5432` |

3. Run the application from the project directory:

   ```shell
   mvn spring-boot:run
   ```

   The API is available at `http://localhost:8080`.

   To stop the database when you are finished:

   ```shell
   docker compose down
   ```

   To also remove the local database data volume, run `docker compose down -v`. This permanently deletes the local PostgreSQL data.

## Build and test

Run the test suite:

```shell
mvn test
```

Build an executable JAR:

```shell
mvn clean package
```

Run the packaged application:

```shell
java -jar target/global-class-booking-system-0.0.1-SNAPSHOT.jar
```

The concurrency integration test uses Testcontainers and may require Docker to be running.

## API documentation

When the application is running, open Swagger UI at:

`http://localhost:8080/swagger-ui/index.html`

You can also import `postman/Global-Class-Offering-Booking-System.postman_collection.json` into Postman. The collection uses `http://localhost:8080` as its default base URL.

## API endpoints

All endpoints use JSON request/response bodies where applicable.

| Method | Endpoint | Description |
| --- | --- | --- |
| `POST` | `/teachers` | Create a teacher |
| `GET` | `/teachers/{teacherId}/offerings` | List a teacher's offerings |
| `POST` | `/parents` | Create a parent |
| `GET` | `/parents/{parentId}/bookings` | List a parent's bookings |
| `POST` | `/courses` | Create a course |
| `POST` | `/offerings` | Create an offering |
| `POST` | `/offerings/{id}/sessions` | Add a session to an offering |
| `GET` | `/offerings?timezone={IANA timezone}` | List offerings with session times in the requested timezone |
| `POST` | `/bookings` | Book an offering |

### Example workflow

Create a teacher:

```http
POST /teachers
Content-Type: application/json
```

```json
{
  "name": "Avery Teacher",
  "timezone": "America/New_York"
}
```

Create a parent:

```http
POST /parents
Content-Type: application/json
```

```json
{
  "name": "Priya Parent",
  "timezone": "Asia/Kolkata"
}
```

Create a course:

```http
POST /courses
Content-Type: application/json
```

```json
{
  "title": "Physics Foundations",
  "description": "Introductory physics for middle school learners"
}
```

Create an offering using the IDs returned by the teacher and course requests:

```http
POST /offerings
Content-Type: application/json
```

```json
{
  "courseId": 1,
  "teacherId": 1,
  "title": "Physics June Cohort",
  "timezone": "America/New_York"
}
```

Add a session. The local date and time in this request are interpreted in the offering's timezone:

```http
POST /offerings/1/sessions
Content-Type: application/json
```

```json
{
  "startTime": "2026-06-01T10:00:00",
  "endTime": "2026-06-01T11:00:00"
}
```

Browse offerings with session times converted to the desired timezone:

```http
GET /offerings?timezone=Asia/Kolkata
```

Book an offering using the parent and offering IDs:

```http
POST /bookings
Content-Type: application/json
```

```json
{
  "parentId": 1,
  "offeringId": 1
}
```

## Timezone and booking behavior

- Timezones must be valid IANA timezone IDs, such as `America/New_York` or `Asia/Kolkata`.
- Session `startTime` and `endTime` are local date-times interpreted using the offering's timezone.
- Session instants are stored in UTC and converted for offering queries.
- A booking is rejected if the parent already has a booking for an overlapping class session.
- Booking operations are transactional and use database locking to handle concurrent requests safely.

## Database

The local PostgreSQL service and its persistent volume are defined in `docker-compose.yml`. The application initializes the tables from `src/main/resources/schema.sql`; Hibernate validates the schema at startup.

The database credentials in the Compose file are intended only for local development. Use secure, externally supplied credentials for any deployed environment.

## Project layout

```text
src/main/java/com/assessment/booking/
  config/       Jackson and validation configuration
  controller/   REST API endpoints
  dto/          Request and response types
  entity/       JPA entities
  exception/    Application exceptions and API error handling
  mapper/       Entity-to-response mapping
  repository/   Spring Data repositories
  service/      Business logic
src/main/resources/
  application.yml
  schema.sql
src/test/
  java/         Unit and integration tests
  resources/    Test configuration
postman/        Postman collection
```