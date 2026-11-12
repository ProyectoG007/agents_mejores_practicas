# Pendiente: agente que detecta frustración y abandono

**Estado:** diseño hecho, implementación en espera.
**Bloqueado por:** acceso completo al sistema (incluido "contab").
**Fecha de la nota:** 2026-11-12

## Origen

Idea tomada de un video de divulgación (1:29) sobre ingeniería inversa de la app de TikTok.
Las afirmaciones del video no están verificadas; se usan como inspiración.

Lo que describe el video:

- Cada video llega con puntajes separados: probabilidad de saltar, terminar, dar like, comentar, seguir.
- Modelos en el teléfono estiman el estado del usuario (quieto, caminando, acostado, en auto) y mandan batería, carga, brillo, mute y más de 50 variables con cada pedido.
- Un modelo predice si el usuario va a saltar los videos ya cargados; si es así, pide un "lote de emergencia".
- Si sigue saltando, lo marca como "lost focus" y como último recurso inserta un video en vivo.
- Otros modelos detectan somnolencia y predicen la próxima apertura de la app para precargarla.

## Diseño

Objetivo: resolver más conversaciones, no retener al usuario.

```
mensaje del usuario
      │
      ├──► Observador (señales + modelo chico) ──► frustración, riesgo de abandono, progreso
      │                                                   │
      │                                          Selector de estrategia (reglas)
      │                                                   │
      └──► Agente principal ◄── instrucción: normal / rescate / último recurso
                  │
              respuesta ──► registro del turno + resultado final
```

### Observador (corre en cada mensaje, en paralelo)

Señales sin IA:

- repite o reformula la misma pregunta;
- mensajes cada vez más cortos;
- mayúsculas, signos repetidos, palabras de enojo;
- pide "humano", "asesor" o "persona";
- muchos turnos sin avanzar;
- tarda cada vez más en contestar.

Clasificador con modelo chico: lee los últimos 4 a 6 turnos y devuelve
`frustracion`, `riesgo_abandono` y `progreso`, cada uno de 0 a 1.

### Selector de estrategia

Umbrales iniciales, a ajustar con datos reales.

| Nivel | Cuándo | Qué hace el agente |
|---|---|---|
| Normal | riesgo < 0,4 | Conversa como siempre |
| Rescate | riesgo 0,4 a 0,7, o pregunta repetida | Reconoce el problema en una línea, no repite, responde corto, ofrece 2 o 3 opciones o una sola pregunta |
| Último recurso | riesgo > 0,7, pide humano, o dos rescates fallidos | Ofrece pasar a una persona o un llamado, con resumen del caso |

### Agente principal

Mismo prompt base. Recibe una instrucción adicional por turno según el nivel.

### Registro

Una fila por turno: señales, puntajes, nivel elegido. Al cerrar: resuelto, abandonó o escaló.

### Seguimiento posterior

Si el usuario se fue sin resolver y dio consentimiento: un único mensaje, a la hora en que suele responder, con resumen y siguiente paso.

## Recomendación de arranque (sin conocer todavía el sistema)

- Canal: WhatsApp primero, web después.
- Registro de turnos: una tabla en la base de datos que ya se use.
- Observador: modelo chico y barato; el agente principal queda como esté.
- Primera entrega: solo el registro de turnos y resultados, sin cambiar el comportamiento del agente.

## Orden de construcción

1. Registro de turnos y resultados.
2. Señales sin IA y los tres niveles con reglas.
3. Clasificador con modelo chico.
4. Seguimiento posterior y prueba comparativa contra el agente actual.

## Métricas

- Tasa de resolución.
- Tasa de abandono.
- Turnos hasta resolver.
- Escalaciones a humano (y si llegan con contexto).

## Límites

- No ocultar la opción de hablar con una persona.
- No analizar texto que el usuario no envió.
- Avisar en la política de privacidad que se analiza el tono de la conversación.

## Qué revisar cuando haya acceso completo

- [ ] Por qué canal conversa hoy el agente y con qué está construido.
- [ ] Dónde se guardan hoy las conversaciones y si hay historial para calibrar umbrales.
- [ ] Si existe un circuito de derivación a una persona y quién atiende.
- [ ] Qué datos de "contab" necesita el agente para resolver (saldos, facturas, estados) y con qué permisos.
- [ ] Qué consentimiento hay para mensajes de seguimiento.
- [ ] Volumen de conversaciones por día, para estimar costo del observador.

## Otras ideas del mismo video, sin diseñar todavía

1. Feed de contenido: puntajes por ítem y lote de rescate.
2. E-commerce: probabilidad de ver, agregar al carrito y comprar; cambiar la vidriera.
3. Aprendizaje: detectar pérdida de foco y cambiar de formato.
4. Agente o chatbot: este documento.
5. Email o WhatsApp marketing: enviar a la hora en que cada persona abre.
6. App móvil: precarga y adaptación a batería, conexión y mute.
7. Tablero interno: precalcular el reporte que cada persona abre a cada hora.
8. Videos propios: gancho en 2 segundos, subtítulos grandes, optimizar para que se terminen.
