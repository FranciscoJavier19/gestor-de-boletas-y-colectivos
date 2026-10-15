# Historial de análisis y decisiones

Este archivo documenta el contexto, la evolución del proyecto y las decisiones clave tomadas durante la definición inicial del sistema.

---

## 2026-10-15 — Revisión inicial del proyecto
- Autor: FranciscoJavier19
- Tema: Definición del alcance del sistema
- Resumen: Se consolidó la idea de una plataforma para gestión de boletas y transporte compartido con rutas predefinidas, pagos transparentes y auditoría para administradores.
- Resultado: Se definieron los roles de pasajero, conductor y administrador; además, se estableció la necesidad de controlar gastos operativos y cálculos dinámicos de tarifa por tramo.
- Archivos relacionados: README.md, docs/estructura_base_de_dato.md
- Estado: Aprobado
- Siguientes pasos: avanzar con la estructura del backend en Django y definir modelos base.

---

## 2026-10-15 — Diseño de la base de datos
- Autor: FranciscoJavier19
- Tema: Modelo relacional y JSONB
- Resumen: Se planteó una base de datos híbrida con PostgreSQL, combinando tablas relacionales para usuarios, rutas y auditoría, y JSONB para datos dinámicos como detalles de viajes y gastos.
- Resultado: Se decidió mantener una tabla central `Boletas_y_Transacciones` para unificar pasajes y gastos operativos.
- Archivos relacionados: docs/estructura_base_de_dato.md
- Estado: Aprobado
- Siguientes pasos: definir las migraciones iniciales y pruebas de validación de integridad.

---

## 2026-10-15 — Decisión de seguridad y trazabilidad
- Autor: FranciscoJavier19
- Tema: Auditoría y autenticación
- Resumen: El sistema requiere control estrictamente auditable; por ello, cada acción importante debe quedar registrada en una tabla de auditoría con datos previos y nuevos.
- Resultado: Se incorporó la capa `Auditoria_Seguridad` y la necesidad de 2FA para pasajeros y conductores.
- Archivos relacionados: README.md, docs/estructura_base_de_dato.md
- Estado: Aprobado
- Siguientes pasos: definir señales de Django y políticas de acceso por rol.

---

## 2026-10-15 — Definición del repositorio como contexto de trabajo
- Autor: FranciscoJavier19
- Tema: Documentación y trazabilidad del proyecto
- Resumen: Se creó la estructura base del repositorio para que el contexto del proyecto quede visible y sea fácil de seguir por cualquier miembro del equipo.
- Resultado: Se documentan tanto la propuesta del proyecto como la estructura de la base de datos, con un historial inicial para registrar decisiones.
- Archivos relacionados: README.md, docs/estructura_base_de_dato.md, docs/historial.md
- Estado: Aprobado
- Siguientes pasos: comenzar con el backend, migraciones y primeros módulos de Django.
