---
name: clima-local
description: Consulta el clima actual y el pronóstico para una ciudad/ubicación usando servicios gratuitos sin API key (wttr.in y Open-Meteo). Úsala cuando el usuario pida el clima, temperatura, pronóstico del tiempo, o info meteorológica de un lugar.
---

# Clima Local

Skill para obtener información del clima (temperatura actual, condición, sensación
térmica, viento, humedad y pronóstico corto) directamente desde la terminal, sin
necesidad de registrar ninguna API key.

## Cuándo usar esta skill

- El usuario pregunta por el clima, temperatura o pronóstico de una ciudad.
- El usuario pide "¿cómo está el clima hoy?", "dame el pronóstico de X", etc.
- No se necesita conexión a servicios de pago ni claves: todo funciona con
  endpoints públicos y gratuitos.

## Ubicación

- Si el usuario da una ciudad/lugar explícito, úsalo tal cual (ej. `Bogota`,
  `Mexico City`, `Madrid,ES`).
- Si no da ninguna ubicación, usa `wttr.in` sin nombre de ciudad: detecta la
  ubicación aproximada por IP automáticamente.

## Método 1 (preferido): wttr.in

Es el método más simple, no requiere parseo de JSON y da salida ya formateada
en texto plano.

Resumen de una línea (rápido, ideal para respuestas cortas):

```bash
curl.exe -s "wttr.in/<CIUDAD>?format=%l:+%c+%t+(sensacion+%f)+humedad:%h+viento:%w"
```

Reporte con más detalle (3 días, sin animación de terminal, ancho reducido):

```bash
curl.exe -s "wttr.in/<CIUDAD>?0Q" 
```

- `?0` = solo el día actual (usa `?1` o `?2` para más días de pronóstico).
- `?Q` = quita el mensaje de ayuda al final.
- `?m` fuerza unidades métricas si el resultado sale en Fahrenheit/millas.
- Reemplaza espacios en el nombre de la ciudad con `+` o enciérralo en comillas
  y usa `%20`, ej: `wttr.in/Ciudad+de+Mexico`.

En PowerShell usa siempre `curl.exe` (no el alias `curl`, que en PowerShell
apunta a `Invoke-WebRequest` y no soporta los mismos flags).

## Método 2 (respaldo): Open-Meteo

Si `wttr.in` no responde o el usuario necesita datos estructurados (JSON) para
procesarlos, usa Open-Meteo, también gratuito y sin API key:

1. Geocodificar el nombre de la ciudad:

```bash
curl.exe -s "https://geocoding-api.open-meteo.com/v1/search?name=<CIUDAD>&count=1&language=es&format=json"
```

Extrae `latitude` y `longitude` del primer resultado.

2. Pedir el clima actual con esas coordenadas:

```bash
curl.exe -s "https://api.open-meteo.com/v1/forecast?latitude=<LAT>&longitude=<LON>&current=temperature_2m,relative_humidity_2m,apparent_temperature,wind_speed_10m,weather_code&timezone=auto"
```

El campo `current.weather_code` es un código WMO; tradúcelo a una descripción
legible (0 = despejado, 1-3 = parcialmente nublado, 45/48 = niebla, 51-67 =
llovizna/lluvia, 71-77 = nieve, 80-82 = chubascos, 95-99 = tormenta).

## Presentación de la respuesta

Responde en español, de forma breve, con: ubicación, condición, temperatura
actual, sensación térmica, y humedad/viento si están disponibles. No muestres
el JSON crudo ni el comando ejecutado salvo que el usuario lo pida.
