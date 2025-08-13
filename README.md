# Api-Camara

# Create a README.md file with the requested Markdown content
readme = """# Especificación de Endpoint — Envío de Evento de Grabación (Consolidado)

Este documento describe la especificación técnica del endpoint para el envío de eventos de grabación **consolidados**.

---

## Tabla de contenidos
1. [Método y URL](#1-método-y-url)
2. [Encabezados (Headers)](#2-encabezados-headers)
   - [Autenticación (JWT Bearer)](#21-autenticación-jwt-bearer)
   - [Idempotency-Key (opcional recomendado)](#22-idempotency-key-opcional-recomendado)
3. [Cuerpo (Body) — JSON](#3-cuerpo-body--json)
   - [Esquema (camelCase)](#31-esquema-camelcase)
   - [Reglas y validaciones](#32-reglas-y-validaciones)
   - [Ejemplo real](#33-ejemplo-real)
4. [Respuestas de la API](#4-respuestas-de-la-api)
5. [Seguridad y Cumplimiento](#5-seguridad-y-cumplimiento)
6. [Ejemplo de llamada cURL](#6-ejemplo-de-llamada-curl)
7. [Notas](#7-notas)

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
```http
Authorization: Bearer <token>


Algoritmo sugerido:

Opción A (simple): HS256 con shared secret (alg: HS256)

Claims mínimos recomendados:

Claim	Descripción	Ejemplo
iss	Issuer	"ordermanager"
sub	ID de la integración o del sistema	"integration-123"
aud	Audience	"algolabs-camaras"
iat	Issued At (epoch seconds)	1723552000
exp	Expiration (epoch seconds)	(5–10 minutos máx.)
jti	ID del token (trazabilidad)	"uuid"
2.2 Idempotency-Key (opcional recomendado)

Para evitar la creación duplicada ante retries, se recomienda enviar:

Idempotency-Key: UUID (por ejemplo: 3d4a1e57-9c7a-4a9b-9a20-1f7a5e2afc93)

3. Cuerpo (Body) — JSON
3.1 Esquema (camelCase)
{
  "fechaHoraInicio": "yyyyMMddHHmmss",
  "fechaHoraFin": "yyyyMMddHHmmss",
  "codigoEquipo": "string",
  "codigoArmado": "string",
  "codigoConsolidado": "string",
  "numeroConsolidado": "string",
  "codigoCliente": "string",
  "data": [
    {
      "comprobante": {
        "codigoComprobante": "string",
        "suc": "string",
        "numeroComprobante": "string"
      }
    }
  ]
}

3.2 Reglas y validaciones

Formato de fechas: yyyyMMddHHmmss (hora local del emisor).

codigoArmado: único por evento.

data: arreglo con ≥ 1 elemento.

Todos los campos son obligatorios.

3.3 Ejemplo real
{
  "fechaHoraInicio": "20250813143000",
  "fechaHoraFin": "20250813144500",
  "codigoEquipo": "7",
  "codigoArmado": "20250813143000-20250813144500-7",
  "codigoConsolidado": "CONSOLIDADO",
  "numeroConsolidado": "000123",
  "codigoCliente": "0000123456",
  "data": [
    {
      "comprobante": {
        "codigoComprobante": "FA",
        "suc": "001",
        "numeroComprobante": "00456789"
      }
    },
    {
      "comprobante": {
        "codigoComprobante": "NC",
        "suc": "001",
        "numeroComprobante": "00456790"
      }
    },
    {
      "comprobante": {
        "codigoComprobante": "RE",
        "suc": "002",
        "numeroComprobante": "00987654"
      }
    }
  ]
}

4. Respuestas de la API

Éxito

202 Accepted — Encolado asíncrono.

Errores

400 Bad Request

401 Unauthorized

403 Forbidden

422 Unprocessable Entity

429 Too Many Requests

500 Internal Server Error

5. Seguridad y Cumplimiento

TLS 1.2+ obligatorio.

Validar exp, aud, iss del JWT.

Limitar el tamaño de la request.

Loguear requestId, codigoArmado, sub (claim) del JWT.

6. Ejemplo de llamada cURL
curl -X POST "https://<host>/api/grabaciones/consolidado" \
  -H "Authorization: Bearer <JWT>" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: 3d4a1e57-9c7a-4a9b-9a20-1f7a5e2afc93" \
  -d '{
    "fechaHoraInicio": "20250813143000",
    "fechaHoraFin": "20250813144500",
    "codigoEquipo": "7",
    "codigoArmado": "20250813143000-20250813144500-7",
    "codigoConsolidado": "CONSOLIDADO",
    "numeroConsolidado": "000123",
    "codigoCliente": "0000123456",
    "data": [
      { "comprobante": { "codigoComprobante": "FA", "suc": "001", "numeroComprobante": "00456789" } },
      { "comprobante": { "codigoComprobante": "NC", "suc": "001", "numeroComprobante": "00456790" } },
      { "comprobante": { "codigoComprobante": "RE", "suc": "002", "numeroComprobante": "00987654" } }
    ]
  }'

7. Notas

El path del endpoint es orientativo.

Las fechas deben expresarse en hora local del emisor con el formato yyyyMMddHHmmss.

Se recomienda utilizar un timeout de JWT de 5–10 minutos para minimizar riesgos de replay.

En escenarios de reintentos, utilice Idempotency-Key para evitar duplicidades.

 ​:contentReference[oaicite:0]{index=0}​
