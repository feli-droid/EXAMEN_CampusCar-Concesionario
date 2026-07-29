# Nombre del Proyecto

Breve descripción (1-2 líneas) de qué trata el proyecto.

## 📋 Descripción

Explicación más detallada del sistema (ventas y mantenimiento de vehículos).

## 🎯 Objetivos

- Objetivo 1
- Objetivo 2
- Objetivo 3

## 🗂️ Estructura del Proyecto
proyecto/
├── diagramas/
│   ├── ER_Chen.drawio          # Diagrama entidad-relación (notación Chen)
│   └── ER_Relacional.drawio    # Modelo relacional (tablas + FK)
├── documentacion/
│   └── Documentacion_Modelo_BD.pdf
└── README.md
text## 📊 Modelo de Datos

### Entidades principales
- Cliente
- Vendedor
- Venta
- Vehículo
- Mantenimiento
- Detalle_Venta

### Relaciones importantes
| Relación              | Cardinalidad | Descripción                     |
|-----------------------|--------------|---------------------------------|
| Cliente → Venta       | 1:N          | Un cliente realiza muchas ventas|
| Vendedor → Venta      | 1:N          | Un vendedor registra muchas ventas|
| Venta ↔ Vehículo      | N:M          | A través de Detalle_Venta       |
| Cliente → Mantenimiento| 1:N         | Un cliente solicita muchos servicios|
| Vehículo → Mantenimiento| 1:N        | Un vehículo recibe muchos servicios|

## 🔑 Decisiones de Diseño

- Se usa `id_vehiculo` como PK en lugar de VIN
- Tabla intermedia `DETALLE_VENTA` para la relación N:M
- `id_cliente` en Mantenimiento permite NULL

## 📁 Archivos Incluidos

| Archivo                              | Descripción                          |
|--------------------------------------|--------------------------------------|
| `ER_Diagrama_Venta_Vehiculos.drawio` | Diagrama Chen                        |
| `ER_Relacional_Venta_Vehiculos.drawio`| Modelo relacional                   |
| `Documentacion_Modelo_BD_Vehiculos.pdf` | Documentación completa             |

## 🛠️ Cómo usar

1. Abrir los archivos `.drawio` en [diagrams.net](https://app.diagrams.net)
2. Revisar el PDF de documentación

## 👤 Autor

Tu nombre / Grupo

## 📅 Fecha

Julio 2026
