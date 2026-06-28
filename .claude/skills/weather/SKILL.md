---
name: weather
description: Obtiene información del clima local o de cualquier ciudad usando wttr.in (sin API key). Úsala cuando el usuario pregunte por el clima, temperatura, lluvia, pronóstico, o condiciones del tiempo. Triggers: "clima", "temperatura", "tiempo", "weather", "pronóstico", "lluvia", "¿qué clima hace?", "¿cómo está el tiempo?".
allowed-tools: Bash(curl:*)
---

# weather — Clima local sin API key

Usa `wttr.in` para obtener clima real. Sin instalación, sin API key.

## Cómo usar esta skill

1. Si el usuario no especifica ciudad → usa Lima, Perú por defecto
2. Si el usuario especifica ciudad → úsala en el comando
3. Muestra resumen legible, no dump crudo

## Comandos

```bash
# Clima actual Lima, Perú (default)
curl -s "wttr.in/Lima?format=4"

# Clima actual de una ciudad específica
curl -s "wttr.in/Bogota?format=4"

# Pronóstico completo 3 días (sin color ANSI para leer limpio)
curl -s "wttr.in/Lima,Peru?format=%l:+%c+%t+%h+%w&lang=es"

# Formato JSON para parsear datos específicos (default: Lima, Perú)
curl -s "wttr.in/Lima,Peru?format=j1" | python3 -c "
import json, sys
d = json.load(sys.stdin)
cc = d['current_condition'][0]
area = d['nearest_area'][0]
city = area['areaName'][0]['value']
country = area['country'][0]['value']
temp_c = cc['temp_C']
feels = cc['FeelsLikeC']
humidity = cc['humidity']
desc = cc['lang_es'][0]['value'] if cc.get('lang_es') else cc['weatherDesc'][0]['value']
wind = cc['windspeedKmph']
print(f'Ciudad: {city}, {country}')
print(f'Temperatura: {temp_c}°C (sensación {feels}°C)')
print(f'Condición: {desc}')
print(f'Humedad: {humidity}%')
print(f'Viento: {wind} km/h')
"

# Pronóstico 3 días en JSON (default: Lima, Perú)
curl -s "wttr.in/Lima,Peru?format=j1" | python3 -c "
import json, sys
d = json.load(sys.stdin)
print('=== Pronóstico 3 días ===')
for day in d['weather']:
    date = day['date']
    max_c = day['maxtempC']
    min_c = day['mintempC']
    desc = day['hourly'][4]['lang_es'][0]['value'] if day['hourly'][4].get('lang_es') else day['hourly'][4]['weatherDesc'][0]['value']
    print(f'{date}: {min_c}°C – {max_c}°C | {desc}')
"
```

## Flujo estándar

1. Pregunta si el usuario quiere clima local o de una ciudad específica (si no está claro)
2. Corre el comando JSON con python3 para parsear
3. Presenta resultado limpio en texto, no código ni dumps
4. Si el usuario pide pronóstico, agrega los 3 días

## Formatos rápidos de wttr.in

| Format code | Output |
|---|---|
| `%l` | Ubicación |
| `%c` | Ícono del clima |
| `%t` | Temperatura |
| `%h` | Humedad |
| `%w` | Viento |
| `%C` | Descripción del clima |
| `format=3` | Una línea: ubicación + temp + condición |
| `format=4` | Una línea: ubicación + condición + temp |
| `format=j1` | JSON completo |

## Errores comunes

- `curl: (6) Could not resolve host` → sin internet
- JSON vacío o error → ciudad no reconocida, prueba nombre en inglés
- Datos en inglés → agrega `&lang=es` al URL o parsea `lang_es` en JSON
