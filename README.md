# Sistema de Gestión de Tareas (Tipo Trello) 🗂️

Buena elección de proyecto, Alan. Es un sistema que toca muchos conceptos avanzados a la vez. Te estructuro todo de forma ordenada.

---

## 📐 Arquitectura General

Antes de dividir tareas, es importante que los 3 entiendan la arquitectura completa:

```
Frontend (HTML/CSS/JS vanilla o con HTMX)
        ↕  HTTP REST
Backend Django (Views, ORM, Permisos)
        ↕  WebSocket
Django Channels (Redis como layer)
        ↕
Base de Datos (PostgreSQL o MariaDB)
```

---

## 🗃️ Modelos y Relaciones

Esta es la base del proyecto. Los 3 deben conocerla bien antes de empezar:

```
Usuario (auth.User de Django)
  └── Tablero (Board)  ←→  Miembros (ManyToMany con User)
        └── Lista (List)  [tiene campo "posición"]
              └── Tarjeta (Card)  [tiene campo "posición"]
                    ├── Asignados (ManyToMany con User)
                    └── Notificación (Notification)
```

---

## 👥 División de Tareas por Persona

### 🔵 Persona 1 — Backend Core (Modelos + Auth + Permisos)

Es la base de todo. Debe estar lista primero.

| Tarea | Descripción |
|---|---|
| Modelos | `Board`, `List`, `Card`, `Membership` |
| Auth | Registro, login, logout, sesiones |
| Permisos | Que cada usuario solo vea sus tableros |
| Admin Django | Panel para gestionar datos en desarrollo |
| Migraciones | Mantener el estado de la BD limpio |

### 🟢 Persona 2 — Vistas + API + Drag & Drop

Construye sobre la base de Persona 1.

| Tarea | Descripción |
|---|---|
| CRUD Tableros | Crear, editar, eliminar tableros |
| CRUD Listas y Tarjetas | Con validación de permisos |
| Orden / Posición | Lógica para reordenar con drag & drop |
| Asignación | Asignar tarjetas a miembros del tablero |
| Endpoints | Vistas que devuelven JSON para el JS del frontend |

### 🔴 Persona 3 — Django Channels + Notificaciones + Frontend JS

La parte más compleja técnicamente.

| Tarea | Descripción |
|---|---|
| Configurar Channels | Instalar, configurar Redis, routing |
| Consumers | Lógica WebSocket para actualizaciones en tiempo real |
| Notificaciones | Modelo + envío cuando se asigna una tarjeta |
| JS Frontend | Drag & drop (con SortableJS), conexión al WebSocket |
| Templates base | Estructura HTML mínima funcional |

---

## 📋 Orden de Implementación Recomendado

El proyecto tiene dependencias claras. Seguir este orden evita bloqueos:

```
Semana 1
├── [P1] Modelos + migraciones + admin
├── [P1] Sistema de auth completo
└── [P2] Empieza CRUD básico de tableros

Semana 2
├── [P2] CRUD listas y tarjetas + permisos
├── [P2] Lógica de posiciones (campo order)
└── [P3] Configuración de Channels + Redis

Semana 3
├── [P3] Consumers WebSocket (sync de tablero)
├── [P2+P3] Endpoints JSON + JS drag & drop
└── [P1+P3] Modelo notificaciones + envío

Semana 4
├── Todo: Integración y pruebas cruzadas
├── Todo: Corrección de bugs de permisos
└── Todo: Deploy o entrega final
```

---

## 🔐 Buenas Prácticas de Seguridad

### Autenticación y Sesiones
- Usar `@login_required` en **todas** las vistas sin excepción
- Configurar `SESSION_COOKIE_HTTPONLY = True` y `SESSION_COOKIE_SECURE = True`
- Usar `django-axes` para bloquear intentos de login fallidos (ya lo conocés de proyectos anteriores)

### Permisos a nivel de objeto
No basta con `@login_required`. Cada vista debe verificar que el usuario **pertenece** al tablero:

```python
# ✅ Correcto — verificar pertenencia antes de devolver datos
def get_board(request, board_id):
    board = get_object_or_404(Board, id=board_id)
    if not board.members.filter(id=request.user.id).exists():
        raise PermissionDenied  # 403, no 404

# ❌ Incorrecto — cualquier usuario logueado puede ver cualquier tablero
def get_board(request, board_id):
    board = get_object_or_404(Board, id=board_id)
    return JsonResponse(...)
```

### WebSockets (Channels)
- Autenticar al usuario en el consumer **antes** de aceptar la conexión
- Verificar que pertenece al tablero al que intenta conectarse por WebSocket
- Usar grupos por tablero: `board_{board_id}` como nombre del grupo

### Variables sensibles
- Todo en `.env` con `python-decouple` o `django-environ`: `SECRET_KEY`, credenciales de BD, Redis URL
- El archivo `.env` en `.gitignore` desde el día 1, nunca al repositorio

### CSRF y XSS
- Nunca deshabilitar el middleware CSRF
- Al hacer peticiones AJAX/fetch, incluir siempre el token CSRF en los headers

---

## 🧹 Buenas Prácticas de Código

### Estructura de carpetas sugerida
```
proyecto/
├── config/          ← settings, urls, asgi, wsgi
├── users/           ← auth, registro, perfil
├── boards/          ← modelos Board, List, Card + sus vistas
├── notifications/   ← modelo Notification + lógica de envío
├── websockets/      ← consumers de Channels
└── templates/
    ├── base.html
    ├── boards/
    └── users/
```

### Reglas de código para el equipo
- Una app Django por dominio (`boards`, `users`, `notifications`) — no meter todo en una sola app
- Los modelos con su lógica de negocio en `models.py`, no en las vistas
- Las vistas solo reciben request, llaman a la lógica, y devuelven response
- Usar `select_related` y `prefetch_related` en queries que traen relaciones — evita el problema N+1
- Comentar las partes no obvias, especialmente los consumers de Channels y la lógica de reordenamiento

### Git en equipo
- Rama `main` protegida — nadie pushea directo
- Cada persona trabaja en su rama: `feature/auth`, `feature/crud-boards`, `feature/websockets`
- Pull Request para mergear, revisado por al menos uno del equipo
- Commits descriptivos: `feat: agregar verificación de permisos en Board`

---

## 🛠️ Stack Técnico Recomendado

| Componente | Tecnología |
|---|---|
| Backend | Django 5.x |
| WebSockets | Django Channels 4.x |
| Message Layer | Redis |
| BD | MariaDB o PostgreSQL |
| Drag & Drop | SortableJS (librería JS liviana) |
| Variables de entorno | python-decouple |
| Frontend | HTML + CSS + JS vanilla (sin framework pesado) |

---
