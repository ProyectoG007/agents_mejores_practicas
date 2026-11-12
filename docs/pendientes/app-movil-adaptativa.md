# Pendiente: app móvil que se adapta al contexto y precarga lo probable

**Estado:** diseño hecho, implementación en espera.
**Depende de:** acceso al servidor Contabo ("contab") y al código de la app móvil.
**Fecha de la nota:** 2026-11-12
**Origen:** idea 6 del video de TikTok (contexto del dispositivo en cada pedido, precarga, mantener la app lista). Ver `agente-deteccion-abandono.md`.

## Objetivo

Que la app se sienta rápida y se comporte según la situación de la persona (poca batería, mala conexión, en silencio, en movimiento), usando solo señales que el sistema operativo ya permite leer sin permisos especiales.

## Diseño

```
Cliente (app)                                      Servidor (Contabo)
─────────────                                      ──────────────────
Lector de contexto ──► encabezado "contexto" ──►   API
  (batería, conexión, mute,                          │
   ahorro de datos, hora)                            ▼
        │                                     Adaptador de respuesta
        ▼                                      (tamaño, calidad, cantidad)
Predictor de próxima pantalla ──► precarga
  (frecuencias locales)                        Registro de uso (anónimo)
```

### 1. Contexto del dispositivo (solo lo necesario)

Se envía en cada pedido, resumido y sin identificar a la persona:

| Señal | Para qué |
|---|---|
| Nivel de batería y modo ahorro | Menos animaciones, sin precarga pesada, menos actualizaciones en segundo plano |
| Tipo de conexión (wifi, datos, lenta) | Imágenes y video más livianos, lotes más chicos |
| Modo silencio o mute | Subtítulos activados por defecto, notificaciones sin sonido |
| Ahorro de datos del sistema | Respetarlo siempre por encima de cualquier otra regla |
| Hora local | Tema oscuro nocturno, evitar avisos fuera de horario |

No se usan giroscopio ni sensores de movimiento: el valor es bajo y el costo en privacidad y permisos es alto.

### 2. Adaptador de respuesta (servidor)

Reglas simples que ajustan la respuesta de la API según el contexto: calidad de imagen, tamaño de página, si se incluyen campos pesados. Empezar con reglas; no hace falta un modelo.

### 3. Predictor de próxima pantalla (en el dispositivo)

- Cuenta, por pantalla actual y franja horaria, a qué pantalla va la persona después.
- Si una transición supera ~50 % de probabilidad y el contexto lo permite (wifi, batería suficiente), precarga los datos de esa pantalla.
- Los datos se guardan solo en el dispositivo; el servidor recibe únicamente cuántas precargas se aprovecharon, para medir.

### 4. App lista al abrir

Si la persona abre la app casi siempre a la misma hora, se refresca el contenido en segundo plano poco antes, con el mecanismo que el sistema operativo ofrece y solo con batería y conexión adecuadas. No se mantiene la app despierta de forma forzada.

### 5. Registro de uso

Anónimo y agregado: tiempo hasta primera pantalla útil, tasa de aciertos de la precarga, consumo de datos y batería por sesión.

## Cómo medir si sirve

Mitad de usuarios con adaptación y mitad sin ella:

- Tiempo hasta primera pantalla útil.
- Aciertos de precarga frente a datos descargados de más (no debe haber desperdicio alto).
- Consumo de datos y batería.
- Retención y uso de la app.

## Límites

- Mínimas señales: solo las que cambian algo concreto en la experiencia.
- Respetar siempre el ahorro de datos y de batería del sistema.
- Nada de rastrear al usuario con estas señales; no se guardan ligadas a su identidad.
- Informar en la política de privacidad qué se envía y para qué.
- Nunca usar el contexto (por ejemplo, "se está durmiendo") para retener a la persona.

## Orden de construcción

1. Encabezado de contexto en las llamadas a la API y registro de tiempos de carga.
2. Adaptador de respuesta por reglas (conexión y ahorro de datos primero).
3. Mute: subtítulos y notificaciones silenciosas.
4. Predictor de próxima pantalla y precarga con control de desperdicio.
5. Refresco en segundo plano antes de la hora habitual.

## Qué revisar cuando haya acceso a Contabo

- [ ] Qué app móvil hay (nativa, React Native, Flutter, PWA) y en qué plataformas.
- [ ] Qué API usa y si puede aceptar un encabezado extra y ajustar respuestas.
- [ ] Qué mide hoy de rendimiento (tiempos de carga, errores) y dónde queda.
- [ ] Cuánto pesan hoy las pantallas más usadas (imágenes, video, datos).
- [ ] Si hay notificaciones push y con qué servicio.
- [ ] Capacidad de Contabo para soportar el tráfico extra de precargas.
