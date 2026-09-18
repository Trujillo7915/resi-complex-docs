# Functional Requirements → Services Mapping

Mapping from functional requirements (04-requirements/user-stories.md) 
to implementing services (09-microservices/services/).

## FR → BC → Service Matrix

| FR | Associated US | Description | Bounded Context | Service |
|---|---|---|---|---|
| FR01 | US-01 | Register new residential unit | Units Management | 03-units-service |
| FR02 | US-02 | Register new commercial unit | Units Management | 03-units-service |
| FR03 | US-03 | Assign resident to unit | People Management | 04-people-service |
| FR04 | US-04 | Register board of directors | People Management | 04-people-service |
| FR05 | US-05 | Create maintenance request | Maintenance | 05-maintenance-service |
| FR06 | US-06 | Assign maintenance staff | Maintenance | 05-maintenance-service |
| FR07 | US-07 | Record extraordinary expenses | Billing | 06-billing-service |
| FR08 | US-08 | Calculate monthly fee | Billing | 06-billing-service |
| FR09 | US-09 | Generate payment receipt | Billing | 06-billing-service |
| FR10 | US-10 | Create expense approval request | Financial Approval | 07-finance-service |
| FR11 | US-11 | Vote on expense approval | Financial Approval | 07-finance-service |
| FR12 | US-12 | Send announcement to residents | Communications | 08-communications-service |
| FR13 | US-13 | Register visitor | Access Control | 09-access-service |
| FR14 | US-14 | Authorize vehicle entry | Access Control | 09-access-service |
| FR15 | US-15 | Register mail/correspondence | Access Control | 09-access-service |
| FR16 | US-16 | Generate collected fees report | Reports | 10-reports-service |
| FR17 | US-17 | Generate approved expenses report | Reports | 10-reports-service |
| FR18 | US-18 | Export access logs | Reports | 10-reports-service |
| FR19 | US-19 | Change audit logs | Reports | 10-reports-service |
| FR20 | US-20 | Authentication and authorization | IAM | 02-auth-service |

## Implementation Notes

1. **Inter-service dependencies:**
   - `03-units-service` must be ready before `04-people-service` (related)
   - `06-billing-service` must be ready before `07-finance-service` (approval)
   - `02-auth-service` is a blocker for EVERYTHING

2. **Recommended development order:**
   - Sprint 1: 02-auth-service (blocker)
   - Sprint 2: 03-units-service, 04-people-service
   - Sprint 3: 06-billing-service, 07-finance-service
   - Sprint 4: 05-maintenance-service, 08-communications-service, 09-access-service
   - Sprint 5+: 10-reports-service (depends on all others)

---

**Created:** [2026-09-18]