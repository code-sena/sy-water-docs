# Delivery Roadmap

> This document maps the 4 deliveries (Entregas) to epics, features, and user stories.
> No fixed dates — deliveries are ordered by priority and executed sequentially.

---

## Delivery overview

```
Entrega 1 (MVP)          Entrega 2 (Smart)         Entrega 3 (Social)       Entrega 4 (Admin)
┌─────────────────┐      ┌──────────────────┐      ┌──────────────────┐      ┌─────────────────┐
│ E1: Auth         │      │ E5: Valve Ctrl    │      │ E9: Guest         │      │ E13: Reports     │
│ E2: Place Mgmt   │      │ E6: Goals         │      │ E10: Collaborator  │      │ E14: Admin Panel │
│ E3: Device Link  │      │ E7: Alerts        │      │ E11: Bills         │      │ E15: Tariffs CO  │
│ E4: Dashboard    │      │ E8: Auto-shutoff  │      │ E12: Tariffs       │      │                  │
│                  │      │ + HU-038 (CSV)    │      │                    │      │                  │
└─────────────────┘      └──────────────────┘      └──────────────────┘      └─────────────────┘
        73 SP                    88 SP                     70 SP                    48 SP
                                                                          Total: 279 SP
```

---

## Entrega 1 — MVP

> **Goal:** A user registers, creates a place, links a device, and sees real-time consumption on a dashboard.

### E1 — Cuentas y acceso

| ID | User story | SP | Priority |
|----|-----------|----|---------:|
| HU-001 | Registro con correo y contraseña | 5 | Must |
| HU-002 | Verificación de cuenta por correo | 3 | Must |
| HU-003 | Inicio de sesión | 5 | Must |
| HU-004 | Registro e inicio de sesión con Google | 5 | Must |
| HU-005 | Recuperación de contraseña | 3 | Must |
| HU-006 | Cierre de sesión | 1 | Must |
| HU-007 | Gestión de perfil | 3 | Should |
| HU-008 | Eliminar cuenta | 3 | Should |

**Epic total: 28 SP · 8 HUs**

### E2 — Gestión de lugares

| ID | User story | SP | Priority |
|----|-----------|----|---------:|
| HU-009 | Crear un lugar | 3 | Must |
| HU-010 | Listar y cambiar entre lugares | 3 | Must |
| HU-011 | Editar y eliminar un lugar | 2 | Should |

**Epic total: 8 SP · 3 HUs**

### E3 — Vinculación del dispositivo IoT

| ID | User story | SP | Priority |
|----|-----------|----|---------:|
| HU-012 | Vincular dispositivo a un lugar | 8 | Must |
| HU-013 | Ver estado del dispositivo | 3 | Must |
| HU-014 | Desvincular dispositivo | 3 | Should |

**Epic total: 14 SP · 3 HUs**

### E4 — Monitoreo de consumo en tiempo real

| ID | User story | SP | Priority |
|----|-----------|----|---------:|
| HU-015 | Ver consumo en tiempo real | 8 | Must |
| HU-016 | Selector de período | 5 | Must |
| HU-017 | Gráfica de consumo | 5 | Must |
| HU-018 | Ver caudal en tiempo real | 5 | Should |

**Epic total: 23 SP · 4 HUs**

### Entrega 1 summary

| Metric | Value |
|--------|-------|
| Epics | 4 (E1–E4) |
| User stories | 18 |
| Story points | 73 SP |
| Services involved | api-gateway, auth-service, place-service, device-service, consumption-service |
| Schemas activated | security, places, devices, monitoring (partial) |

---

## Entrega 2 — Smart Monitoring

> **Goal:** Users control the valve remotely, set consumption goals, and receive alerts for anomalies and leaks.

### E5 — Control manual de la válvula

| ID | User story | SP | Priority |
|----|-----------|----|---------:|
| HU-019 | Ver estado de la válvula | 2 | Must |
| HU-020 | Cerrar la válvula manualmente | 8 | Must |
| HU-021 | Abrir la válvula | 5 | Must |
| HU-022 | Historial de acciones sobre la válvula | 3 | Should |

**Epic total: 18 SP · 4 HUs**

### E6 — Metas de consumo

| ID | User story | SP | Priority |
|----|-----------|----|---------:|
| HU-023 | Definir meta mensual | 3 | Should |
| HU-024 | Ver avance frente a la meta | 3 | Should |

**Epic total: 6 SP · 2 HUs**

### E7 — Alertas y notificaciones

| ID | User story | SP | Priority |
|----|-----------|----|---------:|
| HU-025 | Centro de notificaciones | 5 | Must |
| HU-026 | Notificación push al celular | 5 | Must |
| HU-027 | Notificación por correo | 3 | Should |
| HU-028 | Notificación por WhatsApp / SMS | 8 | Should |
| HU-029 | Alerta de fuga sospechosa | 8 | Must |
| HU-030 | Alerta de meta superada | 3 | Should |
| HU-031 | Alerta preventiva de meta | 3 | Could |
| HU-032 | Alerta de sensor desconectado | 3 | Should |
| HU-033 | Alerta de válvula cerrada | 2 | Must |
| HU-034 | Preferencias de notificación | 5 | Should |

**Epic total: 45 SP · 10 HUs**

### E8 — Cierre automático de la válvula

| ID | User story | SP | Priority |
|----|-----------|----|---------:|
| HU-035 | Configurar reglas de cierre automático | 5 | Should |
| HU-036 | Cierre automático por fuga con confirmación | 8 | Should |
| HU-037 | Reapertura tras cierre automático | 3 | Should |

**Epic total: 16 SP · 3 HUs**

### HU-038 — Exportar datos a Excel/CSV

| ID | User story | SP | Priority |
|----|-----------|----|---------:|
| HU-038 | Exportar datos a Excel/CSV | 3 | Could |

> **Note:** HU-038 belongs to E13 (Reportes descargables) but is delivered early in Entrega 2.

### Entrega 2 summary

| Metric | Value |
|--------|-------|
| Epics | 4 (E5–E8) + HU-038 |
| User stories | 20 |
| Story points | 88 SP |
| New services involved | valve-service, notification-service |
| Schemas activated | valve, goals, alerts, notifications |

---

## Entrega 3 — Social & Billing

> **Goal:** Owners invite guests and collaborators, manage utility bills, and configure tariffs.

### E9 — Invitado anónimo

| ID | User story | SP | Priority |
|----|-----------|----|---------:|
| HU-039 | Generar enlace de acceso para invitados | 5 | Should |
| HU-040 | Ver consumo como invitado anónimo | 3 | Should |
| HU-041 | Restricción de solo lectura del invitado | 5 | Must |

**Epic total: 13 SP · 3 HUs**

### E10 — Colaborador

| ID | User story | SP | Priority |
|----|-----------|----|---------:|
| HU-042 | Invitar colaborador por correo | 5 | Should |
| HU-043 | Aceptar invitación como colaborador | 3 | Should |
| HU-044 | Definir permisos del colaborador | 5 | Should |
| HU-045 | Colaborador ejecuta acciones según permisos | 5 | Must |
| HU-046 | Revocar acceso de colaborador | 3 | Should |
| HU-047 | Colaborador recibe alertas del lugar | 3 | Could |

**Epic total: 24 SP · 6 HUs**

### E11 — Recibos del acueducto

| ID | User story | SP | Priority |
|----|-----------|----|---------:|
| HU-048 | Registrar un recibo manualmente | 5 | Should |
| HU-049 | Ver historial de recibos | 3 | Should |
| HU-050 | Comparar recibo contra sensor | 5 | Should |
| HU-051 | Alerta de recibo desalineado | 3 | Should |
| HU-052 | Recordatorio de pago del recibo | 3 | Should |

**Epic total: 19 SP · 5 HUs**

### E12 — Tarifas y unidades de medida

| ID | User story | SP | Priority |
|----|-----------|----|---------:|
| HU-053 | Seleccionar país y moneda | 3 | Should |
| HU-054 | Ingresar la tarifa manualmente | 3 | Should |
| HU-055 | Seleccionar unidad de medida | 3 | Should |
| HU-056 | Ver el consumo en dinero | 5 | Should |

**Epic total: 14 SP · 4 HUs**

### Entrega 3 summary

| Metric | Value |
|--------|-------|
| Epics | 4 (E9–E12) |
| User stories | 18 |
| Story points | 70 SP |
| Schema changes | billing (bills, tariffs), places (members, invitations, permissions) |

---

## Entrega 4 — Administration & Analytics

> **Goal:** Downloadable reports, admin panel (with savings tips), and pre-loaded tariffs for Colombia.

### E13 — Reportes descargables

| ID | User story | SP | Priority |
|----|-----------|----|---------:|
| HU-057 | Generar reporte por período | 5 | Could |
| HU-058 | Exportar reporte a PDF | 5 | Could |

> **Note:** HU-038 (Exportar datos a Excel/CSV) belongs to E13 but is delivered in Entrega 2.

**Epic total: 10 SP · 2 HUs** (+ HU-038 delivered in Entrega 2)

### E14 — Admin Panel (incluye consejos de ahorro)

| ID | User story | SP | Priority |
|----|-----------|----|---------:|
| HU-059 | Admin ve lista de usuarios registrados | 3 | Must |
| HU-060 | Admin bloquea/desbloquea usuario | 3 | Must |
| HU-061 | Admin ve lista de dispositivos | 3 | Should |
| HU-062 | Admin ve métricas de plataforma | 5 | Should |
| HU-063 | Admin crea recomendación de ahorro | 5 | Should |
| HU-064 | Admin edita o desactiva recomendación | 3 | Could |
| HU-065 | Ver consejos de ahorro de agua | 3 | Could |

**Epic total: 25 SP · 7 HUs**

### E15 — Tarifas precargadas, solo Colombia

| ID | User story | SP | Priority |
|----|-----------|----|---------:|
| HU-066 | Catálogo de tarifas de Colombia | 8 | Could |
| HU-067 | Catálogo de tarifas de Ecuador | 5 | Won't |
| HU-068 | Catálogo de tarifas de Estados Unidos | 8 | Won't |
| HU-069 | Cálculo automático con tarifa precargada | 5 | Could |

> **Note:** HU-067 and HU-068 are Won't (deferred). Active SP: 13; total including Won't: 26 SP.

**Epic total: 13 SP active · 4 HUs** (26 SP including Won't)

### Entrega 4 summary

| Metric | Value |
|--------|-------|
| Epics | 3 (E13–E15) |
| User stories | 13 |
| Story points | 48 SP |
| New role | Admin (Administrator) |

---

## Cumulative progress by delivery

| Entrega | Epics | User stories | Story points | Cumulative SP |
|---------|-------|-------------|-------------|---------------|
| Entrega 1 (MVP) | E1–E4 | 18 | 73 | 73 |
| Entrega 2 (Smart) | E5–E8 + HU-038 | 20 | 88 | 161 |
| Entrega 3 (Social) | E9–E12 | 18 | 70 | 231 |
| Entrega 4 (Admin) | E13–E15 | 13 | 48 | 279 |
| **Total** | **15 epics** | **69 HUs** | **279 SP** | |

> **Changes in this release:**
> - Restructured to 15 epics (E1–E15) matching Product Backlog v2.0
> - Moved Automatic Shutoff (E8) from Entrega 3 to Entrega 2
> - HU-038 (CSV export) belongs to E13 but delivered early in Entrega 2
> - All HU IDs, titles, SP, and priorities aligned to PB v2.0
> - E1 now includes HU-002 (email verification) and HU-006 (logout); total 28 SP, 8 HUs
> - E2 reduced to 3 HUs (8 SP): crear, listar, editar/eliminar lugar
> - E4 includes HU-018 (Ver caudal en tiempo real) replacing old consumption summary
> - E6 reduced to 2 HUs (6 SP): definir meta + ver avance
> - E7 expanded to 10 HUs (45 SP) with full notification channel and alert type coverage
> - E8 reduced to 3 HUs (16 SP)
> - E14 expanded to 7 HUs (25 SP) including savings tips (HU-063–HU-065)
> - E15 includes 4 HUs with Won't items (HU-067, HU-068) explicitly listed
> - Total: 69 HUs, 279 SP across 15 epics

---

## Dependencies between deliveries

```
Entrega 1 (MVP)
    ├──→ Entrega 2 (Smart)
    │        ├──→ Entrega 3 (Social)  [needs alerts + valve from E2]
    │        └──→ Entrega 4 (Admin)   [needs all data from E1+E2]
    └──→ Entrega 3 (Social)          [needs auth + place from E1]
```

| Dependency | From | To | Reason |
|-----------|------|-----|--------|
| Auth & Place | E1 | All others | Every feature requires authenticated users and registered places |
| Device + Dashboard | E3, E4 | E5–E7 | Goals and alerts need consumption data flowing |
| Alerts | E7 | E8 | Auto-shutoff rules trigger alerts |
| Valve control | E5 | E8 | Auto-shutoff sends valve commands |
| All data | E1–E8 | E13, E14 | Admin and reports need all domain data |

---

## Correlations

- Product vision → `03-product/vision.md`
- Problem framing → `03-product/problem-framing.md`
- User stories (detailed) → `04-requirements/user-stories.md`
- Domain events by Entrega → `02-domain/domain-events.md`
- Database tables by Entrega → `06-data/models.md`
- Architecture overview → `05-architecture/overview.md`
