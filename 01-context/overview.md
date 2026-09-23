# System Overview

## What is Save Your Water?

Save Your Water is a web and mobile platform that measures household or business water consumption in real time through an IoT device (ESP32 + flow sensor) installed at the property's main water inlet. It allows users to remotely shut off the water supply via an electrovalve and compare sensor readings against utility bills to detect overcharges.

## Problem it solves

**Before the system:** Households and business owners only discover how much water they consumed when the monthly utility bill arrives. By then, leaks may have gone undetected for weeks, there is no way to verify if the billed amount is accurate, and corrective action is impossible.

**With the system:** Users gain immediate visibility into their water consumption, receive real-time alerts for suspected leaks, can shut off the water supply remotely, and compare measured consumption against utility bills — all from their phone or browser.

## Main users

| Role                                  | Description                                                                                                                                                          | What they do in the system                                                                                                                                |
| ------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Usuario base (Base User)**          | Default role upon registration. Can view places, create a new one, or accept an invitation as collaborator. Cannot access consumption features until associated with a place. | Can view places, create new ones, or accept invitations as collaborator. Cannot access consumption features until associated with a place.                 |
| **Dueño (Owner)**                    | Person who pays the water bill (head of household or business owner). Has full control over a place.                                                                  | Full access: view consumption, invite members, set goals, open/close valve, register utility bills, configure tariffs and alerts.                         |
| **Invitado (Guest)**                  | Anonymous user who accesses the system via a shared link provided by the Owner. Does not require registration or login.                                               | Access via shared link without registration. View-only: consumption, charts, history. Cannot see control actions, bills, or configuration.                |
| **Colaborador (Collaborator)**        | Registered user invited by the Owner with partial permissions defined by the Owner.                                                                                   | Partial access defined by the Owner. Can perform actions according to assigned permissions.                                                                |
| **Administrador (Admin)**             | Platform operator responsible for user management, device validation, platform metrics, and water-saving recommendations.                                             | View registered users list (no sensitive data), block/unblock users, view device list, view platform metrics, manage water-saving recommendations.        |

## Technology stack

| Layer                    | Technology                                              | Justification                                                                              |
| ------------------------ | ------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| Mobile + Web             | React Native (with web extension)                       | Single codebase for Android, iOS, and web browser                                          |
| Backend — IAM           | Spring Boot (Java)                                      | Mature ecosystem for authentication, authorization, and security                           |
| Backend — Other modules | .NET (under evaluation)                                 | Team expertise; strong typing and performance for business services                        |
| Database                 | SQL Server (single shared instance, schema-per-service) | Team familiarity; robust transactional support. One instance, logical separation by schema |
| IoT controller           | ESP32                                                   | Low-cost Wi-Fi-enabled microcontroller; handles sensor reading and valve control           |
| IoT protocol             | MQTT                                                    | Lightweight publish/subscribe protocol optimized for constrained devices                   |
| MQTT Broker              | Mosquitto (under evaluation)                            | Lightweight broker; receives telemetry from ESP32 devices and routes it to backend services. Final selection pending |
| Infrastructure           | Docker (local) / VPS                                    | Docker for development; VPS for initial deployment. Cloud provider TBD                     |

## Hardware context

| Component    | Model / Spec          | Purpose                                                        |
| ------------ | --------------------- | -------------------------------------------------------------- |
| Flow sensor  | YF-S201               | Measures water flow rate at the main inlet                     |
| Electrovalve | Generic normally-open | Allows remote water shutoff                                    |
| Controller   | ESP32                 | Reads sensor, controls valve, communicates via MQTT over Wi-Fi |

> **Limitation:** A single sensor at the main inlet measures total consumption for the entire property. It cannot break down usage by individual fixtures (shower, sink, toilet). Saving tips are therefore generic, not personalized per fixture.

## Current status

- **Phase:** Early development
- **Current version:** v0.1.0
- **Backlog version:** 2.0 (dated 2026-09-23)
- **Next milestone:** MVP — user account + place registration + device linking + real-time consumption chart

## Project contacts

| Role      | Name                      | Contact |
| --------- | ------------------------- | ------- |
| Developer | Juan Esteban Ome Esquivel | —      |
| Developer | Juan Manuel Gómez        | —      |
| Developer | Diego Alejandro Alvarado  | —      |
