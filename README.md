# TPI P2 – Historia Clínica (Paciente → HistoriaClinica)

Repositorio de la entrega integradora de **Programación 2**. Implementa una relación **1→1 unidireccional** de `Paciente` (A) hacia `HistoriaClinica` (B), con persistencia en **MySQL 8**, acceso vía **DAO**, orquestación en **Service** (transacciones), **baja lógica** por campo `eliminado` y scripts SQL de creación/seed/validación.


---

## Tabla de contenidos
- [Contexto y Alcance](#contexto-y-alcance)
- [Estrategia de Modelado y Borrado](#estrategia-de-modelado-y-borrado)
- [Estructura de Proyecto](#estructura-de-proyecto)
- [Requisitos Técnicos](#requisitos-técnicos)
- [Base de Datos: Creación y Datos](#base-de-datos-creación-y-datos)
- [Ejecución de la App](#ejecución-de-la-app)
- [Consultas de Validación](#consultas-de-validación)
- [Checklist de Entregables](#checklist-de-entregables)
- [Resolución de Problemas Comunes](#resolución-de-problemas-comunes)
- [Licencia](#licencia)

---

## Contexto y Alcance

- Dominio: **Historia Clínica** con entidades principales:
    - **Paciente (A)**: `id`, `dni` (único), `nombre`, `apellido`, `fecha_nacimiento`, `eliminado`.
    - **HistoriaClinica (B)**: `id`, `nro_historia` (único), `grupo_sanguineo` (ENUM), campos de texto (`antecedentes`, `medicacion_actual`, `observaciones`), `eliminado`, y **FK única** `paciente_id`.
- Relación: **1→1 unidireccional A→B**, implementada con **FK única en B** hacia A.
- Operaciones mínimas: CRUD de A y B (con **eliminación lógica**), y **búsqueda por DNI** en el menú de la app.
- Objetivo: demostrar diseño correcto, separación por capas, persistencia limpia, y consistencia de datos.

---

## Estrategia de Modelado y Borrado

- **Clave foránea única en B (`historia_clinica.paciente_id`)** para garantizar 1→1.
- **Unicidades**: `paciente.dni`, `historia_clinica.nro_historia` y `historia_clinica.paciente_id`.
- **Baja lógica** (uso normal de la app): marcar `eliminado = TRUE` en A y B.
- **Borrado físico con `ON DELETE CASCADE`** (uso excepcional): definido en la FK de B solo para **limpiezas/depuración controlada** (no en la operatoria cotidiana).

---

## Estructura de Proyecto

```
tpi-p2-historia-clinica/
├─ README.md
├─ docs/
│  ├─ uml/
│  │  └─ diagrama-clases.png
│  └─ informe/
│     └─ informe.pdf
├─ sql/
│  ├─ 01_create_schema.sql
│  ├─ 02_seed.sql
│  └─ 03_validate.sql
├─ scripts/
│  └─ run-local.sh                 # Agregar cuando se tengan los mismos, creacion de tablas, creacion de registros de pruebas y validaciones minimas
├─ src/
│  └─ main/
│     ├─ java/
│     │  ├─ config/
│     │  │  └─ DatabaseConnection.java
│     │  ├─ entities/
│     │  │  ├─ Paciente.java
│     │  │  └─ HistoriaClinica.java
│     │  ├─ dao/
│     │  │  ├─ GenericDao.java
│     │  │  ├─ PacienteDao.java
│     │  │  └─ HistoriaClinicaDao.java
│     │  ├─ service/
│     │  │  ├─ GenericService.java
│     │  │  ├─ PacienteService.java
│     │  │  └─ HistoriaClinicaService.java
│     │  └─ main/
│     │     ├─ AppMenu.java
│     │     └─ Main.java
│     └─ resources/
│        └─ application.properties  # url, user, pass de MySQL
└─ build.gradle
```

**Notas**:
- El menú debe exponer CRUD y búsqueda por DNI, respetando baja lógica.
- La capa `service/` orquesta transacciones y protege la invariante 1→1.
- En `resources/` parametrizá conexión a MySQL.

---

## Requisitos Técnicos

- JDK **21** (recomendado).
- **MySQL 8.x**.
- **Gradle**
- Cliente `mysql` (CLI) o herramienta GUI equivalente.

---

## Base de Datos: Creación y Datos

Scripts incluidos en `./sql`:

1. **01_create_schema.sql** – Crea base `tpi_historia_clinica`, tablas, índices y FK 1→1 con `ON DELETE CASCADE`.
2. **02_seed.sql** – Inserta datos de prueba (3 pacientes y 3 historias clínicas asociadas).
3. **03_validate.sql** – Consultas para verificar unicidades y la relación 1→1.

### Ejecución rápida por CLI

```bash
mysql -u root -p < sql/01_create_schema.sql
mysql -u root -p < sql/02_seed.sql
mysql -u root -p < sql/03_validate.sql
```

> (WIP) Observación adicional: Si se utiliza `scripts/run-local.sh`, asegurarse de `chmod +x scripts/run-local.sh` y adaptar usuario/host/puerto si fuera necesario.

---

## Ejecución de la App
WIP
1. Configurá `src/main/resources/application.properties`:
   ```properties
   db.url=jdbc:mysql://localhost:3306/tpi_historia_clinica?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC
   db.user=root
   db.password=tu_password
   ```
2. Construí y ejecutá:
    - **Gradle**: `./gradlew run`

WIP
> Sugerencia: usar try-with-resources y `PreparedStatement` en DAO, commit/rollback en `Service` para operaciones compuestas (p. ej., crear Paciente + Historia).

---

## Consultas de Validación

Incluidas en `sql/03_validate.sql` (extracto):

```sql
-- Pacientes activos con su historia clínica
SELECT
  p.id AS paciente_id, p.apellido, p.nombre, p.dni, p.fecha_nacimiento,
  h.id AS historia_id, h.nro_historia, h.grupo_sanguineo
FROM paciente p
JOIN historia_clinica h ON h.paciente_id = p.id
WHERE p.eliminado = FALSE AND h.eliminado = FALSE
ORDER BY p.apellido, p.nombre;

-- Búsqueda por DNI
SELECT p.*, h.nro_historia, h.grupo_sanguineo
FROM paciente p
LEFT JOIN historia_clinica h ON h.paciente_id = p.id
WHERE p.dni = '30111222';

-- (Prueba) Intentar 2da historia para el mismo paciente (debe fallar por UNIQUE)
-- INSERT INTO historia_clinica (eliminado, nro_historia, grupo_sanguineo, paciente_id)
-- VALUES (FALSE, 'HC-0001B', 'O+', 1);
```

(WIP)
## Resolución de Problemas Comunes

- **Error de zona horaria**: agregá `serverTimezone=UTC` en la URL JDBC.
- **Handshake/SSL**: `allowPublicKeyRetrieval=true&useSSL=false` en desarrollo local.
- **Conector MySQL**: asegurate de usar `jdbc:mysql://...`, *no* una URL HTTP.
- **Permisos**: usuario con privilegios para `CREATE DATABASE` y `FOREIGN KEY`.
- **UNICODE**: la base usa `utf8mb4`/`utf8mb4_unicode_ci` (emojis/caracteres extendidos).

---

## Licencia

MIT