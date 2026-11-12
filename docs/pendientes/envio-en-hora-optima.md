# Pendiente: envío por email y WhatsApp en la hora en que cada persona responde

**Estado:** diseño hecho, implementación en espera.
**Depende de:** acceso al servidor Contabo ("contab") y a los canales de envío.
**Fecha de la nota:** 2026-11-12
**Origen:** idea 5 del video de TikTok (predecir cuándo abre la app cada usuario). Ver `agente-deteccion-abandono.md`.

## Objetivo

Enviar cada mensaje cuando esa persona tiene más probabilidad de leerlo y responder, en vez de enviar a todos a la misma hora. No se busca enviar más, sino enviar mejor.

## Diseño

```
campaña programada (ventana de envío, p. ej. 48 h)
      │
      ▼
Planificador ──► por contacto: elegir hora dentro de la ventana
      │              ▲
      │              └── Modelo de hora por contacto (histograma + respaldo)
      ▼
Cola de envío (con límites y horario permitido) ──► WhatsApp / email
      │
      ▼
Eventos (entregado, leído, clic, respuesta) ──► Registro ──► actualiza el modelo
```

### Qué señal usar por canal

| Canal | Señal confiable | Señal poco confiable |
|---|---|---|
| WhatsApp | Respuestas y lecturas (si el contacto no las desactivó) | — |
| Email | Clics y respuestas | Aperturas: Apple Mail Privacy Protection y otros clientes las inflan o adelantan |

Por eso el modelo se entrena con **respuestas y clics**, no con aperturas de email.

### Modelo de hora por contacto

1. Por contacto se guarda un histograma de 24 horas x 7 días con sus interacciones, en su zona horaria.
2. Se suaviza con el histograma de su segmento y el general, para no decidir con pocos datos:
   `puntaje(hora) = (interacciones_contacto + k × tasa_segmento(hora)) / (total_contacto + k)`.
3. Contactos con menos de unas 5 interacciones usan directamente el segmento; los nuevos, la hora general.
4. Exploración: ~10 % de los envíos van a una hora distinta de la mejor, para no quedarse atascado y detectar cambios de hábito.
5. Más adelante, si hay volumen, reemplazar por un modelo entrenado; empezar con esto es suficiente y fácil de explicar.

### Planificador y cola

- Recibe la campaña con una ventana (por ejemplo, "dentro de las próximas 48 h").
- Elige para cada contacto la mejor hora dentro de esa ventana y respeta:
  - horario permitido (sin enviar de madrugada, domingos opcional);
  - límite de mensajes por contacto por semana;
  - límite de velocidad por canal.
- Si la persona respondió o compró antes de su hora, el envío se cancela.
- Corre como un proceso con cola en el servidor Contabo, sin servicios externos.

### Reglas propias de cada canal

- **WhatsApp:** solo contactos con consentimiento (opt-in). Fuera de la ventana de 24 h desde su último mensaje solo se pueden enviar plantillas aprobadas por Meta. Enviar mal o a quien no lo espera baja la calidad del número y puede limitarlo.
- **Email:** enlace de baja visible, y control de reputación del dominio (rebotes y quejas).

### Registro

Una fila por envío: contacto, canal, hora elegida, motivo (modelo, segmento, exploración), eventos posteriores. Es la base para medir y reentrenar.

## Cómo medir si sirve

Grupo de control: 20 % de los contactos reciben a la hora habitual de siempre.

- Tasa de respuesta o clic contra el control.
- Bajas y quejas (no deben subir).
- Conversiones atribuidas a la campaña.
- Calidad del número de WhatsApp y rebotes de email.

## Límites

- Sin consentimiento no se envía, por más buena que sea la hora.
- No aumentar la frecuencia por tener mejor horario; la meta es la misma cantidad, mejor momento.
- Dejar que la persona elija cuándo y por dónde quiere recibir, y respetarlo por encima del modelo.

## Orden de construcción

1. Registro de envíos y eventos (entregado, leído, clic, respuesta) con la zona horaria de cada contacto.
2. Cola de envío con límites y horario permitido.
3. Histograma por contacto, con respaldo por segmento.
4. Planificador con exploración y grupo de control.
5. Tablero de resultados y, si hay volumen, un modelo entrenado.

## Qué revisar cuando haya acceso a Contabo

- [ ] Con qué se envía hoy WhatsApp (API oficial de Meta o proveedor) y qué plantillas hay aprobadas.
- [ ] Con qué se envía email y dónde queda el historial de eventos.
- [ ] Cuántos contactos hay, cuántos con consentimiento registrado y dónde se guarda.
- [ ] Cuánta historia de interacciones existe para arrancar sin esperar semanas.
- [ ] Si el servidor soporta la cola y el planificador junto a lo que ya corre.
- [ ] Zona horaria de cada contacto: si se guarda o hay que inferirla.
