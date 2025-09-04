# Implementación de Grabación de Cámaras en Picking Isla

Este documento describe los cambios necesarios para implementar la funcionalidad de **grabación de datos de pedidos armados** en la tabla `GRABACION_ISLA`, controlada por un parámetro de configuración en base de datos y un valor en el archivo `app.config`.

---

## 1. Creación de tabla `GRABACION_ISLA` y modificación de PICKING_ISLA_REIMPRESION_BULTOS y sp_picking_isla_reimpresion_bultos

### 1.1 Script de creación

#### la base de datos debe tener esta tabla, si no existe crearla

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

```
### 1.2 Script de modificacion de tabla PICKING_ISLA_REIMPRESION_BULTOS

#### la tabla PICKING_ISLA_REIMPRESION_BULTOS debe tener este campo, CODIGO_ARMADO_API

```sql
ALTER TABLE dbo.PICKING_ISLA_REIMPRESION_BULTOS
ADD CODIGO_ARMADO_API nvarchar(50) NULL;

```
### 1.3 Script de modificacion de sp_picking_isla_reimpresion_bultos

```sql
GO
/****** Object:  StoredProcedure [dbo].[sp_picking_isla_reimpresion_bultos] ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO

ALTER PROCEDURE [dbo].[sp_picking_isla_reimpresion_bultos]
(
    @CODIGO_CONSOLIDADO        NVARCHAR(5),
    @NUMERO_CONSOLIDADO        NUMERIC(18,0),
    @CODIGO_CLIENTE            NVARCHAR(10),
    @NOMBRE_CLIENTE            NVARCHAR(50),
    @DIRECCION_CLIENTE         NVARCHAR(50),
    @ZONA_CLIENTE              NVARCHAR(10),
    @ORDEN_CLIENTE             NVARCHAR(10),
    @CODIGO_COMPROBANTE        NVARCHAR(5),
    @NUMERO_COMPROBANTE        NUMERIC(18,0),
    @SUCURSAL                  SMALLINT,
    @COMPROBANTES_MULTIPLES    NVARCHAR(20),
    @CANTIDAD_BULTOS           SMALLINT,
    @FECHA_HORA                NVARCHAR(20),
    @CON_DIFERENCIA            NVARCHAR(1),
    @LOCALIDAD_CLIENTE         NVARCHAR(50) = '',
    @ID_TRANSPORTISTA          NVARCHAR(20) = '',
    @ID_USUARIO                NVARCHAR(20) = '',
    @ORDEN_ARMADO              NVARCHAR(10) = '',
    @LISTA_COMPROBANTES        NVARCHAR(500) = '',
    @OBSERVACION_COMPROBANTE   NVARCHAR(500) = '',
    @CODIGO_ARMADO_ISLA        NVARCHAR(50) = '',
    -- NUEVO
    @CODIGO_ARMADO_API         NVARCHAR(50) = NULL
)
AS
BEGIN
    SET NOCOUNT ON;

    -- si no viene CODIGO_ARMADO_ISLA generamos un valor incremental
    IF (@CODIGO_ARMADO_ISLA = '' OR @CODIGO_ARMADO_ISLA IS NULL)
    BEGIN
        SELECT @CODIGO_ARMADO_ISLA = CONVERT(INT, ISNULL(MAX(CODIGO_ARMADO_ISLA), 0)) + 1
        FROM dbo.PICKING_ISLA_REIMPRESION_BULTOS;

        IF (@CODIGO_ARMADO_ISLA IS NULL OR @CODIGO_ARMADO_ISLA = '') 
            SET @CODIGO_ARMADO_ISLA = '1';
    END

    IF NOT EXISTS (
        SELECT 1
        FROM dbo.PICKING_ISLA_REIMPRESION_BULTOS
        WHERE CODIGO_CONSOLIDADO = @CODIGO_CONSOLIDADO
          AND NUMERO_CONSOLIDADO = @NUMERO_CONSOLIDADO
          AND CODIGO_CLIENTE     = @CODIGO_CLIENTE
          AND CODIGO_COMPROBANTE = @CODIGO_COMPROBANTE
          AND NUMERO_COMPROBANTE = @NUMERO_COMPROBANTE
          AND SUCURSAL           = @SUCURSAL
          AND CODIGO_ARMADO_ISLA = @CODIGO_ARMADO_ISLA
    )
    BEGIN
        INSERT INTO dbo.PICKING_ISLA_REIMPRESION_BULTOS
        (
            CODIGO_CONSOLIDADO,
            NUMERO_CONSOLIDADO,
            CODIGO_CLIENTE,
            NOMBRE_CLIENTE,
            DIRECCION_CLIENTE,
            ZONA_CLIENTE,
            ORDEN_CLIENTE,
            CODIGO_COMPROBANTE,
            NUMERO_COMPROBANTE,
            SUCURSAL,
            COMPROBANTES_MULTIPLES,
            CANTIDAD_BULTOS,
            FECHA_HORA,
            CON_DIFERENCIA,
            CLIENTE_LOCALIDAD,
            TRANSPORTISTA,
            ID_USUARIO,
            ORDEN_ARMADO,
            LISTA_COMPROBANTES,
            OBSERVACION_COMPROBANTE,
            CODIGO_ARMADO_ISLA,
            CODIGO_ARMADO_API
        )
        VALUES
        (
            @CODIGO_CONSOLIDADO,
            @NUMERO_CONSOLIDADO,
            @CODIGO_CLIENTE,
            @NOMBRE_CLIENTE,
            @DIRECCION_CLIENTE,
            @ZONA_CLIENTE,
            @ORDEN_CLIENTE,
            @CODIGO_COMPROBANTE,
            @NUMERO_COMPROBANTE,
            @SUCURSAL,
            @COMPROBANTES_MULTIPLES,
            @CANTIDAD_BULTOS,
            @FECHA_HORA,
            @CON_DIFERENCIA,
            @LOCALIDAD_CLIENTE,
            @ID_TRANSPORTISTA,
            @ID_USUARIO,
            @ORDEN_ARMADO,
            @LISTA_COMPROBANTES,
            @OBSERVACION_COMPROBANTE,
            @CODIGO_ARMADO_ISLA,
            @CODIGO_ARMADO_API
        );
    END
    ELSE
    BEGIN
        UPDATE dbo.PICKING_ISLA_REIMPRESION_BULTOS
           SET NOMBRE_CLIENTE        = @NOMBRE_CLIENTE,
               DIRECCION_CLIENTE     = @DIRECCION_CLIENTE,
               ZONA_CLIENTE          = @ZONA_CLIENTE,
               ORDEN_CLIENTE         = @ORDEN_CLIENTE,
               CODIGO_COMPROBANTE    = @CODIGO_COMPROBANTE,
               NUMERO_COMPROBANTE    = @NUMERO_COMPROBANTE,
               SUCURSAL              = @SUCURSAL,
               COMPROBANTES_MULTIPLES= @COMPROBANTES_MULTIPLES,
               CANTIDAD_BULTOS       = @CANTIDAD_BULTOS,
               FECHA_HORA            = @FECHA_HORA,
               CON_DIFERENCIA        = @CON_DIFERENCIA,
               CLIENTE_LOCALIDAD     = @LOCALIDAD_CLIENTE,
               TRANSPORTISTA         = @ID_TRANSPORTISTA,
               ID_USUARIO            = @ID_USUARIO,
               ORDEN_ARMADO          = @ORDEN_ARMADO,
               LISTA_COMPROBANTES    = @LISTA_COMPROBANTES,
               OBSERVACION_COMPROBANTE = @OBSERVACION_COMPROBANTE,
               CODIGO_ARMADO_API     = COALESCE(@CODIGO_ARMADO_API, CODIGO_ARMADO_API)
         WHERE CODIGO_CONSOLIDADO    = @CODIGO_CONSOLIDADO
           AND NUMERO_CONSOLIDADO    = @NUMERO_CONSOLIDADO
           AND CODIGO_CLIENTE        = @CODIGO_CLIENTE
           AND CODIGO_COMPROBANTE    = @CODIGO_COMPROBANTE
           AND NUMERO_COMPROBANTE    = @NUMERO_COMPROBANTE
           AND SUCURSAL              = @SUCURSAL
           AND CODIGO_ARMADO_ISLA    = @CODIGO_ARMADO_ISLA;
    END
END
GO


```
---

## 2. Configuración del puesto de trabajo y url base para la URL en `app.config`

El código de equipo (puesto) ahora se obtiene desde el archivo de configuración de la aplicación.
esto debe configurarsen en el app.config, se puede hacer tambien desde OrderManager desde el menu de Ayuda - Mantenimiento - en la pestaña Puesto de Trabajo.

### 2.1 Ejemplo de `app.config`

En la sección `<appSettings>`:

```xml
<add key="PickingIsla_CodigoEquipo" value="07" />

	  <!-- URL BASE PARA FORMAR EL URL DEL VIDEO SE FORMARA CON ESTE URL  + CODIGO_ARMADO_API -->
<add key="QrBaseUrl" value="https://ec.algolabs.ai/p/" />
```

- **value** = identificador del puesto de trabajo (por ejemplo: `01`, `07`, `B3`, etc.).
- Si está vacío, la aplicación usará el valor por defecto `"00"`.

---

## 3. Parámetro de control en base de datos (`Grabacion_Camaras`)

La ejecución de la lógica de guardado en `GRABACION_ISLA` depende del parámetro **`Grabacion_Camaras`** en la tabla `PARAMETROS`.

- **Valor = 1:** Activa la grabación (se insertan registros en `GRABACION_ISLA`).
- **Valor distinto de 1 o inexistente:** Desactiva la grabación (no se inserta nada).

### 3.1 Script de creación/actualización del parámetro

script idempotente (si existe lo actualiza; si no, lo inserta) para Grabacion_Camaras

```sql
GO

DECLARE @ID           NVARCHAR(50)  = N'Grabacion_Camaras';
DECLARE @Valor        NVARCHAR(100) = N'1';
DECLARE @Descripcion  NVARCHAR(500) = N'este valor indica si utiliza camaras para grabacion en picking isla; si es 1 entonces se ejecutara la logica de guardado de datos en GRABACION_ISLA de los pedidos armados; si no es 1 entonces no se guardara nada. Recordar tener configurado en app.config el <add key="PickingIsla_CodigoEquipo" value="07"/> con value igual al puesto de trabajo.';

IF EXISTS (SELECT 1 FROM dbo.PARAMETROS WHERE ID_Parametro = @ID)
BEGIN
    UPDATE dbo.PARAMETROS
       SET Valor = @Valor,
           Descripcion = @Descripcion
     WHERE ID_Parametro = @ID;
END
ELSE
BEGIN
    INSERT INTO dbo.PARAMETROS (ID_Parametro, Valor, Descripcion)
    VALUES (@ID, @Valor, @Descripcion);
END
GO

-- Verificar
SELECT ID_Parametro, Valor, Descripcion
FROM dbo.PARAMETROS
WHERE ID_Parametro = N'Grabacion_Camaras';
```

**Descripción funcional del parámetro**  
> Indica si se utilizan cámaras para grabación en Picking Isla.  
> Si es `1`, se ejecutará la lógica de guardado en `GRABACION_ISLA`.  
> Si no es `1`, no se guardará nada.  
> **Importante:** recordar configurar en `app.config` la clave `<add key="PickingIsla_CodigoEquipo" value="XX"/>` con el puesto de trabajo correcto.

---

## 4. Flujo resumido

1. **Formulario padre (`ARCOR_frmPickingIsla`):**
   - Lee el parámetro `Grabacion_Camaras` de la base de datos.
   - Si es `1`, lee `PickingIsla_CodigoEquipo` de `app.config`.
   - Pasa al formulario hijo:
     - `sFechaHoraInicio`
     - `sCodigoEquipo`

2. **Formulario hijo (`ARCOR_frmArmadoPedido`):**
   - Solo inserta en `GRABACION_ISLA` si `Grabacion_Camaras = 1`.
   - Usa el `sCodigoEquipo` recibido del padre.
   - Construye el `CODIGO_ARMADO` como:
   ```
   <FechaHoraInicio>-<FechaHoraFin>-<CodigoEquipo>
   ```
   - Inserta el JSON y el estado inicial `PENDIENTE`.

---

## 5. Copiar .dll ZXing


Al actualizar el ejecutable,. Obligatorio incluir la DLL de ZXing junto al .exe.

debe ir en la misma carpeta del ejecutable

OrderManager.exe

OrderManager.exe.config

ZXing.Net.dll (versión 0.16.10.0) ⟵ necesaria para generar el PNG del QR

Si falta, aparecerá el error: “No se puede cargar el archivo o ensamblado 'zxing, Version=0.16.10.0 …”

## 6. Instalar Servicio 

Instalar el servicio desde el MSI como administrador, una vez instalado se generara el servicio GrabacionIslaSenderService, antes de iniciar el servicio primero hay configurar el archivo **ApiCam.exe.config**, que se encuentra en donde se instalo, por defecto en C:\Program Files (x86)\Xionico\ApiCam.Setup
EJEMPLO DEL CONFIG DEBE REEMPLAZAR LOS VALORES POR LOS REALES
```xml

<?xml version="1.0" encoding="utf-8" ?>
	<configuration>
		<startup>
			<supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.8" />
		</startup>
		<appSettings>
			<add key="BatchSize" value="10"/>
			<add key="LogEchoConsole" value="1" /> <!-- 1=eco a consola, 0=no -->
			<!-- Frecuencia del ciclo
			Intervalo global del ciclo en milisegundos.
			Es la frecuencia con la que el servicio busca pendientes
			en la tabla GRABACION_ISLA y los envía. 
			-->
			<add key="PollMs" value="60000" />
			<!-- Pausa entre cada envío dentro de un mismo ciclo.
			Ejemplo: si hay 20 registros pendientes, entre cada POST a la API espera 500 ms.
			Esto sirve para no saturar la API ni la red.
			-->
			<add key="InterSendDelayMs" value="500" />
			<!-- API  -->
  <add key="ApiBaseUrl" value="https://api.algolabs.ai/" />
  <add key="ApiConsolidadoEndpoint" value="grabaciones/consolidado" />
  <add key="ApiLoginEndpoint" value="login/access-token" />
  <!-- credenciales de login -->
  <add key="ApiAuthUsername" value="admin" />
  <add key="ApiAuthPassword" value="1234" />
  <!-- si no usan client_secret, dejalo vacío -->
  <add key="ApiClientId" value="string" />
  <add key="ApiClientSecret" value="s" />
  <!-- segundos de margen para refrescar token antes del exp -->
  <add key="ApiTokenSkewSeconds" value="60" />
		</appSettings>

		<connectionStrings>
			<clear />
			  <!-- Cadena de conexion a la base de datos, encriptada con la misma encriptacion de emser service -->
			<add name="ConnectionString"				 connectionString="sfLYxf09PqDD7puslrd4tsf2XQYZFJua8wZsEo0N1ItDkCZs8pXabxgDA5z+A/dTzJKsg7BHl86aK8Tdb3bpjuFF5XMRyaen/aRUJxTXiIJD9DL4J+8g8RMhoFehd5Ae2qE085j4JwQ="/>
		</connectionStrings>
	</configuration>

```

---

---

## 7. Checklist de implementación

- [x] Crear tabla `GRABACION_ISLA`.
- [x] Crear o actualizar parámetro `Grabacion_Camaras` en base de datos.
- [x] Configurar `<add key="PickingIsla_CodigoEquipo" value="XX" />` en `app.config`.
- [x] Verificar permisos de usuario de aplicación sobre `PARAMETROS` y `GRABACION_ISLA`.
- [x] Copiar en la misma carpeta del .exe la DLL ZXing.Net.dll (0.16.10.0).
- [X] Instalar Servicio como administrador.
- [X] Configurar ApiCam.exe.config
- [X] Iniciar Servicio
---
