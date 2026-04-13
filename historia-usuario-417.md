# Historia de Usuario — ConviveITESO Issue #417
## Gestión del Ciclo de Vida de Eventos

---

## Historia de Usuario

**Como** (Organizador de eventos de la comunidad ITESO (alumno, profesor o empleado con rol de creador/organizador de eventos en ConviveITESO)), quiero **gestionar el ciclo de vida completo de un evento a través de estados definidos (borrador, pendiente, aprobado, publicado, rechazado, cancelado, finalizado y pospuesto)**, para **controlar la visibilidad, el flujo de aprobación, la operación y el cierre de mis eventos de manera consistente, auditable y alineada con las políticas de la plataforma ConviveITESO**.

---

## Descripción (detallada)

ConviveITESO es la plataforma universitaria del ITESO para la gestión y descubrimiento de eventos dentro de la comunidad académica. Para garantizar el orden operativo, la calidad editorial y el control administrativo de los eventos publicados, la plataforma requiere un **modelo de ciclo de vida** bien definido.

Cada evento en ConviveITESO debe transitar por un conjunto de **estados predefinidos**, donde cada estado determina:

- **Quién puede editar** el evento (solo el organizador en ciertos estados; moderadores/admins en otros).
- **Si el evento es visible** para la comunidad general (listados, búsqueda, detalle público).
- **Qué acciones están disponibles** según el estado (inscripción, generación de QR, cancelación, etc.).
- **Las reglas de negocio y validaciones** propias de cada transición.
- **Trazabilidad completa**: registro de quién cambió el estado, cuándo y por qué.

### Estados del ciclo de vida

| Estado | Descripción |
|---|---|
| **Borrador** | El evento está siendo creado/editado por el organizador. No es visible para el público. |
| **Pendiente** | El organizador envió el evento a revisión. Espera decisión de un moderador/admin. |
| **Aprobado** | Un moderador/admin aprobó el evento. Puede proceder a publicarse. |
| **Publicado** | El evento es visible para la comunidad. Admite inscripciones, acceso y generación de QR. |
| **Rechazado** | El moderador/admin no aprobó el evento. El organizador puede corregir y reenviar. |
| **Cancelado** | El evento fue cancelado. Se refleja como cancelado; las inscripciones quedan bloqueadas. |
| **Finalizado** | El evento concluyó. Queda como histórico consultable; no permite acciones operativas. |
| **Pospuesto** | El evento cambió de fecha/hora. Sigue activo pero con visibilidad especial de "pospuesto". |

### Flujo de transiciones permitidas

```
Borrador → Pendiente (por el organizador, al cumplir campos mínimos)
Pendiente → Aprobado (por moderador/admin)
Pendiente → Rechazado (por moderador/admin, con motivo obligatorio)
Aprobado → Publicado (por el organizador o admin)
Rechazado → Pendiente (por el organizador, tras corregir el evento)
Publicado → Cancelado (por el organizador o admin)
Publicado → Pospuesto (por el organizador o admin, con nuevas fechas)
Publicado → Finalizado (automático al vencer la fecha, o manual por admin)
Pospuesto → Cancelado (por el organizador o admin)
Pospuesto → Finalizado (automático al vencer la nueva fecha, o manual por admin)
```

### Contexto y módulos afectados

- **Creación/edición de eventos**: solo permitida en estados Borrador (organizador) y Rechazado (organizador para corrección).
- **Detalle de evento**: muestra el estado actual con etiqueta visual; ajusta las acciones disponibles según el estado.
- **Listado/búsqueda pública**: solo muestra eventos Publicados (y Pospuestos con indicador especial, según política del producto).
- **Panel "Mis eventos"** (organizador): muestra todos sus eventos con su estado.
- **Panel de moderación** (moderador/admin): muestra eventos en estado Pendiente listos para revisión.
- **Integración con inscripciones y QR**: solo eventos en estado Publicado (o Pospuesto si aplica) permiten inscripción y generación de pase QR.

### Reglas de negocio clave

- Un evento en **Borrador** no puede pasar a Publicado directamente; debe pasar por Pendiente → Aprobado → Publicado.
- Al cambiar a **Rechazado**, el moderador/admin debe registrar un motivo obligatorio que el organizador pueda ver.
- Al cambiar a **Pospuesto**, se debe registrar la nueva fecha/hora y opcionalmente conservar la original para historial.
- Al cambiar a **Cancelado** o **Finalizado**, el sistema debe bloquear inscripciones y generación de QR.
- La **finalización automática** ocurre cuando la fecha/hora de término del evento vence (según configuración del sistema).
- Solo roles autorizados (**moderador**, **admin**) pueden aprobar/rechazar eventos.
- El organizador no puede auto-aprobar sus propios eventos.

### Manejo de errores (alto nivel)

- Transición inválida → el sistema bloquea la acción y muestra mensaje explicativo.
- Falta de campos requeridos al enviar a Pendiente → validación en cliente y servidor, mensaje claro.
- Intento de inscripción/QR en evento no Publicado → mensaje informativo según el estado actual.
- Falta de motivo al rechazar → campo obligatorio, no permite enviar sin él.

### Supuestos / Dependencias

- Existe un modelo de roles en ConviveITESO: **organizador**, **moderador**, **admin** (y usuario regular).
- El sistema ya cuenta con entidades de Eventos y Usuarios.
- El módulo de inscripciones y generación de QR (issue #428) depende del estado **Publicado**.
- Se asume un mecanismo de notificaciones (al menos para cambios críticos: aprobado, rechazado, cancelado).

---

## Criterios de aceptación (Checklist)

- [ ] El sistema soporta exactamente los 8 estados: **Borrador, Pendiente, Aprobado, Publicado, Rechazado, Cancelado, Finalizado, Pospuesto**.
- [ ] Un evento nuevo se crea automáticamente en estado **Borrador** y solo el organizador lo ve hasta que sea Publicado.
- [ ] En estado **Borrador**, el organizador puede editar todos los campos del evento sin que sea visible para el público.
- [ ] El organizador puede enviar un evento de **Borrador → Pendiente** solo si cumple los campos mínimos requeridos (título, descripción, fecha de inicio, fecha de fin, ubicación y cupo si aplica).
- [ ] En estado **Pendiente**, el evento no puede publicarse directamente; requiere decisión de moderador/admin.
- [ ] Un moderador o admin puede aprobar un evento (**Pendiente → Aprobado**) y el organizador recibe notificación al respecto.
- [ ] Un moderador o admin puede rechazar un evento (**Pendiente → Rechazado**) y debe ingresar un **motivo obligatorio** visible para el organizador.
- [ ] En estado **Rechazado**, el organizador puede editar el evento para corregirlo y reenviarlo a **Pendiente** nuevamente.
- [ ] Un evento solo puede pasar a **Publicado** si está en estado **Aprobado**; intentos desde Borrador, Pendiente o Rechazado son bloqueados.
- [ ] Cuando el evento está **Publicado**, aparece en los listados/búsqueda para usuarios de la comunidad ITESO.
- [ ] En estado **Publicado**, los miembros de la comunidad pueden inscribirse y generar pase QR (integración con funcionalidades de acceso).
- [ ] Un evento **Publicado** puede marcarse como **Cancelado** por el organizador o admin; al hacerlo, se bloquean inscripciones y generación de QR, y se muestra etiqueta "Cancelado" en su detalle.
- [ ] Un evento **Publicado** puede marcarse como **Pospuesto** por el organizador o admin; deben registrarse las nuevas fechas y el evento sigue visible con etiqueta "Pospuesto".
- [ ] Un evento llega a estado **Finalizado** automáticamente cuando vence su fecha/hora de término, o manualmente por un admin; en este estado el evento solo es consultable como histórico.
- [ ] En estados **Cancelado** y **Finalizado**, el evento no permite inscripciones nuevas, generación de QR ni modificaciones operativas.
- [ ] El organizador no puede aprobar/rechazar sus propios eventos (validación de permisos por rol).
- [ ] Todo cambio de estado genera un registro de auditoría con: estado anterior, estado nuevo, usuario responsable, timestamp y motivo (si aplica).
- [ ] El detalle del evento muestra de forma destacada el estado actual (etiqueta/badge visual), especialmente cuando es Cancelado, Rechazado o Pospuesto.

*(18 criterios: se exceden los 12 por la complejidad del ciclo de vida, los múltiples roles, las transiciones, la auditoría y la integración con otros módulos de ConviveITESO.)*

---

## Detalles de implementación (alto nivel)

### Front End (alto nivel)

- **Vistas/componentes afectados:**
  - **Formulario de creación/edición de evento**: visible y editable solo en estados Borrador y Rechazado (para el organizador).
  - **Detalle del evento**: muestra etiqueta/badge de estado actual; adapta las acciones disponibles según el estado y el rol del usuario actual.
  - **Panel "Mis eventos" (organizador)**: listado con columna/indicador de estado para cada evento.
  - **Panel de moderación** (moderador/admin): listado filtrado de eventos en estado Pendiente; con opciones "Aprobar" y "Rechazar" (campo de motivo obligatorio al rechazar).
  - **Listado/búsqueda pública**: filtrar eventos solo en estado Publicado (y Pospuesto si aplica política) para la comunidad.

- **Estados de UI por componente:**
  - Loading mientras se procesa el cambio de estado.
  - Confirmación antes de cambios críticos (Cancelar, Posponer, Finalizar).
  - Modal o campo en línea para ingresar motivo al rechazar.
  - Error con mensaje claro cuando la transición no es válida.
  - Mensaje de éxito al completar la transición.

- **Eventos UI clave:**
  - Botón "Enviar a revisión" (Borrador → Pendiente) → valida campos y llama al servicio.
  - Botones "Aprobar" / "Rechazar" (moderación) → confirmar y llamar al servicio.
  - Botón "Publicar" (Aprobado → Publicado) → llamar al servicio.
  - Botón "Cancelar evento" / "Posponer evento" → confirmación + llamar al servicio.
  - Badge de estado con tooltip o explicación cuando el estado limita acciones al usuario.

### Back End (alto nivel)

- **Módulo/servicio de ciclo de vida de eventos:**
  - Implementar una **máquina de estados** que centralice las reglas de transición permitidas.
  - Exponer operaciones de cambio de estado con validaciones server-side (rol, transición válida, campos requeridos).
  - Validar que el organizador no puede aprobar/rechazar sus propios eventos.
  - Registrar auditoría en cada cambio de estado.

- **Lógica de negocio clave:**
  - Validar campos mínimos antes de pasar a Pendiente.
  - Requerir motivo al rechazar (campo no nulo/vacío).
  - Al publicar: habilitar visibilidad pública y permitir inscripciones/QR.
  - Al cancelar/finalizar: bloquear inscripciones y acceso QR.
  - Al posponer: actualizar fechas, preservar originales, mantener visibilidad.
  - Proceso automático (job/scheduler) para marcar eventos como Finalizados cuando vence su fecha/hora.

- **Integración interna:**
  - Módulo de inscripciones: verificar estado del evento antes de permitir nueva inscripción.
  - Módulo de QR/acceso rápido (issue #428): verificar estado Publicado antes de generar pase.
  - Módulo de notificaciones: disparar notificación al organizador en cambios de Aprobado, Rechazado, Cancelado (según reglas del producto).

- **Manejo de errores:**
  - Respuestas estandarizadas para: transición inválida, permisos insuficientes, campos faltantes, evento no encontrado.

- **Observabilidad:**
  - Log de cada cambio de estado con contexto (userId, eventId, from, to, reason, timestamp).

### Base de Datos (alto nivel)

- **Tabla/entidad `events` (existente):**
  - Agregar campo `status` (enum o string con valores: `DRAFT`, `PENDING`, `APPROVED`, `PUBLISHED`, `REJECTED`, `CANCELLED`, `FINISHED`, `POSTPONED`).
  - Agregar campos para posposición: `postponed_start_at`, `postponed_end_at` (o equivalente), `original_start_at`, `original_end_at` (para preservar fechas originales si se pospone).
  - Índice sobre `status` para filtrados eficientes en listados.

- **Tabla/entidad `event_status_history` (nueva):**
  - Registra cada cambio de estado como auditoría.
  - Campos: `id`, `event_id` (FK → events), `from_status`, `to_status`, `changed_by_user_id` (FK → users), `changed_at` (timestamp), `reason` (texto, nullable salvo en Rechazado).

- **Relaciones:**
  - `event_status_history.event_id` → `events.id`.
  - `event_status_history.changed_by_user_id` → `users.id`.

- **Operaciones esperadas:**
  - Insertar/actualizar campo `status` en `events` al cambiar de estado.
  - Insertar registro en `event_status_history` en cada transición.
  - Consultar eventos filtrados por `status` para listados (moderación, público, organizador).
  - Consultar historial de un evento para trazabilidad/auditoría.

- **Migraciones necesarias:**
  - Migración para agregar columna `status` (con valor por defecto `DRAFT`) a `events`.
  - Migración para agregar columnas de posposición a `events`.
  - Migración para crear tabla `event_status_history`.

---

## Notas de prueba

### Funcionales (flujo/UI)
- Crear un evento nuevo y verificar que inicia en estado **Borrador** y no aparece en listados públicos.
- Editar el evento en Borrador y confirmar que los cambios se guardan correctamente.
- Enviar a revisión (Borrador → Pendiente) y verificar que cumple campos mínimos; intentar sin campos y confirmar bloqueo con mensaje.
- Con rol de moderador: aprobar un evento Pendiente y verificar cambio a Aprobado + notificación al organizador.
- Con rol de moderador: rechazar un evento Pendiente sin motivo (debe bloquearse); con motivo (debe guardar y notificar).
- Con rol de organizador: corregir un evento Rechazado y reenviarlo a Pendiente.
- Publicar un evento Aprobado y verificar que aparece en listados públicos y permite inscripción/QR.
- Posponer un evento Publicado: verificar nuevas fechas guardadas, original preservado y badge "Pospuesto" visible.
- Cancelar un evento Publicado: verificar bloqueo de inscripciones, badge "Cancelado" visible, no aparece en búsqueda activa.
- Verificar finalización automática al vencer fecha/hora (si el scheduler está activo); o manualmente por admin.

### Negativas (validaciones)
- Intentar pasar de Borrador directamente a Publicado → debe bloquearse con mensaje.
- Intentar aprobar/rechazar teniendo rol de organizador → debe ser denegado.
- Organizador intenta aprobar su propio evento → debe ser bloqueado.
- Intentar inscribirse o generar QR en evento Cancelado o Finalizado → debe bloquearse con mensaje de estado.
- Rechazar sin ingresar motivo → campo requerido, acción bloqueada.

### Roles y permisos
- **Organizador**: puede crear (Borrador), editar (Borrador/Rechazado), enviar a revisión, publicar (si Aprobado), cancelar/posponer (si Publicado).
- **Moderador/Admin**: puede aprobar, rechazar, cancelar, finalizar manualmente.
- **Usuario regular**: solo puede ver eventos Publicados/Pospuestos y registrarse en ellos.

### Consideraciones de regresión
- Verificar que módulo de inscripciones respeta el estado del evento (no permite inscripción si no está Publicado).
- Verificar que módulo de QR (issue #428) respeta el estado del evento.
- Verificar que listados públicos no filtran eventos en estados no públicos.
- Verificar que el historial de estados no genera duplicados en flujos normales.

### Datos de prueba sugeridos
- Evento A: en estado Borrador, sin todos los campos mínimos → para probar validación al enviar a revisión.
- Evento B: en estado Pendiente → para probar aprobación y rechazo.
- Evento C: en estado Publicado con fecha de fin en el pasado → para probar finalización automática.
- Evento D: en estado Publicado → para probar posposición y cancelación.
- Evento E: en estado Rechazado → para probar corrección y reenvío a Pendiente.
