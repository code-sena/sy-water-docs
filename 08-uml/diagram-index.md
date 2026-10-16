# Diagram Index

> Registry of every diagram of Cuida Tu Agua. Sources are PlantUML files in `diagrams/source/`
> (all include the black-and-white style `_estilo.iuml`); PNG renders are in `diagrams/exports/`.
> Diagram labels are in Spanish because the same images are embedded in the SENA deliverables
> (`assets/deliverables/`). Last aligned with the code: **2026-10-10**.

---

## When to create a diagram?

```
✓ When the flow involves 3+ services or actors
✓ When the architecture needs to be communicated to a non-technical stakeholder
✓ When data modeling is complex (ER diagram)
✓ When a bug took more than 2 hours to understand (retroactive sequence diagram)

✗ For simple 2-element flows — code describes them better
✗ To "document for documentation's sake" — an outdated diagram is worse than none
```

## File naming

```
[TYPE]-[NN]_[short-name].puml   →   exports/[TYPE]-[NN]_[short-name].png
TYPE: CU use case · CL class · CO component · DE deployment · ER physical model
      SQ sequence · AC activity · ST state · NV navigation
```

## How to render

```bash
cd 08-uml/diagrams/source
plantuml -tpng -o ../exports *.puml
```

Update the ER sources after every new Liquibase release of a `ms-*-db` repository.

---

## Diagram registry

**Status:** ✅ matches the implemented code · ◐ implemented scope plus planned stories · 📐 design of planned stories only


### Use cases

| ID | Name | Status | Source | Export |
|----|------|--------|--------|--------|
| CU-00 | Diagrama general de casos de uso — Cuida Tu Agua | ◐ | `source/CU-00_general.puml` | [`CU-00_general.png`](diagrams/exports/CU-00_general.png) |
| CU-01 | Cuentas y acceso (E1) | ✅ | `source/CU-01_cuentas.puml` | [`CU-01_cuentas.png`](diagrams/exports/CU-01_cuentas.png) |
| CU-02 | Lugares y dispositivo IoT (E2, E3) | ✅ | `source/CU-02_lugares_dispositivo.puml` | [`CU-02_lugares_dispositivo.png`](diagrams/exports/CU-02_lugares_dispositivo.png) |
| CU-03 | Consumo y metas (E4, E6, HU-038) | ◐ | `source/CU-03_consumo_metas.puml` | [`CU-03_consumo_metas.png`](diagrams/exports/CU-03_consumo_metas.png) |
| CU-04 | Válvula y cierre automático (E5, E8) | ◐ | `source/CU-04_valvula.puml` | [`CU-04_valvula.png`](diagrams/exports/CU-04_valvula.png) |
| CU-05 | Alertas y notificaciones (E7) | ◐ | `source/CU-05_alertas.puml` | [`CU-05_alertas.png`](diagrams/exports/CU-05_alertas.png) |
| CU-06 | Acceso compartido: invitado y colaborador (E9, E10) | 📐 | `source/CU-06_acceso_compartido.puml` | [`CU-06_acceso_compartido.png`](diagrams/exports/CU-06_acceso_compartido.png) |
| CU-07 | Recibos, tarifas y reportes (E11, E12, E13, E15) | ◐ | `source/CU-07_recibos_tarifas_reportes.puml` | [`CU-07_recibos_tarifas_reportes.png`](diagrams/exports/CU-07_recibos_tarifas_reportes.png) |
| CU-08 | Administración y consejos de ahorro (E14) | ◐ | `source/CU-08_administracion.puml` | [`CU-08_administracion.png`](diagrams/exports/CU-08_administracion.png) |

### Class (domain)

| ID | Name | Status | Source | Export |
|----|------|--------|--------|--------|
| CL-00 | Modelo de dominio general (vista conceptual entre microservicios) | ◐ | `source/CL-00_modelo_dominio.puml` | [`CL-00_modelo_dominio.png`](diagrams/exports/CL-00_modelo_dominio.png) |
| CL-01 | Clases de dominio — ms-iam (cuentas, sesiones y roles) | ✅ | `source/CL-01_ms-iam.puml` | [`CL-01_ms-iam.png`](diagrams/exports/CL-01_ms-iam.png) |
| CL-02 | Clases de dominio — ms-places (lugares, tarifas, consejos y acceso compartido) | ◐ | `source/CL-02_ms-places.puml` | [`CL-02_ms-places.png`](diagrams/exports/CL-02_ms-places.png) |
| CL-03 | Clases de dominio — ms-device (medidores IoT) | ✅ | `source/CL-03_ms-device.puml` | [`CL-03_ms-device.png`](diagrams/exports/CL-03_ms-device.png) |
| CL-04 | Clases de dominio — ms-consumption (lecturas, metas, recibos, fugas y reportes) | ◐ | `source/CL-04_ms-consumption.puml` | [`CL-04_ms-consumption.png`](diagrams/exports/CL-04_ms-consumption.png) |
| CL-05 | Clases de dominio — ms-valve (válvula, órdenes y cierre automático) | ◐ | `source/CL-05_ms-valve.puml` | [`CL-05_ms-valve.png`](diagrams/exports/CL-05_ms-valve.png) |
| CL-06 | Clases de dominio — ms-notification (alertas y canales) | ✅ | `source/CL-06_ms-notification.puml` | [`CL-06_ms-notification.png`](diagrams/exports/CL-06_ms-notification.png) |

### Component

| ID | Name | Status | Source | Export |
|----|------|--------|--------|--------|
| CO-01 | Diagrama de componentes (vista por capas) | ✅ | `source/CO-01_componentes.puml` | [`CO-01_componentes.png`](diagrams/exports/CO-01_componentes.png) |
| CO-02 | Integración entre microservicios (eventos y llamadas internas) | ✅ | `source/CO-02_integracion.puml` | [`CO-02_integracion.png`](diagrams/exports/CO-02_integracion.png) |

### Deployment

| ID | Name | Status | Source | Export |
|----|------|--------|--------|--------|
| DE-01 | Diagrama de despliegue | ✅ | `source/DE-01_despliegue.puml` | [`DE-01_despliegue.png`](diagrams/exports/DE-01_despliegue.png) |

### Physical data model (ER)

| ID | Name | Status | Source | Export |
|----|------|--------|--------|--------|
| ER-00 | Modelo físico general: schemas y referencias entre servicios | ✅ | `source/ER-00_general.puml` | [`ER-00_general.png`](diagrams/exports/ER-00_general.png) |
| ER-01 | Modelo físico del schema security (ms-iam) | ✅ | `source/ER-01_security.puml` | [`ER-01_security.png`](diagrams/exports/ER-01_security.png) |
| ER-02 | Modelo físico del schema places (ms-places) | ✅ | `source/ER-02_places.puml` | [`ER-02_places.png`](diagrams/exports/ER-02_places.png) |
| ER-03 | Modelo físico del schema devices (ms-device) | ✅ | `source/ER-03_devices.puml` | [`ER-03_devices.png`](diagrams/exports/ER-03_devices.png) |
| ER-04 | Modelo físico del schema monitoring (ms-consumption) | ✅ | `source/ER-04_monitoring.puml` | [`ER-04_monitoring.png`](diagrams/exports/ER-04_monitoring.png) |
| ER-05 | Modelo físico del schema valve (ms-valve) | ✅ | `source/ER-05_valve.puml` | [`ER-05_valve.png`](diagrams/exports/ER-05_valve.png) |
| ER-06 | Modelo físico del schema notification (ms-notification) | ✅ | `source/ER-06_notification.puml` | [`ER-06_notification.png`](diagrams/exports/ER-06_notification.png) |

### Sequence

| ID | Name | Status | Source | Export |
|----|------|--------|--------|--------|
| SQ-01 | Registro y verificación de la cuenta (HU-001, HU-002) | ✅ | `source/SQ-01_registro_verificacion.puml` | [`SQ-01_registro_verificacion.png`](diagrams/exports/SQ-01_registro_verificacion.png) |
| SQ-02 | Inicio de sesión, uso del token y cierre de sesión (HU-003, HU-006) | ✅ | `source/SQ-02_sesion.puml` | [`SQ-02_sesion.png`](diagrams/exports/SQ-02_sesion.png) |
| SQ-03 | Vincular un medidor a un lugar (HU-012) | ✅ | `source/SQ-03_vincular_medidor.puml` | [`SQ-03_vincular_medidor.png`](diagrams/exports/SQ-03_vincular_medidor.png) |
| SQ-04 | Telemetría del medidor y consulta de consumo (HU-013, HU-015 a HU-018) | ✅ | `source/SQ-04_telemetria_consumo.puml` | [`SQ-04_telemetria_consumo.png`](diagrams/exports/SQ-04_telemetria_consumo.png) |
| SQ-05 | Cerrar la válvula con código de confirmación (HU-020, HU-033) | ✅ | `source/SQ-05_cerrar_valvula.puml` | [`SQ-05_cerrar_valvula.png`](diagrams/exports/SQ-05_cerrar_valvula.png) |
| SQ-06 | Detección de fuga sospechosa y alerta (HU-029) | 📐 | `source/SQ-06_deteccion_fuga.puml` | [`SQ-06_deteccion_fuga.png`](diagrams/exports/SQ-06_deteccion_fuga.png) |
| SQ-07 | Cierre automático por fuga con confirmación y reapertura (HU-035, HU-036, HU-037) | 📐 | `source/SQ-07_cierre_automatico.puml` | [`SQ-07_cierre_automatico.png`](diagrams/exports/SQ-07_cierre_automatico.png) |
| SQ-08 | Invitar, aceptar y revocar un colaborador (HU-042 a HU-046) | 📐 | `source/SQ-08_colaborador.puml` | [`SQ-08_colaborador.png`](diagrams/exports/SQ-08_colaborador.png) |
| SQ-09 | Registrar un recibo y compararlo con el sensor (HU-048, HU-050, HU-051) | 📐 | `source/SQ-09_recibo.puml` | [`SQ-09_recibo.png`](diagrams/exports/SQ-09_recibo.png) |
| SQ-10 | Eliminar la cuenta (HU-008) | ✅ | `source/SQ-10_eliminar_cuenta.puml` | [`SQ-10_eliminar_cuenta.png`](diagrams/exports/SQ-10_eliminar_cuenta.png) |
| SQ-11 | Métricas de plataforma para el administrador (HU-062) | ✅ | `source/SQ-11_metricas_admin.puml` | [`SQ-11_metricas_admin.png`](diagrams/exports/SQ-11_metricas_admin.png) |
| SQ-12 | Alerta de sensor desconectado (HU-013, HU-032) | ✅ | `source/SQ-12_sensor_desconectado.puml` | [`SQ-12_sensor_desconectado.png`](diagrams/exports/SQ-12_sensor_desconectado.png) |

### Activity

| ID | Name | Status | Source | Export |
|----|------|--------|--------|--------|
| AC-01 | Actividad: registro y verificación de la cuenta (HU-001, HU-002) | ✅ | `source/AC-01_registro.puml` | [`AC-01_registro.png`](diagrams/exports/AC-01_registro.png) |
| AC-02 | Actividad: cierre manual de la válvula (HU-020) | ✅ | `source/AC-02_cierre_manual.puml` | [`AC-02_cierre_manual.png`](diagrams/exports/AC-02_cierre_manual.png) |
| AC-03 | Actividad: detección de fuga y cierre automático con confirmación (HU-029, HU-036, HU-037) | 📐 | `source/AC-03_cierre_automatico.puml` | [`AC-03_cierre_automatico.png`](diagrams/exports/AC-03_cierre_automatico.png) |
| AC-04 | Actividad: registrar y comparar un recibo contra el sensor (HU-048 a HU-051) | 📐 | `source/AC-04_comparar_recibo.puml` | [`AC-04_comparar_recibo.png`](diagrams/exports/AC-04_comparar_recibo.png) |

### State

| ID | Name | Status | Source | Export |
|----|------|--------|--------|--------|
| ST-01 | Estados de la cuenta de usuario | ✅ | `source/ST-01_cuenta.puml` | [`ST-01_cuenta.png`](diagrams/exports/ST-01_cuenta.png) |
| ST-02 | Estados del medidor IoT | ✅ | `source/ST-02_medidor.puml` | [`ST-02_medidor.png`](diagrams/exports/ST-02_medidor.png) |
| ST-03 | Estados de una orden de válvula | ✅ | `source/ST-03_orden_valvula.puml` | [`ST-03_orden_valvula.png`](diagrams/exports/ST-03_orden_valvula.png) |
| ST-04 | Estados de la válvula de un lugar | ✅ | `source/ST-04_valvula.puml` | [`ST-04_valvula.png`](diagrams/exports/ST-04_valvula.png) |

### Navigation map

| ID | Name | Status | Source | Export |
|----|------|--------|--------|--------|
| NV-01 | Mapa de navegación de la aplicación de usuario (móvil y web) | ✅ | `source/NV-01_navegacion_usuario.puml` | [`NV-01_navegacion_usuario.png`](diagrams/exports/NV-01_navegacion_usuario.png) |
| NV-02 | Mapa de navegación del panel web de administración | ✅ | `source/NV-02_navegacion_admin.puml` | [`NV-02_navegacion_admin.png`](diagrams/exports/NV-02_navegacion_admin.png) |

---

## Correlations

- Service documentation that links these diagrams → `09-microservices/services/*/README.md`
- Data model in text → `06-data/models.md` and each service `data-model.md`
- Use cases and HU → `04-requirements/user-stories.md`
