---
title: API Reference
tags: [murrnails, api]
up: "[[TechnicalDocument]]"
---

# API Reference

[[TechnicalDocument|← Core technical document]] · [[Architecture]] · [[Security and Access]]

Base path: `/api`. Responses are JSON unless the endpoint explicitly returns an iCalendar file or a static upload. The exact request DTOs live beside each controller under `backend/src/**/dto`.

## Authentication and identity

| Endpoint | Purpose | Access |
|---|---|---|
| `POST /auth/telegram` | Sign in/register from validated Telegram WebApp init data | Telegram init guard |
| `POST /auth/login`, `/auth/register` | Password sign-in/registration | public |
| `POST /auth/password` | Create/change password login | authenticated |
| `GET/PATCH /auth/me` | Current user and editable profile | authenticated |
| `GET /auth/me/activity`, `POST /auth/presence` | Activity/presence | authenticated |
| `GET /auth/users`, `/auth/users/:id`, `PATCH /auth/users/:id` | User/role administration | admin |
| `POST /auth/broadcast` | Telegram broadcast | admin |
| `GET /auth/online-users`, `/auth/new-users` | Admin presence/new users | admin |

## Appointments and booking

| Endpoint group | Purpose |
|---|---|
| `GET /appointments/masters`, `/services`, `/available-slots` | Booking discovery and availability |
| `POST /appointments/book` | Client booking creation |
| `GET /appointments/mine`, `POST /appointments/:id/cancel`, `POST /appointments/:id/reminder` | Client appointment management |
| `GET /appointments/calendar-summary`, `/day-detail`, `/discount-slots` | Client calendar and promotion views |
| `GET/PATCH /appointments/profile`, `GET /appointments/my-stats`, `/my-referral` | Client profile, metrics, referral |
| `GET /appointments/{home,my}-works`, work like endpoints, `/liked-works` | Feed and reactions |

Master CRM appointment endpoints are under `/crm/appointments`: list/create/get/update/delete plus `repeat-suggestion`, `repeat`, `day/message`, and `day/cancel` actions. Client creation, service catalog, availability, discount promotion, clients, analytics, expenses, and calendar feed token are likewise under `/crm`.

## CRM and content

| Prefix | Capability |
|---|---|
| `/crm/clients` | Client CRUD, profile/activity, name approval, student status, reminders/messages, client photos |
| `/crm/services` | Master service catalog CRUD |
| `/crm/availability` | Slot CRUD, alternative booking, promotion/unpromotion |
| `/crm/stats`, `/crm/analytics`, `/crm/expenses` | Reporting and monthly expense entries |
| `/crm/showcase` | Public feed and master management CRUD |
| `/crm/reviews` | Public/manage feeds, visibility, review configuration |
| `/chat` | Recipient list, conversation lifecycle, manual messages, templates |

## Other domain APIs

| Prefix | Capability |
|---|---|
| `/custom-time-requests` | Client creation; master/admin list, accept, decline |
| `/giveaways` | List, CRUD, participants/verification, entry, draw, winners |
| `/learning/courses` | Catalog, course CRUD, enrollment and enrollment-status management |
| `/suggestions` | Client proposal create/list; master/admin status change |
| `/patch-notes` | Release notes list/create and Telegram notification |
| `/admin/strings` | Runtime app-string defaults/overrides; audit log and bot instances |
| `/backup/run`, `/backup/restore` | Protected database backup/restore |
| `/calendar/feed/:token.ics` | Token-authorized iCalendar feed |
| `/summary/send` | Master summary delivery |

## API change checklist

1. Add/adjust a DTO with validation decorators.
2. Put ownership and role checks in service/guards, not only in Vue.
3. Update controller, service, entity/migration as needed.
4. Update the calling view/composable and this note if the public contract changed.
5. Test both an allowed request and a request from the wrong user/role.

> [!tip]
> Controllers are a concise endpoint index: `rg "@(Controller|Get|Post|Patch|Put|Delete)" backend/src --glob '*controller.ts'`.
