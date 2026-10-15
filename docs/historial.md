# Historial de análisis y decisiones

Este archivo documenta el contexto, la evolución del proyecto y las decisiones clave tomadas durante la definición inicial del sistema.

---

## 2026-10-15 — Revisión inicial del proyecto
- Tema: Definición del alcance del sistema
- Resumen: Se consolidó la idea de una plataforma para gestión de boletas y transporte compartido con rutas predefinidas, pagos transparentes y auditoría para administradores.
- Resultado: Se definieron los roles de pasajero, conductor y administrador; además, se estableció la necesidad de controlar gastos operativos y cálculos dinámicos de tarifa por tramo.
- Archivos relacionados: README.md, docs/estructura_base_de_dato.md
- Estado: Aprobado
- Siguientes pasos: avanzar con la estructura del backend en Django y definir modelos base.



## 2026-10-15 — Diseño de la base de datos
- Tema: Modelo relacional y JSONB
- Resumen: Se planteó una base de datos híbrida con PostgreSQL, combinando tablas relacionales para usuarios, rutas y auditoría, y JSONB para datos dinámicos como detalles de viajes y gastos.
- Resultado: Se decidió mantener una tabla central `Boletas_y_Transacciones` para unificar pasajes y gastos operativos.
- Archivos relacionados: docs/estructura_base_de_dato.md
- Estado: Aprobado
- Siguientes pasos: definir las migraciones iniciales y pruebas de validación de integridad.

---
