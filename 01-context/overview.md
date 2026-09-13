# System Overview

## What is Save Your Water?

Save Your Water is a web and mobile platform that measures household or business water consumption in real time through an IoT device (ESP32 + flow sensor) installed at the property's main water inlet. It allows users to remotely shut off the water supply via an electrovalve and compare sensor readings against utility bills to detect overcharges.

## Problem it solves

**Before the system:** Households and business owners only discover how much water they consumed when the monthly utility bill arrives. By then, leaks may have gone undetected for weeks, there is no way to verify if the billed amount is accurate, and corrective action is impossible.

**With the system:** Users gain immediate visibility into their water consumption, receive real-time alerts for suspected leaks, can shut off the water supply remotely, and compare measured consumption against utility bills — all from their phone or browser.

## Main users

| Role                            | Description                                                                    | What they do in the system                                                                                                                             |
| ------------------------------- | ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Usuario (User)**        | Default user role upon accessing the application, well before entering a home. | Will be able to see if you have a home set up; if not, you can create one.                                                                             |
| **Dueño (Owner)**        | Person who pays the water bill (head of household or business owner)           | Full access: view consumption, set goals, open/close the valve, register utility bills, configure alerts, invite members                                |
| **Miembro (Member)**      | Family member, roommate, or administrator invited by the owner                 | It relies on permissions set by the owner of the location: It depends on whether you only want to read data or need to perform actions within the app. |
| **Administrador (Admin)** | Platform operator                                                              | View registered users (no sensitive data), block/unblock users, validate device failures, monitor platform health, create generic water-saving recommendations |

## Technology stack

| Layer                    | Technology                                              | Justification                                                                              |
| ------------------------ | ------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| Mobile + Web             | React Native (with web extension)                       | Single codebase for Android, iOS, and web browser                                          |
| Backend — IAM           | Spring Boot (Java)                                      | Mature ecosystem for authentication, authorization, and security                           |
| Backend — Other modules | .NET (under evaluation)                                 | Team expertise; strong typing and performance for business services                        |
| Database                 | SQL Server (single shared instance, schema-per-service) | Team familiarity; robust transactional support. One instance, logical separation by schema |
| IoT controller           | ESP32                                                   | Low-cost Wi-Fi-enabled microcontroller; handles sensor reading and valve control           |
| IoT protocol             | MQTT                                                    | Lightweight publish/subscribe protocol optimized for constrained devices                   |
| MQTT Broker              | Eclipse Mosquitto 2                                      | Lightweight broker; receives telemetry from ESP32 devices and routes it to backend services |
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
- **Backlog version:** 1.0 (dated 2026-09-02)
- **Next milestone:** MVP — user account + place registration + device linking + real-time consumption chart

## Project contacts

| Role      | Name                      | Contact |
| --------- | ------------------------- | ------- |
| Developer | Juan Esteban Ome Esquivel | —      |
| Developer | Juan Manuel Gómez        | —      |
| Developer | Diego Alejandro Alvarado  | —      |
