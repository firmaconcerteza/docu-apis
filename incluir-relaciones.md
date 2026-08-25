# Bandera opcional `incluirRelaciones`

Esta bandera permite consultar información adicional de demandas/juicios y relaciones personales sin romper la respuesta actual de los endpoints.

## Regla de compatibilidad

- Si `incluirRelaciones` no se envía, la respuesta se mantiene igual que antes.
- Si `incluirRelaciones` viene en `false`, la respuesta se mantiene igual que antes.
- Solo si `incluirRelaciones=true`, el servicio agrega la información adicional disponible.

## Endpoint de detalle

```http
GET /detalle/{id}?incluirRelaciones=true
```

Ejemplo:

```bash
curl -X GET "https://ext.firmaconcerteza.com/detalle/14584ece-d734-531f-942a-1ab927563669?incluirRelaciones=true" \
  -H "x-api-key: *****"
```

### Sin bandera

El detalle no incluye:

- demandas / juicios
- relaciones personales de padre
- relaciones personales de madre
- relaciones personales de conyuge

Esto conserva el formato que ya consume el front.

### Con `incluirRelaciones=true`

El detalle devuelve todo lo anterior mas la informacion adicional.

Las relaciones personales se agregan en la tabla de relaciones existente:

```text
atributos[].titulo: Relaciones
seccion: Personas
subseccion: Activas
campo: personas_activas
fuente: gt-relaciones
tipo: tabla
```

Ejemplo de fila dentro de `valor`:

```json
[
  {
    "Tipo Relación": "Madre",
    "Nombre": "NOMBRE DE LA PERSONA",
    "_id": "uuid-relacionado"
  }
]
```

Las demandas/juicios se agregan en:

```text
atributos[].titulo: Estado
seccion: Demandas
subseccion: Demandas
campo: demandas
fuente: gt-juicios
tipo: tabla
```

Ejemplo de campo:

```json
{
  "campo": "demandas",
  "display": "Demandas",
  "fuente": "gt-juicios",
  "tipo": "tabla",
  "valor": "[{\"Numero\":\"...\",\"Organo\":\"...\",\"Estado\":\"...\"}]"
}
```

> Nota: cuando `tipo` es `tabla`, el campo `valor` viene como string JSON y debe parsearse con `JSON.parse(campo.valor)`.

## Endpoint de relaciones

```http
POST /relaciones
```

Ejemplo sin bandera:

```bash
curl -X POST "https://ext.firmaconcerteza.com/relaciones" \
  -H "Content-Type: application/json" \
  -H "x-api-key: *****" \
  -d '{
    "entityId": "8fa458f1-fdf1-59c0-af1c-ab8fffce7195",
    "depth": 1
  }'
```

Ejemplo con bandera:

```bash
curl -X POST "https://ext.firmaconcerteza.com/relaciones" \
  -H "Content-Type: application/json" \
  -H "x-api-key: *****" \
  -d '{
    "entityId": "8fa458f1-fdf1-59c0-af1c-ab8fffce7195",
    "depth": 1,
    "incluirRelaciones": true
  }'
```

### Comportamiento

Sin bandera o con `incluirRelaciones=false`, el grafo conserva la respuesta original y no incluye relaciones especiales de:

- `demandante`
- `demandado`
- `padre`
- `madre`
- `conyuge`

Con `incluirRelaciones=true`, el grafo incluye esas relaciones cuando existan en Arango.

Ejemplo resumido:

```json
{
  "nodes": [
    {
      "id": "uuid",
      "nombre": "Entidad principal",
      "tipo": "natural"
    }
  ],
  "relations": [
    {
      "source": "uuid-origen",
      "target": "uuid-destino",
      "tipo": "madre",
      "fuente": "gt-tabla-familiares"
    }
  ],
  "metadata": {
    "incluirRelaciones": true
  }
}
```

## Parametro soportado

| Campo | Tipo | Requerido | Default | Descripcion |
| --- | --- | --- | --- | --- |
| `incluirRelaciones` | boolean | No | `false` | Si es `true`, agrega demandas/juicios y relaciones personales. Si se omite o es `false`, devuelve la respuesta original. |

## Resumen para integracion

Para no romper el front actual, no enviar la bandera o enviarla en `false`.

Para obtener demandas y familiares, enviar:

```json
{
  "incluirRelaciones": true
}
```

En detalle, las relaciones familiares salen en la misma tabla de relaciones que ya existe. Las demandas salen en la pestana `Estado`, seccion `Demandas`.
