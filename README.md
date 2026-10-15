# Triage automatizado de tickets de soporte interno

Entrega final · Ecosistema de Automatización IA
**David Jiménez Acevedo** · Octubre de 2026

Sistema que recibe por correo las solicitudes de soporte de TI y mantenimiento de una planta manufacturera, las clasifica con un modelo de lenguaje, aplica reglas de negocio sobre la criticidad, asigna el técnico responsable y responde al solicitante, deteniéndose para aprobación humana cuando el caso es crítico.

> Todos los solicitantes, equipos, técnicos y tickets son datos ficticios creados para el ejercicio.

---

## Video demostrativo

**[Ver el video ]https://youtu.be/VWVipfx0Nq8?si=xj514N4C2hhUEMSz**

---

## Enlaces de la entrega

| Recurso | Enlace |
|---|---|
| **Dashboard de control** (vista pública) | [Panel de KPIs](PEGAR_ENLACE_KPIS) |
| Tickets por estado (vista pública) | [Abrir](PEGAR_ENLACE_TICKETS) |
| Monitor de errores (vista pública) | [Abrir](PEGAR_ENLACE_ERRORES) |
| Base de datos en modo lectura | [Abrir](PEGAR_ENLACE_BASE) |

Los tres enlaces de vistas se verificaron en ventana de incógnito: abren sin iniciar sesión.

---

## Archivos de esta entrega

| Archivo | Contenido |
|---|---|
| [Triage_Tickets_Arquitectura.pdf](Triage_Tickets_Arquitectura.pdf) | Documentación completa: los cinco criterios desarrollados |
| [triage-tickets.json](triage-tickets.json) | Workflow principal de n8n, sin credenciales |
| [error-handler-tickets.json](error-handler-tickets.json) | Workflow de manejo de errores |
| [evidencias/](evidencias/) | Capturas de pantalla del sistema en funcionamiento |

---

## Resumen del sistema

### Qué automatiza

Las solicitudes de soporte llegan por correo en texto libre: la gente no indica el código del equipo, no dice qué tan grave es, o dice que todo es urgente. Alguien tiene que leer cada correo para decidir de qué se trata, qué tan grave es y a quién le corresponde. Eso es trabajo de interpretación, y es donde aporta un modelo de lenguaje.

### Arquitectura

Tres ramas: una reactiva que se dispara con cada correo, una programada que produce el panel de control a las 17:00, y un workflow independiente que captura cualquier error.

El router de criticidad abre cuatro rutas:

| Ruta | Salida |
|---|---|
| Información incompleta | Correo de plantilla pidiendo los datos faltantes |
| Criticidad baja y media | Acuse de plantilla y notificación al técnico |
| Criticidad alta y crítica | Borrador con IA → **aprobación humana** → correo y escalamiento |
| Revisión manual | Aviso por Telegram, sin respuesta automática |

### La IA propone, el código decide

El modelo nunca determina la criticidad final por sí solo. Devuelve una propuesta y un nodo de código la eleva si el activo afectado es crítico para producción o si hay parada de línea. Ambos valores quedan guardados por separado, lo que permite auditar después qué tanto se equivoca el modelo.

| Criticidad | SLA |
|---|---|
| Crítica | 2 h |
| Alta | 4 h |
| Media | 8 h |
| Baja | 24 h |

### Optimización de costos

| Tarea | Modelo | Costo mensual |
|---|---|---|
| Triage (396 llamadas) | Gemini 3.1 Flash-Lite | USD 0,26 |
| Redacción de críticos (79 llamadas) | Gemini 3.8 Flash | USD 0,21 |
| Acuses, solicitudes de datos y KPIs | Sin IA, plantillas y código | USD 0 |

**USD 0,47 al mes** frente a USD 2,69 si se usara un modelo de gama alta para todo: 82 % de ahorro.

### Seguridad y resiliencia

Al modelo solo llegan el texto limpio del problema, el catálogo de categorías y el listado de activos. Se elimina el historial citado y las firmas antes de cada llamada.

Si la API de IA falla, el ticket se crea igual en estado Error, se registra en el log y llega una alerta: nunca se pierde una solicitud. El Gmail Trigger descarta los correos propios para evitar el bucle infinito, y antes de clasificar se verifica que el hilo no tenga ya un ticket.

Ninguna comunicación generada por IA llega a un solicitante sin que una persona la haya aprobado.

El detalle completo de los cinco criterios está en el PDF.

---

## Stack

n8n · Airtable · Gemini API · Gmail · Telegram
