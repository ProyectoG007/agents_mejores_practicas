# Plan: 8 ideas de producto inspiradas en el video sobre el algoritmo de TikTok

**Estado:** diseños hechos, implementación en espera.
**Bloqueado por:** acceso completo al servidor Contabo ("contab"), donde corren los sistemas. La idea 8 se puede empezar antes.
**Fecha:** 2026-11-12

## 1. Origen

Video de divulgación (1:29, vertical) que afirma haber hecho ingeniería inversa de la app de TikTok. Transcripción completa obtenida en la sesión. **Nada de esto está verificado**; se usa como inspiración, no como documentación oficial.

Lo que afirma el video:

- Cada video recibe antes de mostrarse puntajes separados: probabilidad de saltarlo, terminarlo, darle like, comentar y seguir al creador.
- Más de 25 modelos corren en el teléfono y estiman el estado de la persona: quieta, caminando, acostada, en auto (giroscopio), más batería, carga, brillo, mute y más de 50 variables, enviadas con cada pedido de videos.
- Un modelo predice si va a saltar los videos ya cargados; si es así, el teléfono pide un "lote de emergencia".
- Si sigue saltando, la marca como "lost focus" y como último recurso inserta un video en vivo.
- Otros modelos detectan somnolencia y predicen la próxima apertura de la app para dejarla lista.
- Según el video, incluso un texto escrito y no enviado se recolecta.

## 2. Principios comunes a todas las ideas

**Tres mecanismos transferibles:**

1. **Predecir varios resultados por separado** (saltar, terminar, valorar, actuar) en vez de un único puntaje.
2. **Contexto barato en cada decisión** (hora, conexión, batería, mute, historial reciente).
3. **Reaccionar cuando se está perdiendo a la persona** (cambiar de estrategia, no insistir).

**Qué no se copia:**

- Leer sensores de movimiento: poco valor y alto costo en permisos y privacidad.
- Recolectar texto no enviado.
- Detectar somnolencia para retener. En un producto propio se hace lo opuesto: sugerir cortar.
- Ocultar la salida (hablar con una persona, cancelar, parar).

**Base técnica compartida: un único registro de eventos.** Casi todos los diseños empiezan igual, así que se construye una sola vez:

| Campo | Ejemplo |
|---|---|
| Quién | Persona o rol, anonimizable |
| Qué | Evento: mostrado, abierto, saltado, terminado, respondido, comprado, error |
| Sobre qué | Ítem, reporte, producto, mensaje |
| Cuándo | Fecha y hora con zona horaria del contacto |
| Contexto | Canal, dispositivo, conexión, hora local (solo lo necesario) |
| Resultado | Lo que pasó después (resuelto, abandonó, convirtió) |

Con eso se alimentan los 8 diseños. Es el primer entregable, sin cambiar ningún comportamiento todavía.

**Método de validación común:** siempre un grupo de control (10 a 20 %) con el comportamiento actual, y se compara contra él. Sin control no se sabe si algo mejoró.

**Alojamiento:** por defecto en Contabo, con la base y la cola que ya existan, y sin servicios externos. Se confirma cuando se vea el servidor.

## 3. Orden recomendado

| Prioridad | Idea | Esfuerzo | Por qué en ese lugar |
|---|---|---|---|
| Ya | 8. Videos propios | Bajo | No depende de Contabo; se puede empezar hoy |
| 1.º | Registro de eventos común | Bajo | Habilita todo lo demás |
| 2.º | 7. Tablero interno (pasos 1 y 2) | Bajo | Mide qué reportes frenan al equipo; valor inmediato |
| 3.º | 4. Agente que detecta abandono | Medio | Mejora directa de la atención al cliente |
| 4.º | 5. Envío en hora óptima | Medio | No necesita IA para empezar; resultado medible rápido |
| 5.º | 6. App móvil adaptativa | Medio | Depende del tipo de app |
| 6.º | 1, 2 y 3 | Medio-alto | Dependen de que exista el producto correspondiente y su historial |

Es una recomendación sin haber visto el sistema; se ajusta cuando haya acceso.

---

## 4. Diseño 1: feed de contenido (videos, noticias, posteos, cursos)

**Objetivo:** ordenar el contenido para que cada persona vea primero lo que tiene más probabilidad de valorar, y recuperarla cuando empieza a saltar todo.

```
Candidatos ──► Puntajes por ítem ──► Ordenador ──► Lote para la persona
                      ▲                    │
                      │                    ▼
              Registro de eventos ◄── Vigilante de saltos ──► lote de rescate
```

**Puntajes por ítem.** Para cada par persona-ítem se estiman por separado: `p_saltar`, `p_terminar`, `p_valorar` (like o guardar), `p_actuar` (comentar, compartir, seguir). Empezar con tasas históricas por ítem y por segmento, suavizadas para ítems con pocos datos; un modelo entrenado viene después.

**Ordenador.** `valor = w1·p_terminar + w2·p_valorar + w3·p_actuar − w4·p_saltar`, con pesos fijados por el negocio, más:

- diversidad (no más de 2 seguidos del mismo tema o autor);
- novedad para ítems nuevos, que no tienen historial;
- 10 % de exploración para detectar cambios de gusto.

**Vigilante de saltos.** Si en una sesión hay una racha (por ejemplo 5 saltos seguidos o tiempo mediano menor a 2 s), se pide un lote de rescate:

- más variedad de temas;
- ítems con alta tasa de finalización general;
- mezcla de formatos cortos.

Si la racha continúa, se ofrece algo explícito ("¿qué querés ver?") en vez de insistir.

**Medir:** finalización, tiempo útil por sesión, retorno a los 7 días, quejas. Con grupo de control.

**Límites:** sin scroll infinito forzado; recordatorio de pausa tras sesiones largas; no usar señales de cansancio para retener; el usuario puede ver y ajustar sus temas.

**Orden:** (1) registro de impresión, tiempo visto, terminado y valoración; (2) tasas por ítem y segmento; (3) ordenador con pesos y diversidad; (4) vigilante y lote de rescate; (5) modelo entrenado si hay volumen.

---

## 5. Diseño 2: e-commerce o catálogo

**Objetivo:** mostrar primero los productos con más probabilidad de ser vistos, agregados al carrito y comprados, y cambiar de enfoque cuando la persona pasa de largo.

```
Productos ──► Probabilidades por persona/segmento ──► Vidriera ordenada
                    ▲                                    │
              Registro (vistas, carrito, compra) ◄───────┘
                    │
              Vigilante de sesión ──► cambio de categoría o rango de precio
```

**Probabilidades por producto y segmento:** `p_clic`, `p_carrito`, `p_compra`. Empezar con tasas históricas suavizadas por segmento (por ejemplo: nuevo, recurrente, dispositivo).

**Ordenador.** Incluye siempre las restricciones del negocio, no solo la probabilidad:

`valor = p_compra × margen × disponibilidad − costo de devolución esperado`

- No promocionar lo que no hay en stock.
- Mantener variedad de categorías.
- Producto nuevo con exposición mínima asegurada.

**Vigilante de sesión.** Tras varios productos vistos sin clic, cambiar de enfoque: otra categoría, rango de precio cercano a lo que sí miró, o productos más vendidos. Un carrito abandonado dispara un recordatorio único, en la hora que dicta el diseño 5.

**Medir:** conversión, ticket promedio, devoluciones, ingresos por visita; contra un grupo de control.

**Límites:**

- Sin urgencia ni escasez falsas.
- Sin precios distintos por persona sin aviso.
- No ocultar el orden por precio o relevancia si la persona lo pide.
- Respetar leyes de consumo y de protección de datos.

**Orden:** (1) registro de vistas, clics, carrito y compra con producto y segmento; (2) tasas por producto y segmento; (3) ordenador con margen y stock; (4) vigilante y recordatorio de carrito; (5) modelo entrenado.

---

## 6. Diseño 3: app de aprendizaje

**Objetivo:** detectar cuándo la persona pierde el foco o se atasca y cambiar el formato, para que aprenda más, no para que dure más tiempo.

```
Lección en curso ──► Señales de foco y dominio ──► Estado: bien / atascado / cansado
                                                          │
                                                          ▼
                                            Selector de formato ──► siguiente actividad
```

**Señales:**

- tiempo sin tocar la pantalla, saltos de lección, vuelta atrás;
- errores repetidos en el mismo tema;
- respuestas cada vez más rápidas o más lentas;
- hora y duración de la sesión.

**Estimación de dominio por tema:** porcentaje de aciertos ponderado por recencia y dificultad; repaso espaciado para lo que se olvida.

**Selector de formato:**

| Estado | Qué hace |
|---|---|
| Bien | Sigue el plan, un poco más difícil |
| Atascado | Explicación en otro formato (texto, ejemplo, video corto), paso más chico, pista |
| Cansado | Quiz corto o actividad liviana, y sugerencia de pausa si la sesión es larga |

**Medir:** lo que importa es el aprendizaje real: resultado de un examen de repaso a los 7 días, no el tiempo en la app. También finalización de módulos y abandono.

**Límites:** nada de mecánicas para alargar el uso a costa del descanso; cuidado especial con datos de menores; el estudiante puede ver y cambiar su plan; las pausas se sugieren, no se castigan.

**Orden:** (1) registro de aciertos, errores, tiempos y abandonos por lección; (2) dominio por tema y repaso espaciado; (3) reglas de estado (bien, atascado, cansado); (4) selector de formato; (5) examen de repaso y panel de resultados.

---

## 7. Diseño 4: agente que detecta frustración y abandono

**Objetivo:** resolver más conversaciones. Un observador barato mira cada mensaje, calcula el riesgo de que la persona se frustre o se vaya, y le indica al agente principal qué estrategia usar.

```
mensaje ──► Observador (señales + modelo chico) ──► frustración, riesgo, progreso
   │                                                      │
   │                                          Selector de estrategia (reglas)
   │                                                      │
   └──► Agente principal ◄── instrucción: normal / rescate / último recurso
              │
          respuesta ──► registro del turno + resultado final
```

**Observador, en cada mensaje y en paralelo:**

- Señales sin IA: repite o reformula la misma pregunta; mensajes cada vez más cortos; mayúsculas, signos repetidos, palabras de enojo; pide "humano", "asesor" o "persona"; muchos turnos sin avanzar; tarda cada vez más en contestar.
- Clasificador con modelo chico (por ejemplo Haiku 5.5): lee los últimos 4 a 6 turnos y devuelve `frustracion`, `riesgo_abandono` y `progreso`, de 0 a 1.

**Selector de estrategia.** Umbrales iniciales, a ajustar con datos reales:

| Nivel | Cuándo | Qué hace el agente |
|---|---|---|
| Normal | riesgo < 0,4 | Conversa como siempre |
| Rescate | 0,4 a 0,7, o pregunta repetida | Reconoce el problema en una línea, no repite, responde corto, ofrece 2 o 3 opciones o una sola pregunta |
| Último recurso | > 0,7, pide humano o dos rescates fallidos | Ofrece pasar a una persona o un llamado, con resumen del caso |

**Agente principal:** mismo prompt base; recibe una instrucción adicional por turno según el nivel.

**Registro:** una fila por turno con señales, puntajes y nivel; al cerrar, resuelto, abandonó o escaló.

**Seguimiento:** si se fue sin resolver y dio consentimiento, un único mensaje a la hora en que suele responder (ver idea 5), con resumen y siguiente paso.

**Medir:** tasa de resolución, abandono, turnos hasta resolver, escalaciones (que lleguen con contexto). Con control.

**Límites:** nunca ocultar la opción de hablar con una persona; no analizar texto no enviado; avisar en la política de privacidad que se analiza el tono.

**Orden:** (1) registro de turnos y resultados; (2) señales sin IA y tres niveles; (3) clasificador con modelo chico; (4) seguimiento y prueba comparativa.

**Recomendación de arranque:** canal WhatsApp primero y web después; registro en la base que ya haya en Contabo; primera entrega solo de registro, sin cambiar el comportamiento del agente.

---

## 8. Diseño 5: envío por email y WhatsApp en la hora en que cada persona responde

**Objetivo:** mandar el mismo mensaje cuando esa persona tiene más probabilidad de leerlo y responder; no enviar más, sino mejor.

```
campaña con ventana (ej. 48 h) ──► Planificador ◄── modelo de hora por contacto
        ▼
Cola de envío (límites + horario permitido) ──► WhatsApp / email
        ▼
Eventos (entregado, leído, clic, respuesta) ──► registro ──► actualiza el modelo
```

**Señal por canal:**

| Canal | Confiable | Poco confiable |
|---|---|---|
| WhatsApp | Respuestas y lecturas | Lecturas si el contacto las desactivó |
| Email | Clics y respuestas | Aperturas: Apple Mail Privacy Protection las infla |

**Modelo de hora por contacto:**

1. Histograma de 24 h × 7 días por contacto, en su zona horaria.
2. Suavizado con el de su segmento: `puntaje(hora) = (interacciones_contacto + k × tasa_segmento(hora)) / (total_contacto + k)`.
3. Con menos de unas 5 interacciones se usa el segmento; contactos nuevos, la hora general.
4. 10 % de exploración a otra hora.

**Planificador y cola:** elige la hora dentro de la ventana y respeta horario permitido, máximo semanal por contacto y velocidad por canal. Cancela el envío si la persona respondió o compró antes.

**Reglas por canal:**

- WhatsApp: solo con consentimiento; fuera de las 24 h desde su último mensaje solo plantillas aprobadas por Meta; enviar mal baja la calidad del número.
- Email: enlace de baja visible y control de reputación del dominio.

**Medir:** control del 20 % a la hora habitual; respuesta o clic, bajas y quejas (no deben subir), conversiones, calidad del número y rebotes.

**Límites:** sin consentimiento no se envía; no aumentar la frecuencia por tener mejor horario; la elección de la persona sobre cuándo y por dónde recibir gana al modelo.

**Orden:** (1) registro de envíos y eventos con zona horaria; (2) cola con límites; (3) histograma y respaldo por segmento; (4) planificador con exploración y control; (5) tablero y, con volumen, modelo entrenado.

---

## 9. Diseño 6: app móvil adaptativa

**Objetivo:** que la app se sienta rápida y se comporte según la situación, con señales que el sistema operativo ya permite leer sin permisos especiales.

```
App: lector de contexto ──► encabezado "contexto" ──► API ──► adaptador de respuesta
App: predictor de próxima pantalla ──► precarga           Registro de uso anónimo
```

**Qué se adapta:**

| Señal | Qué cambia |
|---|---|
| Batería baja o modo ahorro | Menos animaciones, sin precarga pesada |
| Conexión lenta o datos móviles | Imágenes y lotes más livianos |
| Modo silencio | Subtítulos activados, notificaciones sin sonido |
| Ahorro de datos del sistema | Se respeta siempre, por encima de cualquier regla |
| Hora local | Tema oscuro de noche, sin avisos fuera de horario |

**Las tres piezas:**

1. **Adaptador de respuesta en el servidor:** reglas simples (calidad de imagen, tamaño de página, campos pesados). Sin modelo.
2. **Predictor de próxima pantalla, en el dispositivo:** cuenta a qué pantalla va la persona después de cada una y por franja horaria; si una transición supera ~50 % y hay wifi y batería, precarga. Los datos quedan en el teléfono; al servidor solo llega cuántas precargas se aprovecharon.
3. **Refresco antes de la hora habitual:** con el mecanismo del sistema operativo, sin mantener la app despierta a la fuerza.

**Medir:** mitad con adaptación, mitad sin ella: tiempo hasta primera pantalla útil, aciertos de precarga contra datos descargados de más, consumo de datos y batería, retención.

**Límites:** solo las señales que cambian algo concreto; respetar siempre el ahorro del sistema; no atar el contexto a la identidad; informar en la política de privacidad; no usar el contexto para retener.

**Orden:** (1) encabezado de contexto y registro de tiempos de carga; (2) adaptador por reglas (conexión y ahorro de datos primero); (3) mute: subtítulos y notificaciones silenciosas; (4) predictor y precarga con control de desperdicio; (5) refresco en segundo plano.

---

## 10. Diseño 7: tablero interno que precalcula lo que cada persona abre

**Objetivo:** que los reportes que el equipo usa todos los días estén listos cuando los necesita, y que el inicio muestre primero lo que cada persona suele mirar a esa hora.

**Qué es un "SaaS interno":** una aplicación web que usa el propio equipo, no los clientes: panel de administración, tablero de ventas, sistema de pedidos o de stock.

**Ejemplo práctico:** ventas abre "ventas de ayer por vendedor" todos los días a las 9; administración abre "facturas pendientes" los lunes. A las 8:50 esos reportes ya están calculados y abren al instante.

```
Registro de uso ──► Detector de hábitos ──► Lista de precálculo
                                                  ▼
                       Planificador (fuera de picos) ──► Caché
Usuario abre ──► ¿caché vigente? sí: instantáneo / no: calcula y refresca
```

**Piezas:**

1. **Registro de uso:** persona o rol, reporte, filtros, hora, día, tiempo de carga.
2. **Detector de hábitos:** cada noche mira 4 a 6 semanas; si alguien abrió un reporte en 3 de las últimas 5 semanas en la misma franja, entra en la lista, con sus filtros más usados. Reglas simples.
3. **Planificador:** calcula 10 a 30 minutos antes de la hora habitual, fuera de picos; prioriza reportes lentos y muy usados; calcula una sola vez si dos personas piden lo mismo.
4. **Caché con vigencia:** si cambian los datos de origen, se recalcula; siempre se muestra "actualizado a las HH:MM".
5. **Inicio personalizado:** primero lo habitual de esa persona; lo que fija a mano gana sobre lo detectado.

**Medir:** tiempo de carga antes y después, porcentaje servido desde caché, desperdicio (cálculos que nadie abrió), carga en horas pico, opinión del equipo a las 2 semanas.

**Límites:** los permisos mandan (cada persona solo recibe precálculo de lo que ya puede ver); el registro es para acelerar el sistema, no para evaluar el rendimiento de las personas.

**Orden:** (1) registro de uso y tiempos de carga; (2) informe de reportes más lentos y más usados (ya sirve solo); (3) caché con vigencia; (4) detector de hábitos y planificador; (5) inicio personalizado.

---

## 11. Diseño 8: videos propios optimizados para que se vean completos

**Objetivo:** que cada video retenga más (menos salto en los primeros segundos, más finalización), con guion y medición repetibles.

```
Idea ──► Guion con estructura fija ──► Producción ──► Publicación (hora de la idea 5)
Métricas por video ──► Panel de retención ──► qué funciona ──► próximas ideas
```

**Estructura del guion (35 a 90 segundos):**

| Tramo | Duración | Qué hace |
|---|---|---|
| Gancho | 0 a 3 s | Afirmación concreta o pregunta, sin saludo |
| Promesa | 3 a 8 s | Qué va a saber al final y por qué le importa |
| Desarrollo | el resto | Puntos cortos con prueba visible; cambio de plano cada 2 a 4 s |
| Cierre | últimos 5 s | Un solo pedido: seguir, comentar o guardar |

Ejemplo de gancho (del video analizado): "Hice ingeniería inversa de TikTok para entender por qué es tan adictivo, y encontré más de 25 modelos de IA en tu teléfono".

**Formato visual:** subtítulos grandes palabra por palabra con la clave resaltada; alternar cámara y pantalla; pruebas visibles (código, capturas, datos); vertical 9:16.

**Producción asistida:**

- Subtítulos: la transcripción con Whisper y sherpa-onnx usada en esta sesión (modelo bajado desde GitHub, porque Hugging Face está bloqueado en este entorno); se puede correr en Contabo.
- Armado: HyperFrames o Canva, ya conectados.
- Publicación: Postiz, con la hora elegida por el diseño 5.

**Medición por video, a las 24 h y a los 7 días:** porcentaje que pasa los primeros 3 segundos, curva de retención, finalización, guardados, compartidos, seguidores ganados.

**Aprendizaje semanal:** tabla con una fila por video (gancho, duración, formato, tema, resultados) para decidir qué grabar después. Un punto de caída repetido suele ser un tramo para cortar o reescribir.

**Medir si sirve:** 4 a 6 semanas con la nueva estructura contra las 4 a 6 anteriores, con temas parecidos; lo decisivo son las visitas o clics hacia los proyectos.

**Límites:** el gancho tiene que cumplirse; no presentar como hecho lo no verificado; respetar derechos de imagen y música; anonimizar datos de personas o clientes.

**Orden:** (1) definir redes y recoger métricas de los últimos videos (línea de base); (2) plantilla de guion y lista de ganchos; (3) plantilla de subtítulos y video; (4) tabla y revisión semanal; (5) automatizar transcripción, subtítulos y publicación.

---

## 12. Límites que valen para todo

- Consentimiento y transparencia: decir en la política de privacidad qué se registra y para qué.
- Mínimo necesario: solo las señales que cambian algo concreto.
- La persona manda: cualquier preferencia explícita gana sobre el modelo.
- Nunca usar estas técnicas para retener a costa del bienestar de la persona.
- Siempre con grupo de control, para no engañarse con mejoras que no existen.

## 13. Qué revisar cuando haya acceso a Contabo

- [ ] Qué corre en el servidor: servicios, contenedores, bases de datos y recursos libres (CPU, memoria, disco).
- [ ] Cómo se despliega hoy y cómo se accede (SSH, panel, CI).
- [ ] Qué productos existen realmente (feed, tienda, app de aprendizaje, app móvil, tablero interno) y con qué están hechos. Define cuáles de las ideas 1, 2, 3 y 6 aplican.
- [ ] Por qué canal conversa hoy el agente y con qué está construido (idea 4).
- [ ] Dónde se guardan conversaciones, eventos y uso, y cuánta historia hay para calibrar.
- [ ] Circuito de derivación a una persona y quién atiende.
- [ ] Con qué se envía WhatsApp (API oficial de Meta o proveedor), plantillas aprobadas, con qué se envía email y dónde queda su historial (idea 5).
- [ ] Contactos con consentimiento registrado y dónde se guarda; zona horaria de cada uno.
- [ ] Reportes más lentos y más usados; si ya hay caché; cómo se manejan los permisos (idea 7).
- [ ] Redes en las que se publica y qué métricas se pueden exportar (idea 8).
- [ ] Si Contabo tiene capacidad para el registro, las colas, la transcripción y el precálculo.
