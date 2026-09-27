# IVA Monitoreo para backend

Esta guia explica como usar el microservicio `iva-monitoreo` para:

- registrar entidades monitoreadas por usuario;
- enviar eventos encontrados por otros servicios;
- consultar notificaciones;
- marcar notificaciones como leidas.

## URL base

```text
http://161.35.107.123:3011
```

## Tokens

### Token para monitoreos y notificaciones

Usar este header:

```http
x-api-key: a9087485730ed01568457f3ab620a2f0cfbc10aaca2a0be3
```

Aplica para:

- `POST /api/monitoreos`
- `GET /api/monitoreos/{usuario_id}`
- `DELETE /api/monitoreos/{usuario_id}/{entidad_id}`
- `GET /api/notificaciones/{usuario_id}`
- `GET /api/notificaciones/{usuario_id}/contador`
- `PATCH /api/notificaciones/{notificacion_id}/leida`
- `PATCH /api/notificaciones/leidas`

### Token interno para eventos

Usar este header:

```http
x-internal-token: 902ecb99fb30acea7e23c7525fac0a2cdb77201e08211fd3
```

Aplica para:

- `POST /api/eventos`

## ID usado para validar

```text
8bc0bd7c-b7f0-586f-8c69-db0bb9c4e900
```

En el ambiente actual este ID esta registrado como usuario y entidad monitoreada:

```json
{
  "usuario_id": "8bc0bd7c-b7f0-586f-8c69-db0bb9c4e900",
  "entidad_id": "8bc0bd7c-b7f0-586f-8c69-db0bb9c4e900"
}
```

## Flujo esperado

1. Backend registra que un usuario monitorea una entidad.
2. Otro servicio detecta un evento relacionado con esa entidad.
3. Ese servicio manda el evento a `POST /api/eventos`.
4. `iva-monitoreo` crea una notificacion para cada usuario que monitorea esa entidad.
5. Frontend consulta pendientes con `GET /api/notificaciones/{usuario_id}?estado=PENDIENTE`.
6. Cuando el usuario atiende la notificacion, se marca como leida.

Importante:

Si una notificacion se marca como `LEIDA`, ya no sale cuando se consulta:

```http
GET /api/notificaciones/{usuario_id}?estado=PENDIENTE
```

Para verla despues, consultar sin filtro de estado o con:

```http
GET /api/notificaciones/{usuario_id}?estado=LEIDA
```

## Crear o reactivar monitoreo

```http
### Crear o reactivar monitoreo
POST http://161.35.107.123:3011/api/monitoreos
Content-Type: application/json
x-api-key: a9087485730ed01568457f3ab620a2f0cfbc10aaca2a0be3

{
  "usuario_id": "8bc0bd7c-b7f0-586f-8c69-db0bb9c4e900",
  "entidad_id": "8bc0bd7c-b7f0-586f-8c69-db0bb9c4e900"
}
```

Este endpoint es idempotente:

- si no existe, lo crea;
- si ya existe activo, devuelve el monitoreo;
- si existe inactivo, lo reactiva.

## Listar monitoreos activos

```http
### Listar monitoreos activos
GET http://161.35.107.123:3011/api/monitoreos/8bc0bd7c-b7f0-586f-8c69-db0bb9c4e900?estado=activo
x-api-key: a9087485730ed01568457f3ab620a2f0cfbc10aaca2a0be3
```

## Desactivar monitoreo

```http
### Desactivar monitoreo
DELETE http://161.35.107.123:3011/api/monitoreos/8bc0bd7c-b7f0-586f-8c69-db0bb9c4e900/8bc0bd7c-b7f0-586f-8c69-db0bb9c4e900
x-api-key: a9087485730ed01568457f3ab620a2f0cfbc10aaca2a0be3
```

No elimina el documento fisicamente. Cambia el estado a:

```text
inactivo
```


## Consultar notificaciones pendientes

```http
### Ver notificaciones pendientes
GET http://161.35.107.123:3011/api/notificaciones/8bc0bd7c-b7f0-586f-8c69-db0bb9c4e900?estado=PENDIENTE&limit=5&offset=0
x-api-key: a9087485730ed01568457f3ab620a2f0cfbc10aaca2a0be3
```

Respuesta actual:

```json
{
  "exito": true,
  "mensaje": "Notificaciones obtenidas",
  "error": null,
  "datos": [
    {
      "_id": "6a0c38d6-f4b0-4c11-b416-97ddd101d3bf",
      "entidad_id": "8bc0bd7c-b7f0-586f-8c69-db0bb9c4e900",
      "event_id": "evento-juicio-20260927-1790549838",
      "usuario_id": "8bc0bd7c-b7f0-586f-8c69-db0bb9c4e900",
      "estado": "PENDIENTE",
      "fecha_creacion": "2026-09-27T22:57:18.826Z",
      "fecha_evento": "2026-09-27T18:00:00.000Z",
      "fecha_lectura": null,
      "fuente": "iva-judicial",
      "id_evento_publico": "juicio-20260927-001",
      "rol": "demandado",
      "tipo_evento": "juicio",
      "nombre_canonico_inverso": "VAN DER HENST DIEGUEZ JORGE RAUL"
    }
  ]
}
```

Campos importantes:

| Campo | Uso |
| --- | --- |
| `_id` | ID de la notificacion. Se usa para marcarla como leida. |
| `entidad_id` | Entidad que tuvo movimiento. |
| `nombre_canonico_inverso` | Nombre de la entidad leido desde `entidades`. |
| `event_id` | ID unico del evento recibido por `iva-monitoreo`. |
| `id_evento_publico` | ID del evento que puede abrir el frontend. |
| `tipo_evento` | Tipo del evento. Ejemplo: `juicio`. |
| `rol` | Rol de la entidad en el evento. Ejemplo: `demandado`. |
| `estado` | `PENDIENTE` o `LEIDA`. |

## Consultar todas las notificaciones

```http
### Ver todas las notificaciones del usuario
GET http://161.35.107.123:3011/api/notificaciones/8bc0bd7c-b7f0-586f-8c69-db0bb9c4e900?limit=50&offset=0
x-api-key: a9087485730ed01568457f3ab620a2f0cfbc10aaca2a0be3
```

## Consultar notificaciones leidas

```http
### Ver notificaciones leidas
GET http://161.35.107.123:3011/api/notificaciones/8bc0bd7c-b7f0-586f-8c69-db0bb9c4e900?estado=LEIDA&limit=50&offset=0
x-api-key: a9087485730ed01568457f3ab620a2f0cfbc10aaca2a0be3
```

## Contador de pendientes

```http
### Contador de pendientes
GET http://161.35.107.123:3011/api/notificaciones/8bc0bd7c-b7f0-586f-8c69-db0bb9c4e900/contador
x-api-key: a9087485730ed01568457f3ab620a2f0cfbc10aaca2a0be3
```

Respuesta:

```json
{
  "exito": true,
  "mensaje": "Contador obtenido",
  "error": null,
  "datos": {
    "usuario_id": "8bc0bd7c-b7f0-586f-8c69-db0bb9c4e900",
    "pendientes": 1
  }
}
```

## Marcar una notificacion como leida

```http
### Marcar notificacion como leida
PATCH http://161.35.107.123:3011/api/notificaciones/6a0c38d6-f4b0-4c11-b416-97ddd101d3bf/leida
x-api-key: a9087485730ed01568457f3ab620a2f0cfbc10aaca2a0be3
```

Despues de esto:

- `estado` cambia a `LEIDA`;
- `fecha_lectura` queda con fecha/hora;
- ya no aparece en `GET /api/notificaciones/{usuario_id}?estado=PENDIENTE`.

## Marcar varias notificaciones como leidas

```http
### Marcar varias notificaciones como leidas
PATCH http://161.35.107.123:3011/api/notificaciones/leidas
Content-Type: application/json
x-api-key: a9087485730ed01568457f3ab620a2f0cfbc10aaca2a0be3

{
  "usuario_id": "8bc0bd7c-b7f0-586f-8c69-db0bb9c4e900",
  "notificacion_ids": [
    "6a0c38d6-f4b0-4c11-b416-97ddd101d3bf"
  ]
}
```

## Health checks

```http
### Health
GET http://161.35.107.123:3011/health
```

```http
### Ready Mongo
GET http://161.35.107.123:3011/ready
```

Estos dos endpoints no usan token.
