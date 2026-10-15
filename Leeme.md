[propuesta_del_proyecto.md](https://github.com/user-attachments/files/33470580/propuesta_del_proyecto.md)
# Documento de Propuesta: Gestor y Respaldo Inteligente de Boletas

Este documento detalla la visión, el modelo de negocio y la arquitectura técnica planificada para el desarrollo de una plataforma de gestión de boletas y gastos. Está diseñado para alinear al equipo técnico y estratégico antes de comenzar la fase de desarrollo.

---

## 1. El Problema Actual y la Oportunidad de Negocio

### El Contexto

Actualmente, el control de gastos personales y la gestión de comprobantes físicos es un dolor de cabeza para la mayoría de los adultos. Las personas enfrentan tres problemas críticos:

1. **Pérdida de Dinero y Garantías:** Las boletas impresas en papel térmico se borran en pocos meses o se extravían. Cuando un electrodoméstico falla, el usuario pierde su derecho a garantía por no tener el comprobante legible.
2. **Desorden Financiero:** Muchas personas no saben en qué gastan su sueldo. Llevar un Excel manual es tedioso y termina siendo abandonado.
3. **Falta de Trazabilidad:** No existe un lugar unificado y seguro donde guardar comprobantes importantes con validez en el tiempo.

### La Solución y Visión SaaS (Software as a Service)

Crearemos una aplicación web/móvil donde el usuario pueda registrar sus boletas de forma rápida, adjuntando la fotografía del comprobante original. El sistema ordenará estos gastos en un calendario, generará estadísticas y, lo más importante, enviará un respaldo automático al correo del usuario, actuando como una "caja fuerte digital".

El modelo de negocio se proyecta como un **SaaS alojado en la nube (AWS)**. Se puede ofrecer un plan básico gratuito (límite de boletas mensuales) y un plan Premium (suscripción) que incluiría capacidades de Inteligencia Artificial para lectura automática, mayor almacenamiento y reportes avanzados.

---

## 2. Definición de Usuarios y Roles

El sistema está pensado para ser extremadamente simple en su operación, por lo que se limitará a dos roles principales:

- **Usuario Estándar:** Es el cliente final. Puede registrarse, iniciar sesión, subir sus boletas (fotografías y datos), ver su calendario de gastos, revisar estadísticas de su consumo mensual y recibir sus respaldos por correo.

- **Colaborador:** Colaborador sera asignado al conductor teniendo sus propios permisos de visualizacion de la appv

- **Administrador Técnico:** Un rol interno para nosotros (los creadores). No accede a los datos sensibles de los usuarios, pero tiene un panel para gestionar problemas técnicos, monitorear el estado de los servidores, revisar si hay correos que fallaron al enviarse y administrar las cuentas (bloqueos, restablecimientos).

---

## 3. Funcionalidades Principales del Sistema

### A. Registro de Boletas y Sello de Seguridad

El usuario podrá ingresar los datos de su compra (monto, fecha, local, categoría) y subir la foto de la boleta. Para garantizar que esta imagen y sus datos sean confiables en el futuro (por ej emplo, ante un reclamo en una tienda), el sistema aplicará un proceso de **Hashing**.

- _¿Qué es el Hashing aquí?_ Es una firma digital matemática. Tomaremos la imagen y los datos ingresados y los convertiremos en un código único. Si alguien intenta alterar la foto después, el código no coincidirá, garantizando la integridad del respaldo.

### B. Calendario Interactivo

Toda boleta ingresada se posicionará visualmente en un calendario basado en su fecha de emisión. Esto permite al usuario navegar por meses y días específicos para encontrar gastos, además de servir como un recordatorio visual de cuándo se realizaron las compras importantes.

**AT: Sobrecargar el sistema de interfaces pesadas aumenta el margen sobrecarga del sistema mantengamoslo simple formato fichero**

### C. Sistema de Respaldo por Correo (Emailing)

Una de las funciones más atractivas para el usuario. Tras subir una boleta, el sistema enviará automáticamente un correo electrónico con el resumen del gasto y la imagen adjunta. Esto da tranquilidad: aunque la aplicación deje de existir o el usuario cambie de teléfono, sus boletas siempre estarán a salvo en su propia bandeja de entrada.

### D. Dashboard de Tendencias y Estadísticas

**AT:Para que llenar de info innecesaria al cliente de un pasaje que cuesta menos de dos dolares **

El sistema agrupará los datos para mostrarle al usuario en qué se va su dinero. Mediante gráficos simples, el usuario podrá ver comparativas de gastos (ej. "Este mes gastaste un 20% más en alimentación que el mes pasado") y clasificaciones por categorías.

### E. Integración de Inteligencia Artificial (Visión a Futuro)

Aunque en la primera etapa el ingreso de datos será manual, la arquitectura estará preparada para integrar un motor de IA (como reconocimiento óptico de caracteres - OCR). En el futuro, el usuario solo tendrá que tomar la foto, y la IA leerá automáticamente el nombre del local, la fecha, el total y los impuestos, rellenando el formulario por él.

---

## 4. Arquitectura y Stack Tecnológico

Para que el proyecto sea robusto, escalable y mantenible, se utilizará la siguiente pila tecnológica:

### Backend: Django (Python)

Elegimos Django porque es un framework maduro, extremadamente seguro y rápido para construir plataformas SaaS. Nos proporciona herramientas listas para usar como autenticación de usuarios, paneles de administración (ideal para el rol de Admin Técnico) y un excelente manejo de bases de datos.

### Base de Datos: Enfoque Híbrido con PostgreSQL

Las boletas son documentos complejos. Algunas solo tienen un total, otras tienen 50 productos, propinas, descuentos e impuestos distintos. Usar una base de datos puramente relacional (tablas cuadradas y estrictas) nos limitaría.

**La Solución:** Usaremos **PostgreSQL** de forma híbrida.

- **Parte Relacional:** Para datos estructurados (Usuarios, Fechas, Categorías, Totales). Esto nos permite buscar y filtrar rapidísimo en el calendario.
- **Parte NoSQL (JSON):** Guardaremos el detalle específico de la boleta y sus metadatos en formato de texto plano estructurado (JSON) dentro de la misma base de datos. Si mañana decidimos que queremos empezar a guardar el "número de caja" o la "dirección de la sucursal", simplemente lo añadimos al JSON sin tener que desarmar ni migrar toda la base de datos.

### Infraestructura en la Nube: Amazon Web Services (AWS)

Para soportar el modelo SaaS y dar un servicio profesional ininterrumpido, alojaremos la plataforma en AWS:

- **Amazon S3:** Para almacenar de forma barata, duradera y segura todas las fotografías de las boletas.
- **Amazon RDS:** Para alojar nuestra base de datos PostgreSQL con respaldos automáticos.
- **EC2 o Elastic Beanstalk:** Para correr nuestro servidor Django.
- **Amazon SES (Simple Email Service):** Para garantizar que los miles de correos de respaldo lleguen a la bandeja de entrada de los usuarios y no a la carpeta de Spam.(Opcional)

---

## 5. Diagramas de Flujo Conceptuales

### Flujo de Ingreso y Respaldo

```text
1. El Usuario inicia sesión en la plataforma.
2. Selecciona "Nueva Boleta" y sube la foto desde su celular.
3. Rellena los datos básicos (Fecha, Monto, Categoría).
   *(Aquí es donde, a futuro, la IA haría el trabajo por él).*
4. El servidor Django recibe la información:
   ├─ Guarda la foto de forma segura en Amazon S3.
   ├─ Genera el "Hash" de seguridad para evitar alteraciones.
   └─ Guarda los datos híbridos en PostgreSQL.
5. El sistema envía silenciosamente el correo de respaldo mediante AWS SES.
6. El usuario ve su boleta reflejada inmediatamente en el Calendario y en sus Estadísticas.
```
