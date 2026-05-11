# Manual de Usuario - API Chashin

## Descripción General

La API Chashin proporciona dos endpoints principales para búsqueda y consulta de entidades (personas naturales y jurídicas).

**URL Base:** `https://ext.firmaconcerteza.com/`

---

## Autenticación

Todos los endpoints requieren un token de autenticación que debe enviarse en el header `Authorization`.

```
Authorization: chashin_7k9mP2xL4nQ8vR5wT3yU6zB1dF0gH9jC
```

---

## Endpoints

### 1. Búsqueda de Entidades

Realiza búsquedas de personas naturales o jurídicas en la base de datos.

**Endpoint:** `POST /chashin/busqueda/1`

**Headers:**
```
Content-Type: application/json
Authorization: chashin_7k9mP2xL4nQ8vR5wT3yU6zB1dF0gH9jC
```

**Body (JSON):**

| Campo | Tipo | Requerido | Descripción | Valores Permitidos |
|-------|------|-----------|-------------|-------------------|
| `query` | string | Sí | Texto a buscar | Cualquier texto |
| `tipo` | string | Sí | Tipo de entidad a buscar | `natural`, `juridico` |
| `tipoBusqueda` | string | Sí | Tipo de búsqueda a realizar | `normal`, `fuzzy` |
| `page` | number | Sí | Número de página | Entero positivo (ej: 1, 2, 3...) |
| `size` | number | Sí | Cantidad de resultados por página | Entero positivo (ej: 10, 20, 50...) |

**Ejemplo de Solicitud:**

```http
POST http://161.35.107.123/chashin/busqueda/1
Content-Type: application/json
Authorization: chashin_7k9mP2xL4nQ8vR5wT3yU6zB1dF0gH9jC

{
  "query": "jorge raul",
  "tipoBusqueda": "normal",
  "tipo": "natural",
  "page": 1,
  "size": 20
}
```

**Ejemplo con búsqueda fuzzy:**

```http
POST http://161.35.107.123/chashin/busqueda/1
Content-Type: application/json
Authorization: chashin_7k9mP2xL4nQ8vR5wT3yU6zB1dF0gH9jC

{
  "query": "empresa tecnologia",
  "tipoBusqueda": "fuzzy",
  "tipo": "juridico",
  "page": 1,
  "size": 10
}
```

**Respuesta Exitosa (200):**

```json
{
  "exito": true,
  "mensaje": "Búsqueda exitosa",
  "datos": {
    "resultados": [...],
    "total": 150,
    "pagina": 1,
    "tamanio": 20
  },
  "metadata": [
    { "clave": "fuente", "valor": "mongo+elastic" },
    { "clave": "generado_el", "valor": "2026-01-09T10:30:00.000Z" }
  ]
}
```

---

### 2. Detalle de Entidad

Obtiene información detallada de una entidad específica mediante su ID.

**Endpoint:** `POST /chashin/entidad/1`

**Headers:**
```
Content-Type: application/json
Authorization: chashin_7k9mP2xL4nQ8vR5wT3yU6zB1dF0gH9jC
```

**Body (JSON):**

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| `id` | string | Sí | UUID de la entidad |

**Ejemplo de Solicitud:**

```http
POST http://161.35.107.123/chashin/entidad/1
Content-Type: application/json
Authorization: chashin_7k9mP2xL4nQ8vR5wT3yU6zB1dF0gH9jC

{
  "id": "2a731c57-833c-5477-84bd-9118d0225347"
}
```

**Respuesta Exitosa (200):**

```json
{
  "exito": true,
  "mensaje": "Detalle encontrado",
  "datos": {
    "id": "2a731c57-833c-5477-84bd-9118d0225347",
    "nombre": "Jorge Raul García",
    "tipo": "natural",
    "informacion": {...}
  },
  "metadata": [
    { "clave": "fuente", "valor": "mongo" },
    { "clave": "generado_el", "valor": "2026-01-09T10:30:00.000Z" }
  ]
}
```

---

## Códigos de Respuesta

| Código | Descripción |
|--------|-------------|
| `200` | Solicitud exitosa |
| `400` | Parámetros faltantes o inválidos |
| `401` | Token de autenticación no proporcionado |
| `403` | Token de autenticación inválido |
| `500` | Error interno del servidor |

---

## Ejemplos de Errores

### Error 400 - Parámetro faltante

```json
{
  "exito": false,
  "mensaje": "El campo 'id' es requerido en el body",
  "error": "Parámetro faltante"
}
```

### Error 401 - Sin token

```json
{
  "exito": false,
  "mensaje": "Token de autenticación requerido",
  "error": "No se proporcionó token en el header Authorization"
}
```

### Error 403 - Token inválido

```json
{
  "exito": false,
  "mensaje": "Token inválido",
  "error": "El token proporcionado no es válido para Chashin"
}
```

---

## Notas Importantes

- **Búsqueda Normal vs Fuzzy:**
  - `normal`: Búsqueda exacta o por coincidencia parcial
  - `fuzzy`: Búsqueda tolerante a errores tipográficos
  
- **Tipos de Entidad:**
  - `natural`: Personas físicas
  - `juridico`: Empresas u organizaciones
  
- **Paginación:**
  - La paginación comienza en `page: 1`
  - El parámetro `size` determina cuántos resultados se devuelven por página
  - Valores recomendados de `size`: 10, 20, 50, 100

- **Formato del ID:**
  - Los IDs son UUIDs en formato: `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`

---

## Ejemplos con cURL

### Búsqueda

```bash
curl -X POST http://161.35.107.123/chashin/busqueda/1 \
  -H "Content-Type: application/json" \
  -H "Authorization: chashin_7k9mP2xL4nQ8vR5wT3yU6zB1dF0gH9jC" \
  -d '{
    "query": "jorge raul",
    "tipoBusqueda": "normal",
    "tipo": "natural",
    "page": 1,
    "size": 20
  }'
```

### Detalle

```bash
curl -X POST http://161.35.107.123/chashin/entidad/1 \
  -H "Content-Type: application/json" \
  -H "Authorization: chashin_7k9mP2xL4nQ8vR5wT3yU6zB1dF0gH9jC" \
  -d '{
    "id": "2a731c57-833c-5477-84bd-9118d0225347"
  }'
```

---

## Ejemplos con JavaScript (fetch)

### Búsqueda

```javascript
const buscarEntidad = async () => {
  const response = await fetch('http://161.35.107.123/chashin/busqueda/1', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': 'chashin_7k9mP2xL4nQ8vR5wT3yU6zB1dF0gH9jC'
    },
    body: JSON.stringify({
      query: 'jorge raul',
      tipoBusqueda: 'normal',
      tipo: 'natural',
      page: 1,
      size: 20
    })
  });
  
  const data = await response.json();
  console.log(data);
};
```

### Detalle

```javascript
const obtenerDetalle = async () => {
  const response = await fetch('http://161.35.107.123/chashin/entidad/1', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': 'chashin_7k9mP2xL4nQ8vR5wT3yU6zB1dF0gH9jC'
    },
    body: JSON.stringify({
      id: '2a731c57-833c-5477-84bd-9118d0225347'
    })
  });
  
  const data = await response.json();
  console.log(data);
};
```

---

## Soporte

Para reportar problemas o solicitar ayuda, contacta al equipo de desarrollo.
