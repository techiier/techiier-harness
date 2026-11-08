---
name: sql-guidelines
description: Contexto para interacción general con bases de datos relacionales (ej. SQL Server, PostgreSQL) y escritura o revisión de sentencias SQL.
---
# SQL Guidelines

## Contexto
Interacción general con bases de datos relacionales (ej. SQL Server, PostgreSQL) y escritura o revisión de sentencias SQL.

## Reglas Principales

### Seguridad
Tolerancia cero frente a vulnerabilidades de inyección SQL, haciendo obligatorio el uso de consultas parametrizadas.

### Contención de Daños
Prevención de operaciones masivas accidentales exigiendo una cláusula `WHERE` explícita en sentencias `UPDATE` y `DELETE`.

### Rendimiento Óptimo
- Evitar terminantemente consultas tipo `SELECT *`.
- Exigir el uso de cláusulas SARGables para aprovechar índices de manera correcta.
- Prevención del clásico problema de consultas N+1 en el uso de ORMs como Entity Framework Core o Dapper.