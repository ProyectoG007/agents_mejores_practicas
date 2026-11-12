# Pendiente: videos propios optimizados para que se vean completos

**Estado:** diseño hecho, implementación en espera.
**Depende de:** definir en qué redes se publica y acceso a las métricas de cada una; opcionalmente, el servidor Contabo ("contab") para alojar el proceso.
**Fecha de la nota:** 2026-11-12
**Origen:** idea 8 del video de TikTok. Según el video, cada contenido recibe puntajes de probabilidad de saltarlo, terminarlo, darle like, comentar y seguir. No está verificado, pero coincide con lo que las plataformas dicen sobre retención. Ver `agente-deteccion-abandono.md`.

## Objetivo

Que cada video de los proyectos retenga más: que menos gente lo salte en los primeros segundos y que más gente lo termine. Se logra con un guion, un formato y un proceso de medición repetibles, no con trucos.

## Diseño

```
Idea ──► Guion con estructura fija ──► Producción (cámara + pantalla + subtítulos)
                                                │
                                                ▼
                               Publicación (con hora de la idea 5)
                                                │
Métricas por video ──► Panel de retención ──► Qué ganchos y formatos funcionan ──► próximas ideas
```

### 1. Estructura del guion (35 a 90 segundos)

| Tramo | Duración | Qué hace |
|---|---|---|
| Gancho | 0 a 3 s | Una afirmación concreta o una pregunta que el espectador quiere ver resuelta. Sin saludo ni presentación |
| Promesa | 3 a 8 s | Qué va a saber al final y por qué le importa |
| Desarrollo | el resto | Puntos cortos, cada uno con una prueba o ejemplo visible; cambio de plano o de pantalla cada 2 a 4 segundos |
| Cierre | últimos 5 s | Un solo pedido (seguir, comentar con una pregunta concreta, guardar) |

Ejemplo de gancho del video analizado: "Hice ingeniería inversa de TikTok para entender por qué es tan adictivo, y encontré más de 25 modelos de IA corriendo en tu teléfono". Es concreto, tiene un número y plantea una pregunta.

### 2. Formato visual

- Subtítulos grandes, palabra por palabra, con la palabra clave resaltada en color. Muchas personas ven sin sonido.
- Cámara a la cara o a la pantalla, alternando; sin planos largos sin cambio.
- Pruebas visibles: código, captura, dato en pantalla, no solo la afirmación.
- Vertical 9:16, 720x1280 como mínimo.

### 3. Producción asistida

- **Transcripción para subtítulos:** el mismo método usado en esta sesión (Whisper con sherpa-onnx, modelo descargado desde GitHub) genera la transcripción con tiempos. Se puede correr en Contabo.
- **Armado del video:** HyperFrames o Canva, que ya están conectados, para subtítulos animados y cortes.
- **Publicación:** la herramienta Postiz o la de cada red; la hora se elige con el modelo de la idea 5.

### 4. Medición por video

Cada publicación guarda, a las 24 h y a los 7 días:

- Porcentaje que pasa de los primeros 3 segundos (es el gancho).
- Curva de retención: en qué segundo se va la gente.
- Porcentaje que lo termina y tiempo promedio visto.
- Guardados, compartidos, comentarios y seguidores ganados.

### 5. Aprendizaje

Una tabla con una fila por video: tipo de gancho, duración, formato, tema y resultados. Cada semana se revisa:

- Qué ganchos pasan más del 3.er segundo.
- En qué tramos cae la curva y qué había ahí.
- Qué duración rinde mejor por tema.

Con esa tabla se decide qué grabar la semana siguiente. Un punto de caída repetido suele mostrar un tramo para cortar o reescribir.

## Cómo medir si sirve

Comparar las primeras 4 a 6 semanas con la nueva estructura contra las 4 a 6 anteriores, con temas parecidos:

- Porcentaje que pasa los primeros 3 segundos.
- Porcentaje que termina el video.
- Seguidores ganados por video.
- Visitas al proyecto o clics en el enlace, que es lo que importa de verdad.

## Límites

- No prometer lo que el video no muestra: el gancho tiene que cumplirse, o se pierde confianza y el espectador lo abandona al final.
- No presentar como hecho lo que no se pudo verificar; el propio video analizado no está verificado.
- Respetar derechos de imagen y de música de cada red.
- Si el video usa datos de personas o clientes, pedir permiso y anonimizar.

## Orden de construcción

1. Definir redes y recoger las métricas de los últimos videos publicados (línea de base).
2. Plantilla de guion con la estructura fija y una lista de ganchos por probar.
3. Plantilla de subtítulos y de video para repetir rápido.
4. Tabla de aprendizaje y revisión semanal.
5. Automatizar transcripción, subtítulos y publicación.

## Qué revisar cuando haya acceso

- [ ] En qué redes se publica hoy y con qué frecuencia.
- [ ] Qué métricas entrega cada red y cómo exportarlas (por API o manual).
- [ ] Si ya hay herramienta de publicación o calendario.
- [ ] Qué videos propios existen y cómo rindieron, para la línea de base.
- [ ] Si Contabo tiene capacidad para transcribir y renderizar o conviene hacerlo fuera.
