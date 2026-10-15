# Estructura Base de Datos

Este documento detalla la visión de la base de datos del sistema de transporte compartido y gestión de boletas. La propuesta está orientada a un esquema híbrido con PostgreSQL, combinando tablas relacionales y campos JSONB para flexibilidad técnica.

---

## 1. El Problema Actual y la Solución

El transporte en rutas predefinidas (como colectivos o furgones) y la gestión de sus comprobantes sufre de informalidad. Los pasajeros no tienen un registro claro de lo que gastan, y los conductores pierden dinero al no llevar un control estricto de sus gastos operativos (bencina, peajes) ni tener un cálculo exacto de la tarifa cuando un pasajero sube a mitad de camino.

Nuestra solución es una plataforma híbrida donde los pasajeros registran y pagan sus viajes de manera transparente, y los conductores cobran tarifas dinámicas y suben las boletas de sus gastos operativos. Todo esto queda vigilado por un sistema de auditoría estricta gestionado por un administrador preexistente.

---

## 2. Definición de Usuarios y Roles

El sistema restringe el registro público a solo dos roles operativos. El rol de administrador es interno y se crea directamente en la base de datos mediante la consola de Django.

### A. Administrador (Superusuario - No registrable públicamente)
- Ya existe en el sistema por defecto.
- Funciones: panel de control total, revisión y aprobación de documentos, visualización de perfiles, acceso exclusivo al registro de auditoría.

### B. Pasajero (User)
- Se registra mediante la aplicación.
- Requisitos de perfil: nombre, correo, contraseña, teléfono y foto de perfil.
- Seguridad: autenticación de dos factores (2FA), contraseñas encriptadas.
- Funciones: paga pasajes, revisa historial y obtiene boletas/comprobantes.

### C. Conductor (Colaborador)
- Se registra mediante la aplicación, pero entra en estado "Pendiente de Revisión" hasta que el administrador valide sus datos.
- Requisitos base: mismos que el pasajero.
- Requisitos extra: licencia, fecha de vencimiento, datos del vehículo, permiso de circulación y fotos del vehículo.
- Funciones: validar viajes, calcular tarifas dinámicas y subir boletas de gastos operativos.

---

## 3. Funcionalidades Principales

### 1. Cálculo de Tarifa Dinámica en Rutas Predefinidas

Las rutas están preestablecidas. Si un pasajero sube en el punto A, se cobra la tarifa base completa. Sin embargo, el sistema incluye un motor de cálculo para fraccionar el cobro si sube en un punto intermedio. El monto se calcula según la distancia o tramo restante y se emite la boleta por el valor exacto.

### 2. Gestión Dual de Boletas (JSON Híbrido)

El sistema guarda dos tipos de boletas:
- Pasajes: comprobante del viaje, método de pago, patente del vehículo y tramo recorrido.
- Gastos operativos: boletas de bencina, peajes o repuestos.

Nota: para agilizar búsquedas por fechas y reportes mensuales, no se crea una tabla de fechas aislada; se utilizan índices nativos de PostgreSQL sobre el campo de fecha de la boleta.

### 3. Módulo de Seguridad y Auditoría

Toda acción en el sistema queda grabada en una tabla de auditoría intocable. El administrador puede usar esta tabla para rastrear cualquier anomalía.

---

## 4. Estructura de la Base de Datos (PostgreSQL + Django)

### Módulo de Autenticación y Perfiles

#### 1. Tabla: Usuarios_Base (Auth User)
Tabla central para el login. La contraseña se guarda como un hash irreversible.

Campos:
- id (UUID - PK)
- email (String - Unique)
- password_hash (String - cifrado por Django)
- rol (Enum: PASAJERO, CONDUCTOR, ADMIN)
- telefono (String)
- foto_perfil_url (String - S3 URL)
- is_2fa_enabled (Boolean)
- two_fa_secret (String - Encriptado)

#### 2. Tabla: Perfiles_Conductores
Se relaciona 1 a 1 con Usuarios_Base. Solo existe si el rol es CONDUCTOR.

Campos:
- usuario_id (FK -> Usuarios_Base)
- licencia_img_url (String)
- fecha_vencimiento_licencia (Date)
- estado_aprobacion (Enum: PENDIENTE, APROBADO, RECHAZADO)
- patente_vehiculo (String)
- permiso_circulacion_img (String)
- fotos_vehiculo (JSONB - Array de URLs)

### Módulo de Operaciones (Rutas y Boletas)

#### 3. Tabla: Rutas_Predefinidas
Campos:
- id (UUID - PK)
- nombre_ruta (String)
- tarifa_base_completa (Decimal)
- tramos (JSONB): contiene paraderos intermedios y porcentajes de cobro.

#### 4. Tabla: Boletas_y_Transacciones
Almacena tanto pasajes como recargas de bencina. Tiene un índice en fecha_emision para búsquedas ultra rápidas.

Campos:
- id (UUID - PK)
- tipo_transaccion (Enum: VIAJE_PASAJERO, GASTO_CONDUCTOR)
- usuario_emisor_id (FK -> Usuarios_Base)
- conductor_receptor_id (FK -> Usuarios_Base - Nullable)
- monto_total (Decimal)
- fecha_emision (DateTime - indexado)
- detalles_dinamicos (JSONB)
- comprobante_img_url (String)

Ejemplos de detalles_dinamicos:
- Viaje:
  {
    "ruta_id": 1,
    "subida": "Paradero 3",
    "bajada": "Terminal",
    "metodo_pago": "Tarjeta"
  }
- Gasto:
  {
    "tipo": "Bencina",
    "litros": 15,
    "estacion": "Copec"
  }

### Módulo de Trazabilidad

#### 5. Tabla: Auditoria_Seguridad
Registra cada movimiento del sistema. Solo es visible para el administrador.

Campos:
- id (UUID - PK)
- usuario_id (FK -> Usuarios_Base)
- accion (String)
- tabla_afectada (String)
- id_registro_afectado (String)
- datos_previos (JSONB)
- datos_nuevos (JSONB)
- ip_origen (String)
- fecha_hora (DateTime - indexado)

---

## 5. Próximos Pasos para el Desarrollo en Django

1. Configurar Custom User Model: sobreescribir AbstractUser para incluir el campo rol y usar email como identificador de login.
2. Lógica 2FA: implementar django-two-factor-auth.
3. Señales (Signals): utilizar post_save y post_delete para escribir automáticamente registros en Auditoria_Seguridad.
4. Definir serializers y endpoints para usuarios, viajes, gastos y auditoría.
5. Desarrollar migraciones iniciales y pruebas de integridad.

---

## 6. Recomendaciones de diseño

- Usar UUID como clave primaria para facilitar trazabilidad y evitar colisiones.
- Agregar índices sobre fecha_emision y campos de búsqueda frecuentes.
- Usar JSONB solo en datos heterogéneos o con evolución futura.
- Mantener auditoría inmutable por diseño, usando procesos de write-only para log de seguridad.

---

## 7. Objetivo final

La base de datos debe ser segura, trazable y flexible para soportar tanto el sistema de transporte compartido como la gestión de boletas y gastos operativos, sin perder velocidad ni claridad para reportes y análisis del negocio.
