# DOCUMENTACIÓN DEL MODELO DE BASE DE DATOS
## Sistema de Venta y Mantenimiento de Vehículos

Este documento presenta el diseño completo del modelo de datos de un sistema orientado a la gestión de ventas de vehículos y los servicios de mantenimiento posteriores. Se incluyen tanto el diagrama conceptual (notación de Chen) como el modelo relacional implementable, junto con la justificación de cada decisión de diseño, las restricciones de integridad y la explicación detallada de todas las relaciones entre entidades.

---

## 1. Justificación del Diseño

### 1.1 Contexto y alcance del sistema
El sistema debe permitir: registrar clientes y vendedores, gestionar el catálogo de vehículos (con su estado y disponibilidad), registrar ventas (que pueden incluir uno o varios vehículos), y llevar el historial de mantenimientos de cada vehículo. El modelo se diseñó para soportar estas operaciones manteniendo la integridad de los datos y facilitando reportes de negocio (ventas por vendedor, historial por vehículo, servicios por cliente, etc.).

### 1.2 Estructura de las tablas
Se definieron seis tablas. Las cuatro entidades fuertes (CLIENTE, VENDEDOR, VEHICULO y VENTA) representan los objetos principales del negocio. MANTENIMIENTO modela los servicios técnicos. DETALLE_VENTA es una tabla de unión necesaria para resolver la relación muchos-a-muchos entre VENTA y VEHICULO, y además almacena el precio real de venta de cada unidad.

### 1.3 Decisiones de diseño importantes

**a) Clave primaria de VEHICULO = id_vehiculo (no el VIN)**  
El VIN es un identificador natural único, pero es una cadena larga (17 caracteres) y compleja. Usar un entero autoincremental como PK simplifica las claves foráneas, reduce el tamaño de los índices y mejora el rendimiento de los JOINs. El VIN se conserva como atributo UNIQUE.

**b) Tabla DETALLE_VENTA para la relación N:M**  
Una venta puede incluir varios vehículos (compra de flota) y, a lo largo del tiempo, un mismo vehículo podría aparecer en más de una transacción (reventa, consignación). Por eso no se coloca una simple FK en una de las dos tablas. La tabla intermedia permite además guardar el **precio_venta** pactado en esa operación concreta (que puede diferir del precio de lista).

**c) id_cliente opcional en MANTENIMIENTO**  
Se permite NULL para poder registrar servicios realizados a vehículos de la propia empresa, unidades de demostración o vehículos cuyo propietario no esté registrado como cliente.

**d) Separación de responsabilidades**  
Los datos personales del vendedor viven en VENDEDOR. La información de la factura (número, fecha, total, método de pago) vive en VENTA. Los atributos técnicos del vehículo (combustible, transmisión, estado, disponibilidad) se concentran en VEHICULO. Esto evita redundancia y facilita el mantenimiento del esquema.

### 1.4 Mapeo Chen → Relacional

| Relación (Chen)                      | Cardinalidad | Implementación                                      |
|--------------------------------------|--------------|-----------------------------------------------------|
| Realiza (Cliente – Venta)            | 1:N          | FK id_cliente en VENTA                              |
| Registra (Vendedor – Venta)          | 1:N          | FK id_vendedor en VENTA                             |
| Incluye (Venta – Vehículo)           | N:M          | Tabla DETALLE_VENTA (PK compuesta)                  |
| Solicita (Cliente – Mantenimiento)   | 1:N          | FK id_cliente en MANTENIMIENTO (NULL permitido)     |
| Recibe (Vehículo – Mantenimiento)    | 1:N          | FK id_vehiculo en MANTENIMIENTO                     |

---

## 2. Restricciones y Validaciones

### 2.1 Claves Primarias

| Tabla            | PK                      | Tipo     | Notas                          |
|------------------|-------------------------|----------|--------------------------------|
| CLIENTE          | id_cliente              | INT      | Autoincremental                |
| VENDEDOR         | id_vendedor             | INT      | Autoincremental                |
| VENTA            | id_venta                | INT      | Autoincremental                |
| VEHICULO         | id_vehiculo             | INT      | Autoincremental (VIN no es PK) |
| MANTENIMIENTO    | id_mantenimiento        | INT      | Autoincremental                |
| DETALLE_VENTA    | id_venta + id_vehiculo  | INT + INT| Clave primaria compuesta       |

### 2.2 Claves Foráneas
- **VENTA.id_cliente** → CLIENTE.id_cliente — toda venta debe tener un cliente válido
- **VENTA.id_vendedor** → VENDEDOR.id_vendedor — toda venta es registrada por un vendedor
- **DETALLE_VENTA.id_venta** → VENTA.id_venta
- **DETALLE_VENTA.id_vehiculo** → VEHICULO.id_vehiculo
- **MANTENIMIENTO.id_vehiculo** → VEHICULO.id_vehiculo — obligatorio
- **MANTENIMIENTO.id_cliente** → CLIENTE.id_cliente — opcional (acepta NULL)

### 2.3 Restricciones UNIQUE y otras reglas
- **VEHICULO.vin** UNIQUE — el número de identificación vehicular es único a nivel mundial.
- **VENDEDOR.numero_empleado** UNIQUE — número interno único de cada empleado.
- **VENTA.num_factura** — se recomienda UNIQUE para evitar facturas duplicadas.
- La mayoría de atributos de negocio son NOT NULL. Solo **MANTENIMIENTO.id_cliente** admite nulos.
- Tipos precisos: DECIMAL(10,2) para montos, DATETIME para fechas con hora, BOOLEAN para disponibilidad.

---

## 3. Relaciones entre Entidades (explicación detallada)

Esta sección explica el significado de cada relación que aparece en los dos diagramas (notación Chen y modelo relacional de tablas).

### 3.1 Realiza (Cliente → Venta) — 1:N
Un cliente puede realizar muchas ventas a lo largo del tiempo, pero cada venta pertenece a un único cliente. En el modelo relacional se implementa con la FK **id_cliente** en la tabla VENTA. Garantiza que no existan ventas “huérfanas” sin cliente asociado.

### 3.2 Registra (Vendedor → Venta) — 1:N
Cada venta es registrada por un solo vendedor; un vendedor puede registrar muchas ventas. Se implementa con la FK **id_vendedor** en VENTA. Permite calcular comisiones, ranking de vendedores y reportes de productividad.

### 3.3 Incluye (Venta ↔ Vehículo) — N:M
Una venta puede incluir varios vehículos y un vehículo puede aparecer en más de una venta histórica. Por eso se crea la tabla intermedia **DETALLE_VENTA** con clave primaria compuesta (id_venta + id_vehiculo). Además se guarda el atributo **precio_venta**, que pertenece a la relación (el precio real de esa transacción, no el precio de lista).

### 3.4 Solicita (Cliente → Mantenimiento) — 1:N
Un cliente puede solicitar muchos servicios de mantenimiento. Se refleja con la FK **id_cliente** en MANTENIMIENTO. Se permite NULL para servicios realizados sin cliente registrado (flota interna, demostraciones, etc.).

### 3.5 Recibe (Vehículo → Mantenimiento) — 1:N
Cada mantenimiento se aplica a un vehículo concreto y un vehículo puede recibir muchos servicios a lo largo de su vida. Se implementa con la FK obligatoria **id_vehiculo** en MANTENIMIENTO. Permite construir el historial técnico completo de cada unidad.

### 3.6 Resumen de cardinalidades

| Entidad A   | Relación  | Entidad B     | Cardinalidad |
|--------------|-----------|----------------|--------------|
| Cliente      | Realiza   | Venta          | 1 : N        |
| Vendedor     | Registra  | Venta          | 1 : N        |
| Venta        | Incluye   | Vehículo       | N : M        |
| Cliente      | Solicita  | Mantenimiento  | 1 : N        |
| Vehículo     | Recibe    | Mantenimiento  | 1 : N        |

---

## 4. Descripción detallada de cada tabla

**CLIENTE** — Personas o empresas que compran vehículos o solicitan servicios.  
Atributos: id_cliente (PK), nombre, telefono, correo, direccion.

**VENDEDOR** — Empleados que registran las ventas.  
Atributos: id_vendedor (PK), nombre, numero_empleado (UNIQUE), fecha_contratacion.

**VENTA** — Cabecera de cada transacción de venta.  
Atributos: id_venta (PK), num_factura, fecha_venta, total, metodo_pago, id_cliente (FK), id_vendedor (FK).

**VEHICULO** — Catálogo de vehículos disponibles o vendidos.  
Atributos: id_vehiculo (PK), marca, modelo, anio, vin (UNIQUE), precio, color, tipo_combustible, tipo_transmision, estado, disponible.

**MANTENIMIENTO** — Servicios técnicos realizados a los vehículos.  
Atributos: id_mantenimiento (PK), tipo_servicio, costo, fecha_servicio, id_vehiculo (FK), id_cliente (FK, NULL).

**DETALLE_VENTA** — Líneas de detalle de cada venta (relación N:M).  
Atributos: id_venta (PK/FK), id_vehiculo (PK/FK), precio_venta.

---

## 5. Beneficios del diseño y recomendaciones

### 5.1 Beneficios
- Normalización hasta 3FN: se elimina la redundancia de datos de clientes, vendedores y vehículos.
- Integridad referencial garantizada por las claves foráneas.
- Flexibilidad para ventas de uno o varios vehículos gracias a DETALLE_VENTA.
- Historial completo de mantenimientos por vehículo (y opcionalmente por cliente).
- Facilidad para generar reportes: ventas por vendedor, por periodo, por marca, ingresos por mantenimiento, etc.

### 5.2 Recomendaciones de implementación
- Crear índices sobre todas las claves foráneas (id_cliente, id_vendedor, id_vehiculo) para acelerar los JOINs.
- Crear índice UNIQUE sobre VEHICULO.vin y VENDEDOR.numero_empleado.
- Considerar un trigger o regla de negocio que impida vender un vehículo con disponible = FALSE.
- En entornos de producción se puede agregar auditoría (quién creó/modificó cada registro) y soft-delete.

### 5.3 Posibles extensiones futuras
- Tabla de Pagos (una venta puede pagarse en varias cuotas).
- Tabla de Inventario / Movimientos de stock.
- Tabla de Citas de mantenimiento (agendar servicios futuros).
- Tabla de Usuarios y roles para control de acceso a la aplicación.

---

*Documento generado para el proyecto de modelado de base de datos del sistema de venta y mantenimiento de vehículos. — Julio 2026*
