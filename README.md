<p align="center">
  <img alt="logo" src="https://img.shields.io/badge/TFI%20P2-Historia%20Cl%C3%ADnica-0A66C2?style=for-the-badge" />
</p>

<h1 align="center">Trabajo Final Integrador Programación 2 – Historia Clínica</h1>

<p align="center">
  Aplicación Java con relación <b>1→1 unidireccional</b> (Paciente A → HistoriaClinica B),
  DAO + Service, MySQL 8, Gradle, y baja lógica 🗂️
</p>

<p align="center">
  <!-- Java -->
  <img alt="Java" src="https://img.shields.io/badge/Java-21-007396?style=flat-square&logo=openjdk&logoColor=white" />
  <!-- Gradle -->
  <img alt="Gradle" src="https://img.shields.io/badge/Gradle-8.x-02303A?style=flat-square&logo=gradle&logoColor=white" />
  <!-- MySQL -->
  <img alt="MySQL" src="https://img.shields.io/badge/MySQL-8.x-4479A1?style=flat-square&logo=mysql&logoColor=white" />
  <!-- License -->
  <a href="LICENSE">
    <img alt="License" src="https://img.shields.io/badge/License-MIT-green?style=flat-square" />
  </a>
</p>

Repositorio de la entrega integradora de **Programación 2**. Implementa una relación **1→1 unidireccional** de `Paciente` (A) hacia `HistoriaClinica` (B), con persistencia en **MySQL 8**, acceso vía **DAO**, orquestación en **Service** (transacciones), **baja lógica** por campo `eliminado` y scripts SQL de creación/seed/validación.

---
> **Cátedra:** Programación II  
> **Docente:** GIULIANO ESPEJO  
> **Año/Cuat.**: 2025 / 2C

## 👥 Integrantes

| Nombre              | usuario github |
|---------------------|----------------------|
| Luis Cisneros       | [@luiscisneros356](https://github.com/luiscisneros356) |
| Nicolás Colman      | [@ncolman94](https://github.com/ncolman94) |
| Santiago Caiciia Massello| [@scaiciia](https://github.com/scaiciia) |
| Guillermo Campoy    | [@guillecampoy](https://github.com/guillecampoy) |

---

## Índice
- [📝 Contexto y Alcance](#-contexto-y-alcance)
- [🛠️ Tech Stack](#-tech-stack)
- [📐 Estrategia de Modelado y Borrado](#-estrategia-de-modelado-y-borrado)
- [📦 Estructura](#-estructura)
- [🧪 Requisitos Técnicos](#-requisitos-técnicos)
- [🗄️ Base de datos](#-base-de-datos)
- [🚀 Ejecutar](#-ejecutar)
- [✅ Validaciones](#-validaciones)
- [🧰 Troubleshooting](#-troubleshooting)
- [📚 Documentación y Entregables](#-documentación-y-entregables)
- [📄 Licencia](#-licencia)

---

## 📝 Contexto y Alcance

- Dominio: **Historia Clínica** con entidades principales:
    - **Paciente (A)**: `id`, `dni` (único), `nombre`, `apellido`, `fecha_nacimiento`, `eliminado`.
    - **HistoriaClinica (B)**: `id`, `nro_historia` (único), `grupo_sanguineo` (ENUM), campos de texto (`antecedentes`, `medicacion_actual`, `observaciones`), `eliminado`, y **FK única** `paciente_id`.
- Relación: **1→1 unidireccional A→B**, implementada con **FK única en B** hacia A.
- Operaciones mínimas: CRUD de A y B (con **eliminación lógica**), y **búsqueda por DNI** en el menú de la app.
- Objetivo: demostrar diseño correcto, separación por capas, persistencia limpia, y consistencia de datos.

---
## 🛠️ Tech Stack

- **Java 21** · **Gradle 8.x** · JDBC
- **MySQL 8** (charset `utf8mb4`)
- Patrón **DAO** + capa **Service** (transacciones, orquestación)
- **Baja lógica** por campo `eliminado` en A y B
---
## 📐 Estrategia de Modelado y Borrado

- **Clave foránea única en B (`historia_clinica.paciente_id`)** para garantizar 1→1.
- **Unicidades**: `paciente.dni`, `historia_clinica.nro_historia` y `historia_clinica.paciente_id`.
- **Baja lógica** (uso normal de la app): marcar `eliminado = TRUE` en A y B.
- **Borrado físico con `ON DELETE CASCADE`** (uso excepcional): definido en la FK de B solo para **limpiezas/depuración controlada** (no en la operatoria cotidiana).

---

## 📦 Estructura de Proyecto

```
tpi-p2-historia-clinica/
├─ README.md
├─ docs/
│  ├─ uml/
│  │  └─ TBD.png  # Diagramas UML necesarios
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
---

## 🧪️ Requisitos Técnicos

- JDK **21** (recomendado).
- **MySQL 8.x**.
- **Gradle**
- Cliente `mysql` (CLI) o herramienta GUI equivalente.

---

## 🗄️ Base de Datos: Creación y Datos

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

## 🚀 Ejecución de la App
WIP / TBD
1. Configurar `src/main/resources/application.properties`:
   ```properties
   db.url=jdbc:mysql://localhost:3306/tpi_historia_clinica?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC
   db.user=root
   db.password=tu_password
   ```
2. Construcción y ejecución:
    - **Gradle**: `./gradlew run`

Otras recomendaciones
> TBD.

---

## ✅ Consultas de Validación

Incluidas en `sql/03_validate.sql` (porción):

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
## 🧰 Resolución de Problemas Comunes

- **Error de zona horaria**: agregar `serverTimezone=UTC` en la URL JDBC.
- **Handshake/SSL**: `allowPublicKeyRetrieval=true&useSSL=false` en desarrollo local.
- **Conector MySQL**: asegurarse de usar `jdbc:mysql://...`, *no* una URL HTTP.
- **Permisos**: usuario con privilegios para `CREATE DATABASE` y `FOREIGN KEY`.
- **UNICODE**: la base usa `utf8mb4`/`utf8mb4_unicode_ci`.

---

## 📚 Documentación y Entregables

- **UML** de clases en `docs/uml/`.
- **Informe** en `docs/informe/`.
- **Video** mostrando CRUD, relación 1→1, búsqueda por DNI, baja lógica y un rollback.
- **SQL** completos en `sql/`.

---

## 📄 Licencia
MIT