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
