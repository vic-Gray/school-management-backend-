Skul Africa v2 — Enterprise School Management Platform (Offline-First, Modular Architecture)

Skul Africa v2 is a complete architectural redesign of the v1 system. It is engineered to meet real-world school demands, provide stable offline-first operation, and support enterprise-level deployment, modular scaling, and long-term maintainability.

Version 2 is built with a multi-layered software architecture, optimized data structures, and a hybrid sync model suitable for Africa’s connectivity challenges.

1. System Architecture Overview
Backend Architecture (NestJS — Layered Architecture)
/src
  /modules
    /auth
    /schools
    /classes
    /students
    /teachers
    /subjects
    /attendance
  /core
    /database
    /guards
    /interceptors
    /middleware
    /decorators
  /shared
    /dto
    /interfaces
    /helpers

Architecture Layers

Controller Layer → HTTP layer only

Service Layer → Business logic

Repository Layer → Entity persistence

Guards & Interceptors → Security, validation

Events / Cache → Redis-backed caching + queues

2. Data Structure & Relationships
School-Centric Multi-Tenant Model
School
 ├── Classes
 │      └── Students
 ├── Teachers
 ├── Subjects
 └── Academic Sessions

Entities (Simplified)
School

id

name

proprietor details

address

settings (theme, session, etc.)

ClassRoom

id

schoolId (FK)

name ("Basic 1")

formTeacherId (FK)

capacity

studentCount (cached)

Student

id

schoolId

classId

firstName, lastName

gender

parent info

admissionNumber (generated)

offline UUID for local sync

Teacher

id

schoolId

fullName

email

phone

subjects (relation)

Subject

id

schoolId

subject name

teacherId

3. Offline-First Architecture
Hybrid Offline Sync Model

Skul Africa v2 uses a Local-First Write technique:

1. Local Cache (Client PWA)

IndexedDB for storage

Service Workers for caching routes

Offline queue for unsent write requests

2. Request Queue

All user actions when offline are stored:

{
  id: uuid,
  type: "create_student",
  payload: {...},
  timestamp: 1731000000,
  status: "pending"
}

3. Smart Sync Engine

When network returns:

Queue is replayed in FIFO

Conflicts resolved using versioning

Server returns authoritative state

4. Delta Sync

Only changed data is synced:

New students

Updated profile

Newly created classes

4. Deployment Strategy (Enterprise)

Skul Africa v2 is designed for scalable deployment under multiple environments.

A. Deployment Environments
1. Development (Local)

Docker Compose

PostgreSQL + Redis local containers

Hot reload

2. Staging

GitHub Actions → CI/CD

Automated migrations

Pre-release testing

Pre-cache regeneration

3. Production

Infrastructure Options:

AWS ECS / EC2

DigitalOcean Kubernetes

Render / Railway

Vercel (Frontend)

5. Scaling Strategy
Horizontal Scalability

Stateless servers

Redis-backed session + cache

Load balancer (NGINX or AWS ALB)

Database Scaling

Read replicas

Partitioning by school ID

Query optimizations for large schools

Caching Strategy

Redis for:

Frequently accessed data

Student count per class

Form teacher assignments

Token blacklists

6. Security Model
Authentication

JWT (Access + Refresh)

Password hashing (bcrypt)

Authorization

Multi-tenant guard

Role-based access:

Proprietor

Teacher

Admin

Data Isolation

Every query is scoped by:

where: { schoolId: currentSchool.id }

Request Validation

DTO validation

Sanitizers

Rate limiting

7. Code Standards
Naming

PascalCase → Classes

camelCase → Functions and fields

DTO Standards

Each module uses:

create.dto

update.dto

filter.dto

Error Handling

All errors follow a unified shape:

{
  statusCode: 400,
  message: "Invalid class ID",
  error: "Bad Request"
}
