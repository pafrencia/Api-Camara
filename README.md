# Api-Camara

# Create a README.md file with the requested Markdown content
readme = """# Especificación de Endpoint — Envío de Evento de Grabación (Consolidado)

Este documento describe la especificación técnica del endpoint para el envío de eventos de grabación **consolidados**.

---

## Tabla de contenidos
1. [Método y URL](#1-método-y-url)
2. [Encabezados (Headers)](#2-encabezados-headers)
   - [Autenticación (JWT Bearer)](#21-autenticación-jwt-bearer)
3. [Cuerpo (Body) — JSON](#3-cuerpo-body--json)
   - [Esquema (camelCase)](#31-esquema-camelcase)
   - [Reglas y validaciones](#32-reglas-y-validaciones)
   - [Ejemplo real](#33-ejemplo-real)
4. [Respuestas de la API](#4-respuestas-de-la-api)
5. [Ejemplo de llamada cURL](#5-ejemplo-de-llamada-curl)
6. [Notas](#6-notas)

---

## 1. Método y URL
- **HTTP Method:** `POST`
- **URL sugerida:** `/api/grabaciones/consolidado`  
  *El path es orientativo; pueden proponer otro si lo prefieren.*

---

## 2. Encabezados (Headers)

### 2.1 Autenticación (JWT Bearer)
- **Authorization:** `Bearer <JWT>`
- **Content-Type:** `application/json; charset=utf-8`

**Formato del encabezado:**
Authorization: Bearer <token>

Para esto será necesario un endpoint para loguearse y generar el token o en su defecto deberíamos generar un token "permanente" (lo cual es inseguro.)
---

## 3. Cuerpo (Body) — JSON
### 3.1 Esquema (camelCase)
   ```json
      {
      "fechaHoraInicio":"20250813152521",
      "fechaHoraFin":"20250813152548",
      "codigoEquipo":"07",
      "codigoArmado":"20250813152521-20250813152548-07",
      "codigoConsolidado":"CONSO",
      "numeroConsolidado":"132189",
      "codigoCliente":"0000114921",
      "data":[
         {
            "comprobante":{
               "codigoComprobante":"PEDI",
               "suc":"003",
               "numeroComprobante":"02576234"
            }
         },
         {
            "comprobante":{
               "codigoComprobante":"PEDI",
               "suc":"003",
               "numeroComprobante":"02576235"
            }
         }
      ]
   }
```
### 3.2 Reglas y validaciones

Formato de fechas: yyyyMMddHHmmss (hora local del emisor).

codigoArmado: único por evento.

data: arreglo con ≥ 1 elemento.

Todos los campos son obligatorios.

---

### 3.3 Ejemplo real
   ```json
         {
         "fechaHoraInicio":"20250813152521",
         "fechaHoraFin":"20250813152548",
         "codigoEquipo":"07",
         "codigoArmado":"20250813152521-20250813152548-07",
         "codigoConsolidado":"CONSO",
         "numeroConsolidado":"132189",
         "codigoCliente":"0000114921",
         "data":[
            {
               "comprobante":{
                  "codigoComprobante":"PEDI",
                  "suc":"003",
                  "numeroComprobante":"02576234"
               }
            },
            {
               "comprobante":{
                  "codigoComprobante":"PEDI",
                  "suc":"003",
                  "numeroComprobante":"02576235"
               }
            }
         ]
      }
   ```
---

## 4. Respuestas de la API

Éxito
   ```json

         {
         "url":"...",
         "success":true,
         "error":"..."
         }
   ```

Errores

400 Bad Request
   ```json
         {
         "url": null,
         "success":false,
         "error":"error campo obligatorio X faltante"
         }
   ```
401 Unauthorized
   ```json
         {
         "url": null,
         "success":false,
         "error":"token invalido"
         }
   ```
403 Forbidden
   ```json
         {
         "url": null,
         "success":false,
         "error":"permisos insuficientes"
         }
   ```
500 Internal Server Error
   ```json
         {
         "url": null,
         "success":false,
         "error":"Error en procesamiento..."
         }
   ```
---

## 5. Ejemplo de llamada cURL
   ```curl
   curl -X POST "https://<host>/api/grabaciones/consolidado" \
     -H "Authorization: Bearer <JWT>" \
     -H "Content-Type: application/json" \
     -d '         {
            "fechaHoraInicio":"20250813152521",
            "fechaHoraFin":"20250813152548",
            "codigoEquipo":"07",
            "codigoArmado":"20250813152521-20250813152548-07",
            "codigoConsolidado":"CONSO",
            "numeroConsolidado":"132189",
            "codigoCliente":"0000114921",
            "data":[
               {
                  "comprobante":{
                     "codigoComprobante":"PEDI",
                     "suc":"003",
                     "numeroComprobante":"02576234"
                  }
               },
               {
                  "comprobante":{
                     "codigoComprobante":"PEDI",
                     "suc":"003",
                     "numeroComprobante":"02576235"
                  }
               }
            ]
         }'
   ```
---

## 6. Notas

El path del endpoint es orientativo.

Las fechas deben expresarse con el siguiente formato yyyyMMddHHmmss.

Se recomienda utilizar un timeout de JWT de 5–10 minutos para minimizar riesgos de replay.

 ​:contentReference[oaicite:0]{index=0}​
