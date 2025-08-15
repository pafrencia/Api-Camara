# Implementación de Grabación de Cámaras en Picking Isla

Este documento describe los cambios necesarios para implementar la funcionalidad de **grabación de datos de pedidos armados** en la tabla `GRABACION_ISLA`, controlada por un parámetro de configuración en base de datos y un valor en el archivo `app.config`.

---

## 1. Creación de tabla `GRABACION_ISLA`

### 1.1 Script de creación

```sql
CREATE TABLE dbo.GRABACION_ISLA
(
    CODIGO_ARMADO     NVARCHAR(50)  NOT NULL,   -- FechaInicio-FechaFin-CodigoEquipo (clave lógica del armado)
    ESTADO            NVARCHAR(20)  NOT NULL,   -- PENDIENTE | ENVIADO | ERROR (u otros)
    JSON              NVARCHAR(MAX) NOT NULL,   -- Payload con datos del armado
    FECHA_ENVIO       DATETIME      NULL,       -- Timestamp al despachar a la API externa
    RESPUESTA_DE_API  NVARCHAR(MAX) NULL,       -- Respuesta o error devuelto por la API
    FECHA_CREACION    DATETIME      NOT NULL DEFAULT GETDATE()
);
