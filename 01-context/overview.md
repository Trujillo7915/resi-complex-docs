# System Overview

---

## What is Residential-complex?

Residential-complex (resi-complex) is a web-based information system that lets the administration of a mixed residential complex — one with both housing units and commercial establishments — manage everything from a single place: units, residents and shop owners/tenants, maintenance requests, differentiated administration fees, visitor and vehicle access, correspondence, segmented announcements, and Board-approved extraordinary expenses. It replaces manual, fragmented tools (notebooks, spreadsheets, WhatsApp) with one traceable digital record.

## Problem it solves

**Before the system:** Small and medium residential complexes manage these processes manually or across disconnected tools (notebooks, Excel, WhatsApp, physical logbooks). This causes duplicated or lost visitor records, difficulty tracking administration fees — especially with different rates for residential and commercial units — no traceability on maintenance requests, and no formal record of Board approvals for extraordinary expenses. The result is a lack of transparency toward residents and commercial tenants.

**With the system:** Units, fees, maintenance, access control, correspondence, and expense approvals are centralized and traceable. Residents and commercial owners/tenants get transparency into their own fees, requests, and approved expenses; the Board gets a permanent, auditable history of every approval or rejection; and the Administrator gets real-time information (pending fees, request status, approved expenses) to support decisions.

## Main users

| Role | Description | What they do in the system |
|------|-------------|--------------------------|
| Administrator | Staff member who runs the day-to-day administration of the complex | Manages units and people, generates fees, assigns maintenance requests, publishes announcements, creates extraordinary expense proposals |
| Board of Directors | Elected body responsible for financial oversight of the complex | Approves or rejects expense proposals, reviews financial reports |
| Person (Resident / Commercial Owner or Tenant) | Occupant of a residential unit, or owner/tenant of a commercial unit | Creates maintenance requests, checks their fee status, views announcements segmented to their unit type, checks pending correspondence |
| Maintenance Staff | Staff member who resolves maintenance requests | Views assigned requests and updates their status (received → in progress → resolved) |
| Security Guard | Staff member responsible for the entrance/lobby (portería) | Logs visitor and vehicle entry/exit (personal visitors or commercial clients), logs incoming correspondence |

## Technology stack

| Layer | Technology | Justification |
|-------|-----------|---------------|
| Frontend | Pending decision: Thymeleaf (server-rendered) or plain HTML/CSS/JS consuming a REST API | Team decision still open, depending on available development time and learning objectives for the cohort |
| Backend | Java + Spring Boot, layered architecture (Controller–Service–Repository) per microservice | Language and framework required by the SENA ADSO formative program; layered/microservices structure enforces separation of concerns and independent deployability |
| Database | MySQL, relational, in principle one database per microservice | Matches the relational nature of the domain (units, fees, approvals) and is the database confirmed for the project |
| Message broker | Pending definition | Not yet selected; needed once inter-service communication patterns (e.g. fee generation, notifications) are finalized with the instructor |
| Infrastructure | Pending definition | Not yet selected; to be defined together with the final microservices catalog (see `05-architecture/`) |

## Current status

- **Phase:** In development (formative project — requirements, technology watch, and governance completed; implementation not started)
- **Current version:** v0.1.0
- **Last release:** Not yet released
- **Next milestone:** Confirm final microservices catalog with the instructor, then define the data model per service (`06-data/`)

## Project contacts

| Role | Name | Contact |
|------|------|---------|
| Tech Lead | Pending — to be assigned by the team | |
| Product Owner | Pending — to be assigned by the team | |
| DevOps | Pending — to be assigned by the team | |
