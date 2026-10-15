# Documento de Propuesta: Gestor de Boletas y Colectivos

Este documento detalla la visión, el modelo de negocio y la arquitectura técnica planificada para el desarrollo de una plataforma de gestión de boletas, gastos operativos y transporte compartido en rutas predefinidas. Está diseñado para alinear al equipo técnico y estratégico antes de comenzar la fase de desarrollo.

---

## 1. El Problema Actual y la Oportunidad de Negocio

### El Contexto

Actualmente, el control de gastos personales y la gestión de comprobantes físicos es un dolor de cabeza para la mayoría de las personas y operadores del transporte. Existen tres problemas críticos:

1. Pérdida de dinero y garantías: las boletas impresas en papel térmico se borran con el tiempo o se extravían. Cuando un equipo falla, el usuario pierde su derecho a garantía por no tener el comprobante legible.
2. Desorden financiero: muchas personas no saben en qué gastan su sueldo. Llevar un Excel manual es tedioso y termina siendo abandonado.
3. Falta de trazabilidad: no existe un lugar unificado y seguro donde guardar comprobantes importantes con validez en el tiempo.

En el caso del transporte compartido, también existe un problema operativo: los pasajeros no tienen un registro claro de lo que pagan, y los conductores no llevan un control estricto de los gastos de operación ni una forma precisa de calcular tarifas cuando se sube en mitad del recorrido.

### La Solución y Visión SaaS

Crearemos una aplicación web y móvil donde el usuario pueda registrar sus boletas y gastos, adjuntando la fotografía del comprobante original. El sistema ordenará estos gastos en un calendario, generará estadísticas y, lo más importante, enviará un respaldo automático al correo del usuario, actuando como una "caja fuerte digital".

Para el ámbito de transporte, se incluirá un módulo de rutas predefinidas y cálculo dinámico de tarifa por tramo. El sistema permitirá que pasajeros paguen viajes de forma transparente y que conductores registren gastos de combustible, peajes y mantenimientos. La plataforma podrá operar como SaaS con un modelo de administración centralizada para auditoría y control.

---

## 2. Definición de Usuarios y Roles

El sistema contará con roles bien definidos para mantener un flujo claro y seguro:

- Usuario Estándar / Pasajero: puede registrarse, iniciar sesión, pagar viajes, revisar su historial, y obtener boletas o comprobantes.
- Conductor: registra sus rutas y gastos, calcula tarifas dinámicas según tramo, y sube comprobantes de operación.
- Administrador: rol interno con acceso completo al panel de control, validación de conductores, aprobación de documentos y revisión de auditoría.

La idea es mantener un flujo simple para la parte pública del sistema, mientras el administrador central mantiene el control de seguridad, validación y trazabilidad.

---

## 3. Funcionalidades Principales del Sistema

### A. Registro de Boletas y Sello de Seguridad

El usuario podrá ingresar los datos de su compra (monto, fecha, local, categoría) y subir la foto de la boleta. Para garantizar que esta imagen y sus datos sean confiables en el futuro, el sistema aplicará un proceso de hash y trazabilidad, de manera que si alguien intenta alterar el documento, el sistema pueda detectar la incoherencia.

### B. Calendario Interactivo

Toda boleta ingresada se posicionará visualmente en un calendario por fecha de emisión. Esto permite al usuario navegar por meses y días específicos para encontrar gastos y recordar compras importantes.

### C. Sistema de Respaldo por Correo

Tras subir una boleta o registrar un gasto, el sistema enviará automáticamente un correo electrónico con el resumen y el comprobante adjunto. Así, aunque la aplicación deje de existir o el usuario cambie de teléfono, sus información y documentos seguirán resguardados.

### D. Dashboard de Tendencias y Estadísticas

El sistema agrupará datos para mostrar patrones de gasto por categoría, mes o local. Esto da al usuario una visión clara de su consumo y permite detectar diferencias con meses anteriores.

### E. Cálculo de Tarifa Dinámica en Rutas Predefinidas

Las rutas se encuentran definidas previamente. Cuando un pasajero sube a mitad de recorrido, el sistema calcula el importe proporcional según el tramo recorrido. Esto evita cobros arbitrarios y mejora la transparencia del servicio.

### F. Gestión de Gastos Operativos del Conductor

El conductor podrá registrar boletas de combustible, peajes, mantención y otros gastos operativos. La plataforma centraliza estos comprobantes para facilitar el control financiero y la revisión del administrador.

### G. Auditoría y Seguridad

Toda acción importante queda registrada con datos de usuario, fecha, operación y resultados. Esto incluye ingresos, aprobación o rechazo de conductores, pagos, creación de viajes, carga de comprobantes y eventos de seguridad.

---

## 4. Arquitectura y Stack Tecnológico

### Backend: Django (Python)

Elegimos Django por ser un framework maduro, seguro y rápido para construir plataformas SaaS. Nos proporciona herramientas listas para usar como autenticación, administración, modelos y reglas de negocio.

### Base de Datos: Enfoque Híbrido con PostgreSQL

Las boletas y los registros de operación pueden ser complejos. Algunas solo tienen un total, otras tienen múltiples productos, descuentos e impuestos. Por eso, usamos PostgreSQL de forma híbrida:

- Parte relacional: para datos estructurados (usuarios, rutas, roles, pagos, boletas básicas).
- Parte no relacional (JSONB): para detalles dinámicos que pueden cambiar con el tiempo, como el contenido de una boleta, tramo recorrido o datos específicos del viaje.

### Infraestructura en la Nube

La plataforma se diseñará para operar en la nube, con apoyo de servicios como:

- Amazon S3: almacenamiento seguro de fotos y comprobantes.
- Amazon RDS: base de datos PostgreSQL con respaldos automáticos.
- EC2 o Elastic Beanstalk: para desplegar Django.
- Amazon SES: para correos de respaldo y notificaciones.

---

## 5. Modelo de negocio y propuesta de valor

El sistema busca resolver dos frentes al mismo tiempo:

1. Para pasajeros: mejor control de pagos, historial de viajes y boletas confiables.
2. Para conductores: seguimiento de gastos operativos, cálculo justo de tarifas y evidencia documental para la gestión.

Con una capa de auditoría y validación, la plataforma mejora la transparencia y reduce la pérdida de dinero por errores manuales o falta de control documental.

---

## 6. Diagramas de Flujo Conceptuales

### Flujo de Ingreso y Respaldo

```text
1. El usuario inicia sesión en la plataforma.
2. Selecciona "Nueva Boleta" o "Nuevo Viaje" y sube la evidencia desde su celular.
3. Rellena los datos básicos (fecha, monto, categoría, ruta, tramo, etc.).
4. El servidor Django recibe la información:
   ├─ Guarda la foto o comprobante de forma segura en Amazon S3.
   ├─ Genera el hash o trazabilidad necesaria para verificar integridad.
   └─ Guarda los datos híbridos en PostgreSQL.
5. El sistema envía el correo de respaldo o notificación correspondiente.
6. El usuario ve la información reflejada en su calendario, historial y estadísticas.
```

### Flujo de transporte y cálculo de tarifa

```text
1. El pasajero ingresa a la ruta predefinida.
2. El sistema identifica el punto de subida.
3. Se calcula el tramo recorrido y el porcentaje de tarifa aplicable.
4. Se genera la boleta con el valor exacto.
5. El conductor recibe el registro y el pasajero guarda el comprobante.
```

---

## 7. Objetivos del Proyecto

- Centralizar pagos y comprobantes del transporte compartido.
- Mejorar la trazabilidad y la transparencia para pasajeros y conductores.
- Reducir pérdidas por falta de control de gastos operativos.
- Crear una base sólida en Django + PostgreSQL para crecimiento futuro.
- Mantener una arquitectura preparada para extensiones de IA, reportes avanzados y escalamiento SaaS.

---

## 8. Siguiente Etapa

La siguiente fase será el diseño técnico detallado:

1. Definición final de modelos y tablas.
2. Implementación del custom user model para roles y autenticación.
3. Diseño de rutas, tarifas dinámicas y módulo de auditoría.
4. Migración inicial de PostgreSQL y estructura de JSONB.
5. Desarrollo de la primera versión funcional del backend con Django.

Este repositorio será el punto de partida para documentar, versionar y estructurar todo el proyecto de forma clara y mantenible.
