# Implementación de Grabación de Cámaras en Picking Isla

Este documento describe los cambios necesarios para implementar la funcionalidad de **grabación de datos de pedidos armados** en la tabla `GRABACION_ISLA`, controlada por un parámetro de configuración en base de datos y un valor en el archivo `app.config`.

---

## 1. Creación de tabla `GRABACION_ISLA`

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
---

## 2. Configuración del puesto de trabajo en `app.config`

El código de equipo (puesto) ahora se obtiene desde el archivo de configuración de la aplicación.

### 2.1 Ejemplo de `app.config`

En la sección `<appSettings>`:

```xml
<add key="PickingIsla_CodigoEquipo" value="07" />
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

## 5. Consultas útiles

- **Ver registros pendientes:**
```sql
SELECT *
  FROM dbo.GRABACION_ISLA
 WHERE ESTADO = 'PENDIENTE'
 ORDER BY FECHA_CREACION DESC;
```

- **Ver últimos registros:**
```sql
SELECT TOP 100 *
  FROM dbo.GRABACION_ISLA
 ORDER BY FECHA_CREACION DESC;
```

---

## 6. Instalar Servicio 

Instalar el servicio desde el MSI como administrador, una vez instalado se generara el servicio GrabacionIslaSenderService, antes de iniciar el servicio primero hay configurar el archivo **ApiCam.exe.config**, que se encuentra en donde se instalo, por defecto en C:\Program Files (x86)\Xionico\ApiCam.Setup

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
- [X] Instalar Servicio como administrador.
- [X] Configurar ApiCam.exe.config
- [X] Iniciar Servicio
---
