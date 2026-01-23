# Platziflix - Memoria de Proyecto Multi-plataforma

## Visión General

Platziflix es una plataforma de cursos online estilo Netflix con arquitectura multi-plataforma:
- **Backend**: API REST con FastAPI + PostgreSQL (Docker)
- **Frontend**: Aplicación web con Next.js 15 + React 19
- **Mobile**: Apps nativas Android (Kotlin) + iOS (Swift)

---

## Arquitectura del Sistema

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           CLIENTES (Multi-Plataforma)                       │
├─────────────────┬─────────────────────┬─────────────────────────────────────┤
│   Frontend      │   Android           │   iOS                               │
│   Next.js 15    │   Kotlin + Compose  │   Swift + SwiftUI                   │
│   Puerto: 3000  │   MVVM + MVI        │   MVVM + Repository                 │
└────────┬────────┴──────────┬──────────┴──────────────┬──────────────────────┘
         │              HTTP REST API                  │
         └───────────────────┼─────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────────────────────────┐
│                        Backend (FastAPI) - Puerto: 8000                     │
│  Endpoints → Services → Models (SQLAlchemy) → PostgreSQL                   │
└────────────────────────────┬────────────────────────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────────────────────────┐
│                     PostgreSQL 15 (Docker) - Puerto: 5432                   │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Estructura del Proyecto

```
claude-code-master/
├── Backend/                    # API FastAPI + PostgreSQL
│   ├── app/
│   │   ├── main.py            # Endpoints principales
│   │   ├── core/config.py     # Configuración y settings
│   │   ├── models/            # Modelos SQLAlchemy
│   │   ├── services/          # Lógica de negocio
│   │   ├── schemas/           # Validación Pydantic
│   │   ├── db/                # Conexión BD y seeds
│   │   ├── alembic/           # Migraciones
│   │   └── tests/             # Tests
│   ├── docker-compose.yml
│   ├── Dockerfile
│   ├── Makefile
│   └── pyproject.toml
│
├── Frontend/                   # Next.js 15 App
│   ├── src/
│   │   ├── app/               # App Router (páginas)
│   │   │   ├── page.tsx       # Home (catálogo)
│   │   │   ├── course/[slug]/ # Detalle curso
│   │   │   └── classes/[id]/  # Reproductor video
│   │   ├── components/        # Componentes React
│   │   ├── services/          # API clients
│   │   ├── types/             # TypeScript types
│   │   └── styles/            # SCSS globales
│   ├── package.json
│   └── vitest.config.ts
│
├── Mobile/
│   ├── PlatziFlixAndroid/     # Kotlin + Jetpack Compose
│   │   └── app/src/main/java/com/espaciotiago/platziflixandroid/
│   │       ├── di/            # Dependency Injection
│   │       ├── domain/        # Models + Repository interfaces
│   │       ├── data/          # DTOs, Mappers, Remote repos
│   │       └── presentation/  # ViewModels, Screens, Components
│   │
│   └── PlatziFlixiOS/         # Swift + SwiftUI
│       └── PlatziFlixiOS/
│           ├── Domain/        # Models + Protocols
│           ├── Data/          # DTOs, Mappers, Repositories
│           ├── Services/      # Network layer
│           └── Presentation/  # ViewModels, Views
│
└── spec/                       # Documentación técnica
```

---

## Stack Tecnológico Detallado

### Backend (Python)
| Componente | Tecnología | Versión |
|------------|------------|---------|
| Framework | FastAPI | >=0.104.0 |
| Server | Uvicorn | >=0.24.0 |
| ORM | SQLAlchemy | >=2.0.0 |
| BD Driver | psycopg2-binary | >=2.9.0 |
| Validación | Pydantic | >=2.0.0 |
| Migraciones | Alembic | >=1.13.0 |
| Dependencias | UV | latest |
| Testing | pytest + httpx | >=7.0.0 |

### Frontend (TypeScript)
| Componente | Tecnología | Versión |
|------------|------------|---------|
| Framework | Next.js | 15.3.3 |
| UI | React | 19.0.0 |
| Lenguaje | TypeScript | 5.x |
| Estilos | SCSS + CSS Modules | 1.77.0 |
| Testing | Vitest + RTL | 3.2.3 |
| Linting | ESLint | 9.x |

### Android (Kotlin)
| Componente | Tecnología | Versión |
|------------|------------|---------|
| UI | Jetpack Compose | BOM |
| HTTP | Retrofit + OkHttp | 2.9.0 / 4.12.0 |
| JSON | GSON | 2.10.1 |
| Async | Coroutines | 1.7.3 |
| Images | Coil | 2.5.0 |
| Arch | ViewModel | 2.7.0 |

### iOS (Swift)
| Componente | Tecnología |
|------------|------------|
| UI | SwiftUI |
| HTTP | URLSession (nativo) |
| Async | async/await |
| Reactive | Combine |
| Testing | XCTest |

---

## Modelo de Datos

### Diagrama ER
```
┌──────────────────┐         N:M          ┌──────────────────┐
│     TEACHER      │◄────────────────────►│     COURSE       │
├──────────────────┤   course_teachers    ├──────────────────┤
│ id (PK)          │                      │ id (PK)          │
│ name             │                      │ name             │
│ email (UNIQUE)   │                      │ description      │
│ timestamps       │                      │ thumbnail (URL)  │
│ deleted_at       │                      │ slug (UNIQUE)    │
└──────────────────┘                      │ timestamps       │
                                          │ deleted_at       │
                                          └────────┬─────────┘
                                                   │
                              ┌────────────────────┼────────────────────┐
                              │ 1:N                │ 1:N                │
                              ▼                    ▼
                   ┌──────────────────┐  ┌──────────────────┐
                   │     LESSON       │  │  COURSE_RATING   │
                   ├──────────────────┤  ├──────────────────┤
                   │ id (PK)          │  │ id (PK)          │
                   │ course_id (FK)   │  │ course_id (FK)   │
                   │ name             │  │ user_id          │
                   │ description      │  │ rating (1-5)     │
                   │ slug             │  │ timestamps       │
                   │ video_url        │  │ deleted_at       │
                   │ timestamps       │  └──────────────────┘
                   │ deleted_at       │  UNIQUE(course_id, user_id, deleted_at)
                   └──────────────────┘
```

### Características del Modelo
- **Soft Deletes**: Todos los modelos tienen `deleted_at` para borrado lógico
- **Timestamps**: `created_at`, `updated_at` automáticos
- **Slugs**: URLs SEO-friendly en lugar de IDs numéricos
- **Ratings**: Un usuario solo puede tener un rating activo por curso

---

## API Endpoints

### Cursos
| Método | Endpoint | Descripción |
|--------|----------|-------------|
| `GET` | `/` | Mensaje de bienvenida |
| `GET` | `/health` | Health check + estado BD + conteo cursos |
| `GET` | `/courses` | Lista todos los cursos con ratings |
| `GET` | `/courses/{slug}` | Detalle completo (teachers, lessons, ratings) |
| `GET` | `/classes/{class_id}` | Detalle de una clase/lección |

### Ratings
| Método | Endpoint | Descripción |
|--------|----------|-------------|
| `POST` | `/courses/{id}/ratings` | Crear o actualizar rating (upsert) |
| `GET` | `/courses/{id}/ratings` | Todos los ratings de un curso |
| `GET` | `/courses/{id}/ratings/stats` | Estadísticas (promedio, distribución) |
| `GET` | `/courses/{id}/ratings/user/{user_id}` | Rating de un usuario específico |
| `PUT` | `/courses/{id}/ratings/{user_id}` | Actualizar rating existente |
| `DELETE` | `/courses/{id}/ratings/{user_id}` | Soft delete del rating |

### Documentación Interactiva
- **Swagger UI**: http://localhost:8000/docs
- **ReDoc**: http://localhost:8000/redoc

---

## Patrones de Arquitectura

### Backend - Service Layer Pattern
```
Endpoint (main.py)
    │
    ▼ Dependency Injection
CourseService (lógica de negocio)
    │
    ▼ SQLAlchemy ORM
Models → PostgreSQL
    │
    ▲ Pydantic Schemas
Validación de entrada/salida
```

**Archivos clave:**
- `app/main.py`: Todos los endpoints (~435 líneas)
- `app/services/course_service.py`: Lógica de negocio de cursos y ratings
- `app/models/`: Modelos SQLAlchemy (Course, Teacher, Lesson, CourseRating)
- `app/schemas/rating.py`: Validación Pydantic para ratings

### Frontend - Server Components + App Router
```
Page (Server Component)
    │
    ▼ fetch() con cache: "no-store"
API Backend
    │
    ▼ Props
Client Components (presentational)
```

**Archivos clave:**
- `src/app/page.tsx`: Home con grid de cursos
- `src/app/course/[slug]/page.tsx`: Detalle de curso dinámico
- `src/components/`: Course, CourseDetail, VideoPlayer, StarRating
- `src/services/ratingsApi.ts`: Cliente API para ratings
- `src/types/`: Definiciones TypeScript

### Mobile - Clean Architecture (MVVM)
```
┌─────────────────────────────────────────┐
│           PRESENTATION                  │
│  ViewModel (StateFlow/@Published)       │
│  UI (Compose/SwiftUI)                   │
└─────────────────┬───────────────────────┘
                  │
┌─────────────────▼───────────────────────┐
│              DOMAIN                     │
│  Repository (Interface/Protocol)        │
│  Models (Course, Teacher, etc.)         │
└─────────────────┬───────────────────────┘
                  │
┌─────────────────▼───────────────────────┐
│               DATA                      │
│  DTOs → Mappers → Remote Repository     │
│  Network (Retrofit/URLSession)          │
└─────────────────────────────────────────┘
```

**Archivos clave Android:**
- `presentation/courses/viewmodel/CourseListViewModel.kt`
- `domain/repositories/CourseRepository.kt` (interface)
- `data/repositories/RemoteCourseRepository.kt`
- `data/network/ApiService.kt` (Retrofit)

**Archivos clave iOS:**
- `Presentation/ViewModels/CourseListViewModel.swift`
- `Domain/Repositories/CourseRepositoryProtocol.swift`
- `Data/Repositories/RemoteCourseRepository.swift`
- `Services/NetworkManager.swift`

---

## Comandos de Desarrollo

### Backend (Docker obligatorio)
```bash
cd Backend

# Lifecycle
make start              # Iniciar Docker Compose (BD + API)
make stop               # Detener contenedores
make restart            # Reiniciar todo
make clean              # Eliminar contenedores y volúmenes

# Base de datos
make migrate            # Aplicar migraciones pendientes
make create-migration   # Crear nueva migración (auto-detecta cambios)
make seed               # Poblar datos de prueba
make seed-fresh         # Limpiar BD y repoblar

# Debugging
make logs               # Ver logs en tiempo real
make shell              # Shell en contenedor API

# Testing
docker-compose exec api uv run pytest
```

### Frontend
```bash
cd Frontend

yarn dev                # Desarrollo con Turbopack (http://localhost:3000)
yarn build              # Build de producción
yarn start              # Iniciar build de producción
yarn test               # Ejecutar tests (Vitest)
yarn lint               # Linter ESLint
```

### Mobile
```bash
# Android
# Abrir en Android Studio → Run 'app'
# O: ./gradlew assembleDebug

# iOS
# Abrir PlatziFlixiOS.xcodeproj en Xcode → Run
# O: xcodebuild -scheme PlatziFlixiOS
```

---

## Configuración de Base de Datos

### Credenciales Docker
```
Host: localhost (o db desde contenedor)
Puerto: 5432
Usuario: platziflix_user
Password: platziflix_password
Database: platziflix_db
```

### Connection String
```
postgresql://platziflix_user:platziflix_password@db:5432/platziflix_db
```

### Migraciones
- **Ubicación**: `Backend/app/alembic/versions/`
- **Migraciones existentes**:
  1. `d18a08253457_`: Esquema inicial (courses, teachers, lessons, course_teachers)
  2. `0e3a8766f785_`: Tabla course_ratings

---

## URLs del Sistema

| Servicio | URL | Descripción |
|----------|-----|-------------|
| Backend API | http://localhost:8000 | API REST |
| Swagger Docs | http://localhost:8000/docs | Documentación interactiva |
| Frontend Web | http://localhost:3000 | Aplicación Next.js |
| PostgreSQL | localhost:5432 | Base de datos |

### URLs para Mobile (Emuladores)
- **Android Emulator**: `http://10.0.2.2:8000` (alias de localhost)
- **iOS Simulator**: `http://localhost:8000`

---

## Convenciones de Código

### Naming
| Contexto | Convención | Ejemplo |
|----------|------------|---------|
| Python | snake_case | `course_service.py`, `get_all_courses()` |
| TypeScript | camelCase (vars), PascalCase (components) | `courseList`, `CourseDetail.tsx` |
| Kotlin | camelCase (vars), PascalCase (clases) | `courseList`, `CourseViewModel` |
| Swift | camelCase (vars), PascalCase (types) | `courseList`, `CourseViewModel` |
| SQL/Models | snake_case | `course_id`, `created_at` |
| URLs/Slugs | kebab-case | `curso-de-python` |

### Estructura de Componentes (Frontend)
```
ComponentName/
├── ComponentName.tsx          # Implementación
├── ComponentName.module.scss  # Estilos CSS Modules
└── ComponentName.test.tsx     # Tests (o __tests__/)
```

### Testing
- **Backend**: pytest con httpx para tests de API
- **Frontend**: Vitest + React Testing Library
- **Mobile**: JUnit (Android), XCTest (iOS)

---

## Funcionalidades Implementadas

### Core
- [x] Catálogo de cursos con grid estilo Netflix
- [x] Detalle de cursos (profesores, lecciones)
- [x] Navegación por slug SEO-friendly
- [x] Reproductor de video HTML5 integrado
- [x] Health checks de API y BD

### Sistema de Ratings
- [x] Crear/actualizar ratings (1-5 estrellas)
- [x] Estadísticas agregadas (promedio, distribución)
- [x] Rating por usuario
- [x] Soft delete de ratings
- [x] Componente StarRating con accesibilidad

### Mobile
- [x] Apps nativas Android e iOS
- [x] Arquitectura Clean Architecture
- [x] Consumo de API REST
- [x] Manejo de estados (loading, error, success)

---

## Consideraciones de Desarrollo

### Reglas Generales
1. **Docker obligatorio** para el backend - siempre verificar que esté corriendo
2. **TypeScript strict** habilitado en Frontend - no usar `any`
3. **Testing requerido** para nuevas funcionalidades
4. **Migraciones** para cualquier cambio de esquema de BD
5. **API REST** es la única fuente de datos para todos los clientes

### Backend
- Ejecutar comandos dentro del contenedor Docker (usar `make`)
- Usar `CourseService` para lógica de negocio, no en endpoints directamente
- Soft deletes: usar `deleted_at` en lugar de DELETE físico
- Validar con Pydantic schemas antes de procesar

### Frontend
- Preferir Server Components para data fetching
- Usar `cache: "no-store"` para datos dinámicos
- CSS Modules para estilos con scope
- Variables SCSS en `src/styles/vars.scss`

### Mobile
- Mapear DTOs a modelos de dominio (nunca usar DTOs en UI)
- ViewModels manejan estado y lógica de presentación
- Repository pattern para abstracción de datos
- Android: `10.0.2.2` para localhost desde emulador

---

## Troubleshooting

### Backend no inicia
```bash
# Verificar que Docker esté corriendo
docker ps

# Reiniciar contenedores
cd Backend && make clean && make start

# Ver logs de error
make logs
```

### Error de conexión a BD
```bash
# Verificar que PostgreSQL esté corriendo
docker-compose ps

# Reconectar
make stop && make start
```

### Migraciones fallidas
```bash
# Ver estado actual
docker-compose exec api alembic current

# Aplicar desde cero (CUIDADO: borra datos)
make clean && make start && make migrate && make seed
```

### Frontend no conecta al backend
```bash
# Verificar que backend esté corriendo
curl http://localhost:8000/health

# Verificar CORS si hay errores
# Backend permite todos los origins en desarrollo
```

---

## Próximos Pasos Sugeridos

### Pendientes
- [ ] Autenticación de usuarios (JWT)
- [ ] Sistema de progreso de cursos
- [ ] Favoritos/Watchlist
- [ ] Búsqueda de cursos
- [ ] Paginación en listados
- [ ] Cache de API responses
- [ ] CI/CD pipeline

### Mejoras Técnicas
- [ ] Consolidar modelo Class/Lesson (duplicado)
- [ ] Agregar tabla User para integridad referencial
- [ ] Implementar rate limiting
- [ ] Agregar logging estructurado
- [ ] Dockerizar Frontend para producción

---

## Referencias

### Documentación Oficial
- [FastAPI](https://fastapi.tiangolo.com/)
- [SQLAlchemy 2.0](https://docs.sqlalchemy.org/)
- [Next.js 15](https://nextjs.org/docs)
- [Jetpack Compose](https://developer.android.com/jetpack/compose)
- [SwiftUI](https://developer.apple.com/xcode/swiftui/)

### Archivos de Especificación
- `spec/00_sistema_ratings_cursos.md` - Spec del sistema de ratings
- `spec/01_backend_ratings_implementation_plan.md` - Plan de implementación
- `spec/02_frontend_ratings_implementation_plan.md` - Plan frontend
- `spec/03_backend_security_review.md` - Review de seguridad

---

*Esta memoria se actualiza conforme evoluciona el proyecto. Última actualización basada en análisis completo de arquitectura.*
