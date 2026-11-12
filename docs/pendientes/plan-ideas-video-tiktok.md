# Plan: 9 ideas de producto inspiradas en videos sobre el diseño de apps populares

**Estado:** diseños hechos, implementación en espera.
**Bloqueado por:** acceso completo al servidor Contabo ("contab"), donde corren los sistemas. La idea 8 se puede empezar antes.
**Fecha:** 2026-11-12

## 1. Origen

Dos videos de divulgación del mismo creador. El primero (1:29, vertical) afirma haber hecho ingeniería inversa de la app de TikTok (ideas 1 a 8). El segundo (1:15) hace lo mismo con Candy Crush (idea 9, sección 16).

### Video 1: TikTok

Transcripción completa obtenida en la sesión. Transcripción completa obtenida en la sesión. **Nada de esto está verificado**; se usa como inspiración, no como documentación oficial.

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
| Resultado | Lo que pasó después (resuelto, abandonó, convirtió), como un evento nuevo enlazado al original |

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
| 7.º | 9. Ayuda y dificultad adaptativa | Medio-alto | Necesita la base común andando para calcular la tabla de referencia. Con niños, ver sección 17 |

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

---

## 14. Con qué plugins del repo se construiría cada punto

Mapeo hecho a partir de la descripción de cada plugin (`plugins/*/.claude-plugin/plugin.json`); no se probaron todavía. Ningún plugin hace la lógica de recomendación por sí mismo: aportan los especialistas que programan, prueban y despliegan cada pieza.

**Base común (registro de eventos, idea 0):** `database-design`, `database-migrations`, `data-engineering`, `backend-development`, `data-validation-suite`, `security-compliance`.

| Idea | Plugins principales | Apoyo |
|---|---|---|
| 1. Feed | `backend-development`, `data-engineering`, `machine-learning-ops` (modelo entrenado, más adelante) | `python-development`, `database-design`, `business-analytics` |
| 2. Tienda | `backend-development`, `database-design`, `customer-sales-automation` (recordatorio de carrito) | `payment-processing` (si se toca el checkout), `business-analytics`, `machine-learning-ops` |
| 3. Aprendizaje | `frontend-mobile-development`, `ui-design`, `backend-development` | `llm-application-dev` (explicaciones en otro formato), `data-engineering`, `business-analytics` |
| 4. Agente de atención | `llm-application-dev`, `agent-orchestration`, `customer-sales-automation` | `backend-development`, `python-development`, `observability-monitoring`, `hr-legal-compliance` (texto de privacidad) |
| 5. Envío en hora óptima | `customer-sales-automation`, `backend-development` (flujos con Temporal para el planificador) | `database-design`, `data-engineering`, `security-compliance` (consentimiento) |
| 6. App móvil | `frontend-mobile-development`, `multi-platform-apps`, `application-performance` | `api-scaffolding`, `frontend-mobile-security`, `observability-monitoring` |
| 7. Tablero interno | `database-cloud-optimization`, `application-performance`, `data-engineering` | `backend-development`, `observability-monitoring`, `business-analytics` |
| 8. Videos propios | `content-marketing` | `business-analytics`; la producción y publicación usan herramientas externas al repo (HyperFrames, Canva, Postiz) |
| 9. Ayuda y dificultad adaptativa | `backend-development`, `data-engineering`, `frontend-mobile-development`, `ui-design` | `business-analytics`, `machine-learning-ops` (más adelante), `llm-application-dev` (pistas generadas) |

**Transversales para todo, sobre todo para llevarlo a Contabo:** `deployment-strategies`, `deployment-validation`, `cicd-automation`, `shell-scripting`, `observability-monitoring`, `security-scanning`, `unit-testing`, `tdd-workflows`, `documentation-generation`, `incident-response`. Para organizar el trabajo: `conductor` (seguimiento por fases) y `agent-teams` (varios agentes en paralelo).

**Salvedad:** `cloud-infrastructure` y `kubernetes-operations` apuntan a AWS, Azure, GCP y Kubernetes. Contabo es un servidor VPS, así que solo sirven si se usa Terraform o Kubernetes allí; hay que confirmarlo al ver el servidor.

---

## 15. Plan paso a paso: base común (registro de eventos)

**Qué es:** un único lugar donde cada sistema anota qué pasó (mostrado, abierto, saltado, terminado, respondido, comprado, error) y cómo terminó. Alimenta las 9 ideas. No cambia ningún comportamiento del producto.

**Revisión técnica:** esta versión incorpora la revisión hecha con el agente `backend-architect` del repo (corrido como agente genérico con sus instrucciones, porque el plugin no está instalado). Cambios principales respecto de la primera versión: la clave de idempotencia incluye la columna de partición, los resultados son eventos nuevos en vez de una actualización, el seudónimo de la persona es un UUID aleatorio con tabla de mapeo, la entrada no usa broker al inicio, y el consentimiento es un historial. El esquema SQL se verificó solo en su sintaxis (análisis con `pglast`); no se ejecutó contra una base real.

**Supuestos a confirmar al ver Contabo:** hay una base PostgreSQL 14 o superior en el servidor (si no, es el punto de partida razonable); los sistemas pueden enviar eventos por HTTP.

### Reglas de diseño (valen para todas las fases)

1. **Solo se agrega, nunca se actualiza.** Un resultado posterior ("resuelto", "convirtió") es un evento nuevo que apunta al original con `correlation_id`.
2. **El emisor fija `event_id` (UUIDv7) y `occurred_at` una sola vez** y los reenvía idénticos en cada reintento. Así la clave `(event_id, occurred_at)` evita duplicados.
3. **El `actor_id` es un UUID aleatorio**, no un hash. El hash con clave solo sirve para ubicar a la persona en una tabla de mapeo.
4. **El producto nunca espera al registro.** Si el registro falla, el sistema sigue funcionando.
5. **`context` tiene claves permitidas por tipo de evento**, con tope de tamaño, sin IP ni user-agent ni texto de mensajes.
6. **Los informes analíticos usan `occurred_at`; los operativos (retraso, rechazos) usan `received_at`.**

### Fase 0. Reconocimiento (solo lectura, sin cambiar nada)

Plugins: `c4-architecture`, `cloud-infrastructure` (solo para leer), `observability-monitoring`.

1. Inventariar qué corre en Contabo: servicios, contenedores, bases de datos, puertos, versiones.
2. Medir recursos libres: CPU, memoria, tipo de disco, espacio libre, límite de conexiones.
3. Anotar si la base es compartida con otros sistemas y qué extensiones tiene (`pg_partman`, `pg_cron`, `pgcrypto`).
4. Listar los sistemas que van a emitir eventos (agente, tienda, feed, app móvil, tablero, envíos), su lenguaje y qué identificador real tiene cada persona en cada uno (email, teléfono, id interno).
5. Estimar el volumen de eventos por día por sistema.
6. Ver cómo se hacen hoy los backups: herramienta, frecuencia, retención, dónde se guardan.
7. Anotar cómo se despliega hoy, quién tiene acceso y dónde podría guardarse la clave del hash fuera de la base.
8. Revisar qué datos personales ya se guardan, con qué consentimiento y si ya existe un almacén de consentimientos o un procedimiento de borrado.

**Listo cuando:** hay un mapa de una página del servidor y la lista de sistemas emisores con su volumen estimado.

### Fase 1. Modelo de datos y decisión de seudonimización

Plugins: `database-design`, `database-migrations`, `data-validation-suite`.

1. Decidir la seudonimización aquí, porque las fases 3 y 4 dependen de ella (ver reglas 3 y el esquema).
2. Crear el esquema de abajo. Los nombres de esquema y de rol son propuestas.
3. Si la Fase 0 muestra un volumen muy bajo, evaluar una tabla simple en lugar de particionada, con índice por `(actor_id, occurred_at)` y borrado por lotes. Es una decisión de datos, no de gusto.
4. Escribir las migraciones, con vuelta atrás, y probarlas en una copia antes de tocar la base real.
5. Crear las particiones por adelantado con una tarea programada (`pg_partman` si está, o una función propia).

**Esquema propuesto (PostgreSQL 14 o superior, sin extensiones):**

```sql
-- Esquemas: la identidad real queda aparte, con permisos restringidos.
CREATE SCHEMA IF NOT EXISTS identity;
CREATE SCHEMA IF NOT EXISTS tracking;

-- 1) Persona -> seudónimo aleatorio. Borrar la fila = desvincular.
CREATE TABLE identity.actor (
  actor_id     uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  ref_hmac     bytea NOT NULL,        -- HMAC(clave_vN, id_real); la clave vive fuera de la base
  key_version  smallint NOT NULL,
  timezone     text,                  -- zona horaria del contacto (IANA)
  created_at   timestamptz NOT NULL DEFAULT now(),
  UNIQUE (key_version, ref_hmac)
);

-- 2) Registro de borrados, solo con el id aleatorio. Se re-aplica tras restaurar un backup.
CREATE TABLE identity.erasure_log (
  actor_id      uuid PRIMARY KEY,
  requested_at  timestamptz NOT NULL,
  completed_at  timestamptz
);

-- 3) Consentimiento: historial append-only (alta y baja).
CREATE TABLE tracking.consent_event (
  consent_id     bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  actor_id       uuid NOT NULL,
  purpose        text NOT NULL,             -- p. ej. 'analytics', 'whatsapp_marketing'
  channel        text NOT NULL CHECK (channel IN ('web','app','whatsapp','email')),
  granted        boolean NOT NULL,
  policy_version text NOT NULL,
  source         text NOT NULL,             -- formulario, soporte, importación
  occurred_at    timestamptz NOT NULL,
  recorded_at    timestamptz NOT NULL DEFAULT now()
);
CREATE INDEX consent_event_lookup
  ON tracking.consent_event (actor_id, purpose, channel, occurred_at DESC);

-- Estado vigente = último registro por (actor, purpose, channel).
CREATE VIEW tracking.consent_current AS
SELECT DISTINCT ON (actor_id, purpose, channel)
       actor_id, purpose, channel, granted, policy_version, occurred_at
FROM tracking.consent_event
ORDER BY actor_id, purpose, channel, occurred_at DESC;

-- 4) Registro de eventos: solo se agrega, nunca se actualiza.
CREATE TABLE tracking.event (
  event_id        uuid        NOT NULL,   -- lo genera el emisor (UUIDv7) y lo reenvía igual
  occurred_at     timestamptz NOT NULL,   -- lo fija el emisor al crear el evento
  received_at     timestamptz NOT NULL DEFAULT now(),
  source_system   text        NOT NULL,
  schema_version  smallint    NOT NULL,
  actor_id        uuid        NOT NULL,   -- sin FK: el borrado no debe depender del orden
  actor_type      text        NOT NULL CHECK (actor_type IN ('customer','employee','contact')),
  event_type      text        NOT NULL,   -- validado contra lista en la entrada
  object_type     text,
  object_id       text,
  channel         text        NOT NULL CHECK (channel IN ('web','app','whatsapp','email')),
  correlation_id  uuid,                   -- evento de origen (para resultados)
  context         jsonb       NOT NULL DEFAULT '{}'
                  CHECK (jsonb_typeof(context) = 'object' AND pg_column_size(context) <= 2048),
  PRIMARY KEY (event_id, occurred_at)     -- la clave de partición debe estar en la PK
) PARTITION BY RANGE (occurred_at);

-- Red de seguridad: alertar si tiene filas.
CREATE TABLE tracking.event_default PARTITION OF tracking.event DEFAULT;
-- Ejemplo de partición mensual (crearlas por adelantado con una tarea programada):
CREATE TABLE tracking.event_2026_11 PARTITION OF tracking.event
  FOR VALUES FROM ('2026-11-01') TO ('2026-12-01');

CREATE INDEX event_actor_time  ON tracking.event (actor_id, occurred_at);
CREATE INDEX event_type_time   ON tracking.event (event_type, occurred_at);
CREATE INDEX event_correlation ON tracking.event (correlation_id) WHERE correlation_id IS NOT NULL;

-- 5) Rechazos: solo conteo y motivo, nunca el contenido.
CREATE TABLE tracking.ingest_reject (
  rejected_at   timestamptz NOT NULL DEFAULT now(),
  source_system text NOT NULL,
  reason        text NOT NULL,
  batch_size    int
);

-- 6) Experimentos y asignaciones.
CREATE TABLE tracking.experiment (
  experiment_key text PRIMARY KEY,
  salt           text NOT NULL,
  control_bp     smallint NOT NULL CHECK (control_bp BETWEEN 0 AND 10000), -- 1000 = 10 %
  status         text NOT NULL DEFAULT 'draft' CHECK (status IN ('draft','running','stopped')),
  started_at     timestamptz
);

CREATE TABLE tracking.experiment_assignment (
  experiment_key text NOT NULL REFERENCES tracking.experiment,
  actor_id       uuid NOT NULL,
  bucket         smallint NOT NULL CHECK (bucket BETWEEN 0 AND 9999), -- hash(salt||actor_id) mod 10000, calculado en la app
  arm            text NOT NULL CHECK (arm IN ('control','treatment')),
  assigned_at    timestamptz NOT NULL DEFAULT now(),                  -- primera exposición
  PRIMARY KEY (experiment_key, actor_id)
);
```

Inserción idempotente desde la entrada:

```sql
INSERT INTO tracking.event (...) VALUES (...)
ON CONFLICT (event_id, occurred_at) DO NOTHING;
```

Borrado a pedido, en este orden: 1) anotar en `identity.erasure_log`; 2) borrar de `tracking.event`, `tracking.experiment_assignment` y `tracking.consent_event` por `actor_id` (o conservar solo la prueba mínima que exija la ley); 3) borrar la fila de `identity.actor`.

Permisos sugeridos: el servicio de entrada inserta en `tracking.*` y lee y escribe `identity.actor`; el rol de informes solo lee `tracking.*`, sin acceso a `identity`.

**Listo cuando:** las migraciones corren y se deshacen limpias en una copia, y una inserción repetida del mismo evento no duplica nada.

### Fase 2. Entrada de eventos

Plugins: `backend-development`, `api-scaffolding`, `python-development` o el lenguaje que use el sistema.

1. Un único punto de entrada que recibe eventos de a lotes, con tope (por ejemplo 500 eventos o 1 MB por lote).
2. Cada evento se valida contra el esquema, la lista de tipos y las claves permitidas de `context`.
3. **Sin broker al inicio:** el endpoint hace `INSERT ... ON CONFLICT DO NOTHING` en una transacción corta y confirma recién después del commit. La durabilidad es la propia tabla. Un broker o una tabla de preparación se evalúan solo si el volumen lo pide.
4. Respuesta por evento: `accepted`, `duplicate` o `rejected(motivo)`. Los rechazos solo guardan conteo y motivo, no el contenido.
5. Se rechazan eventos con `occurred_at` en el futuro o con más de 7 días de atraso. Los que llegan atrasados dentro de ese margen se aceptan; la partición `DEFAULT` queda como red de seguridad y dispara alerta si recibe filas.
6. Antes de guardar, se consulta el consentimiento vigente (con caché en memoria) y se descartan los eventos de una persona con baja anterior a su `occurred_at`.
7. Librería para los emisores: asíncrona, con buffer acotado, tiempo de espera corto, reintentos con espera creciente y aleatoriedad, y descarte documentado si el buffer se llena. Así el producto no espera.
8. Autenticación por sistema emisor, TLS si no es por localhost, y límite de velocidad.

**Listo cuando:** un sistema de prueba envía mil eventos y llegan exactos, sin duplicados, aun reiniciando el servicio a mitad.

### Fase 3. Privacidad y consentimiento

Plugins: `security-compliance`, `hr-legal-compliance`, `backend-api-security`.

1. La persona se ubica con `HMAC(clave_vN, id_real)` en `identity.actor`; los eventos usan solo el `actor_id` aleatorio. La clave vive fuera de la base y tiene versión: rotarla es agregar una versión y re-hashear de forma perezosa, sin tocar los eventos.
2. No se guarda el texto de los mensajes; las conversaciones siguen en su propio almacén y el registro solo guarda señales derivadas (por ejemplo, "pregunta repetida: sí").
3. El consentimiento es un historial con finalidad, canal, versión de la política y fuente, no un solo estado. Una baja dispara el borrado de arriba.
4. Retención: eventos crudos por un período acotado (propuesta: 13 meses) y agregados sin identificar por más tiempo.
5. Copias de seguridad: no se pueden editar. Definir una retención corta y documentada (propuesta: 30 días), declararla en la política de privacidad y **re-aplicar `identity.erasure_log` después de cualquier restauración**, antes de volver a exponer la base.
6. Procedimiento para exportar los datos de una persona a pedido.
7. Texto de política de privacidad actualizado.
8. Si esto alcanza legalmente (jurisdicción, ley local) lo decide un abogado, no el equipo técnico.

**Listo cuando:** una persona de prueba no aparece en ninguna tabla viva después del borrado, y el registro de borrados se re-aplica tras una restauración de prueba. No se afirma "no queda rastro" mientras existan copias de seguridad con retención.

### Fase 4. Grupos de control

Plugins: `backend-development`, `business-analytics`.

1. Asignación estable por balde en puntos básicos: `bucket = hash(sal_del_experimento || actor_id) mod 10000`. El grupo de control es `bucket < control_bp` (1.000 = 10 %).
2. Durante un experimento en curso, los rangos solo pueden crecer, nunca achicarse, para que nadie cambie de grupo.
3. La asignación se registra en la **primera exposición** (no a todos), con `bucket` y `arm`.
4. Los porcentajes se configuran en la tabla `experiment` sin tocar código.
5. El informe incluye una prueba de desbalance: el control real debe coincidir con el configurado; si no, hay un error de asignación.

**Listo cuando:** una persona de prueba cae siempre en el mismo grupo y el control real de una muestra grande coincide con el configurado.

### Fase 5. Primer informe

Plugins: `business-analytics`, `data-engineering`, `database-cloud-optimization`.

Consultas y un panel mínimo, sobre agregados diarios y no sobre la tabla cruda:

- Eventos por día y por sistema.
- Eventos rechazados y por qué.
- Retraso entre `occurred_at` y `received_at`.
- Filas en la partición `DEFAULT` (debería ser cero).
- Tiempo de carga de reportes, por reporte (alimenta la idea 7).
- Mensajes sin resolver, por canal (alimenta la idea 4).
- Desbalance de los grupos de control.

Los agregados se recalculan sobre una ventana de 7 días, por los eventos atrasados. El rol de informes tiene `statement_timeout`.

**Listo cuando:** el panel muestra datos reales de al menos un sistema durante una semana.

### Fase 6. Despliegue y operación

Plugins: `deployment-strategies`, `deployment-validation`, `cicd-automation`, `shell-scripting`, `observability-monitoring`, `incident-response`, `security-scanning`.

1. Despliegue con revisión previa y vuelta atrás.
2. Pool de conexiones acotado para el servicio de entrada y un rol de base con límite de conexiones.
3. Copias de seguridad probadas con una restauración real, que incluya re-aplicar el registro de borrados.
4. Alertas: partición del mes siguiente ausente, filas en `DEFAULT`, rechazos que suben, disco al 80 %, retraso de guardado alto.
5. Tarea nocturna de retención: elimina particiones vencidas con `DROP`/`DETACH` y resume lo demás.
6. Medir el consumo de recursos para ver si el registro cabe junto a lo que ya corre; si la base está compartida y compite, evaluar una instancia separada.
7. Documentar con `documentation-generation`: cómo emitir eventos, el esquema y cómo operarlo.

**Listo cuando:** una restauración desde la copia funciona (con el registro de borrados re-aplicado) y las alertas disparan en una prueba controlada.

### Pruebas en todas las fases

Plugins: `unit-testing`, `tdd-workflows`. Pruebas del esquema, de la validación, de la idempotencia (mismo evento reenviado con el mismo `occurred_at`), de los eventos atrasados y de la carga. La librería de emisión se prueba con el servicio caído.

### Organización del trabajo

Plugins: `conductor` (seguir cada fase con su criterio de "listo") y `agent-teams`. Una vez fijado el esquema y la decisión de seudonimización (fase 1), las fases 3 y 4 pueden avanzar en paralelo con la 2.

### Riesgos principales

| Riesgo | Qué hacer |
|---|---|
| Idempotencia que no detecta duplicados por el particionado | Clave `(event_id, occurred_at)`; el emisor reenvía ambos idénticos |
| El registro frena el producto | Guardado corto y asíncrono en el emisor; el producto nunca espera |
| Reidentificación o clave perdida o rotada | `actor_id` aleatorio, tabla de mapeo con versión de clave, clave fuera de la base |
| Una persona sigue en las copias tras un borrado | Retención corta de copias, declarada, y re-aplicar el registro de borrados al restaurar |
| Eventos atrasados que rompen informes | Margen de 7 días, partición `DEFAULT` con alerta y ventana de recálculo |
| Se junta más de lo permitido | Solo el campo que hace falta; claves permitidas en `context`; sin texto de mensajes; retención corta |
| El servidor se queda sin disco, memoria o conexiones | Medir en Fase 0 y 5; particionar solo si vale la pena; límites de conexión; archivar lo viejo |
| El esquema queda corto | `schema_version` y migraciones con vuelta atrás |

### Decisiones que no se pueden tomar sin ver Contabo

- Motor y versión de la base, y si es compartida con otros sistemas (define si el registro vive ahí o en una instancia separada).
- Extensiones disponibles y recursos reales (define si el particionado mensual vale la pena).
- Volumen esperado de eventos por día.
- Cómo se hacen los backups hoy y por cuánto tiempo se conservan.
- Dónde guardar la clave del hash y quién la administra.
- Qué identificador real tiene cada persona en cada sistema, y si hay que unificar identidades entre sistemas.
- Lenguaje y forma de despliegue de cada emisor.
- Si ya existe un almacén de consentimientos, un broker o un procedimiento de borrado que convenga reutilizar.
- Jurisdicción aplicable (decisión legal).

### Primera semana, versión mínima

1. Fase 0 completa.
2. Esquema (en su versión simple o particionada, según el volumen) y migración.
3. Entrada de eventos con validación e idempotencia.
4. Un solo sistema emisor, el más simple, y el informe de eventos por día y de rechazos.

Con esto ya se ve si el registro funciona antes de invertir en el resto.

---

## 16. Diseño 9: ayuda y dificultad adaptativa

**Origen:** segundo video del mismo creador, sobre Candy Crush (1:15). Mismas reservas que en la sección 1: lo que afirma no está verificado.

Lo que afirma el video:

- El juego calcula un número, la "pass proximity" (cercanía a pasar el nivel), tras cada movimiento, a partir de una tabla de unas 600.000 filas descargada al teléfono (rango inferior y superior esperado por nivel y movimiento).
- Si la persona va atrasada, una acción `MakeEasierAction` cambia las probabilidades a su favor (hasta 56 caramelos extra en el último movimiento); si va demasiado bien, `MakeHarderAction` endurece el juego.
- Al perder hay dos salidas, ver un anuncio o pagar, y ambas dependen de esa cercanía: con el anuncio, a mayor cercanía menor recompensa; con el pago, un modelo usa la cercanía, la racha de derrotas, las vidas restantes y la posición en la carrera semanal para decidir cuánto dar.
- La configuración llega asignada por grupo, con un interruptor de activación y umbrales (`is_treatment_active`).

**Qué se toma y qué no.** La idea transferible es medir el progreso hacia un objetivo en cada paso y ajustar la ayuda. No se copia el ajuste oculto de probabilidades, ni el precio o la recompensa según la frustración (ver límites).

**Objetivo:** que la persona logre lo que vino a hacer (resolver una consulta, completar una lección, terminar un formulario o una compra) con la ayuda justa, de forma visible y sin trampas.

```
Tarea con pasos ──► Puntaje de progreso por paso ──► Estado: adelantado / en rango / atrasado
                              ▲                                   │
              Tabla de referencia (de datos reales)               ▼
                                                    Ayuda visible o desafío opcional
                                                                  │
                                          Registro de eventos ◄───┘ ──► medición con control
```

### Piezas

1. **Tareas con pasos definidos.** Cada objetivo se descompone en pasos medibles (por ejemplo: datos cargados, opción elegida, confirmación).
2. **Tabla de referencia.** Por tarea y paso, el rango esperado de tiempo y errores, calculado con datos históricos reales (por ejemplo, percentiles 25 y 75). Es la versión chica y honesta de la tabla de 600.000 filas. Se calcula en el servidor y se puede enviar al dispositivo (idea 6) para consultarla sin conexión.
3. **Puntaje de progreso.** En cada paso se compara a la persona con el rango de la tabla y se clasifica: adelantada, en rango o atrasada.
4. **Ayuda visible según el estado:**

| Estado | Qué ofrece |
|---|---|
| Atrasada o trabada | Una pista, un paso más chico, un ejemplo resuelto o un camino alternativo, siempre diciendo que es una ayuda |
| En rango | No interrumpe |
| Adelantada | Ofrece un desafío mayor como opción; la persona elige |

5. **Ayuda pagada, solo si existe.** Con precio fijo, igual para todas las personas, y con la opción gratuita siempre visible.
6. **Configuración por grupo.** Umbrales e interruptor de activación guardados en la tabla `experiment` de la base común, con grupo de control.
7. **Registro.** Eventos nuevos sobre la base común: `progress.step_completed`, `help.offered`, `help.accepted`, `help.declined`, enlazados con `correlation_id` a la tarea.

### Cómo medir

Con grupo de control, contra el comportamiento actual:

- Finalización de la tarea y tiempo hasta completarla.
- Uso de ayuda y si la ayuda ofrecida fue aceptada.
- Abandono por paso.
- Satisfacción declarada.
- **No** se usa como éxito el ingreso por ayuda pagada en momentos de frustración.

### Límites

- Sin cambios ocultos en las probabilidades ni en el resultado: toda ayuda se muestra como ayuda.
- Sin precio ni recompensa que dependan de la frustración, la racha de derrotas o la cercanía a abandonar.
- Sin menor recompensa para quien más se esforzó.
- La salida gratuita y la opción de parar siempre disponibles.
- Si el público incluye menores, aplica además la sección 17.

### Orden de construcción

1. Definir los objetivos y sus pasos en cada producto.
2. Calcular la tabla de referencia con datos históricos (requiere la base común andando).
3. Puntaje de progreso y estados, solo registrando, sin cambiar nada.
4. Ayudas visibles para el estado "atrasada".
5. Desafío opcional para el estado "adelantada".
6. Experimento con control; ayuda pagada solo si el negocio la necesita.

### Plugins

Principales: `backend-development`, `data-engineering`, `frontend-mobile-development`, `ui-design`. Apoyo: `business-analytics`, `machine-learning-ops` (más adelante) y `llm-application-dev` si las pistas se generan con un modelo.

### Qué revisar cuando haya acceso

- [ ] Qué tareas con pasos existen hoy en cada producto y si hay datos de tiempos y errores por paso.
- [ ] Si hay una oferta de ayuda pagada y cómo se fija su precio hoy.
- [ ] Qué público usa cada producto (adultos, menores).

---

## 17. Caso: app de coaching ontológico para niños

**Criterio general:** con niños, la regla es **bienestar antes que métricas**. Las ideas de este plan sirven en parte, y varias no deben usarse. Este apartado es una orientación técnica, no asesoramiento legal ni clínico: ambos requieren revisión de profesionales.

### Qué se puede tomar

| Idea | Cómo aplica |
|---|---|
| 3 y 9: ayuda adaptativa | Ajustar el desafío de cada actividad al nivel del niño, de forma visible y con pistas amables. Es la que mejor encaja |
| Registro de eventos (base común) | Solo lo mínimo; sin identificar al niño más de lo necesario; retención más corta que para adultos |
| 4: agente | Reconvertido: no para retener, sino para **detectar señales de malestar y derivar a un adulto o profesional** |
| 7: tablero | Para el lado adulto (coach, familia, docente): ver el progreso y las señales, no para el niño |
| 6: app móvil | Mute, conexión y ahorro de datos sí; todo lo demás, no |

### Qué no se debe usar con niños

- **Retención y "enganche":** rachas, recompensas que presionan, notificaciones para que vuelva, lote de rescate para que no se vaya.
- **Monetización dirigida al niño:** ninguna oferta de pago al niño; las compras las decide un adulto.
- **Idea 5 (envío por email y WhatsApp):** solo a madres, padres o tutores, nunca al niño.
- **Idea 8 (videos):** dirigidos a adultos y profesionales, no a niños.
- **Sensores y contexto del dispositivo** (movimiento, somnolencia): innecesarios y de alto costo en privacidad.
- **Perfilado:** nada de armar un perfil emocional del niño con fines comerciales.

### Datos y consentimiento (a validar con un abogado)

- Consentimiento verificable de madre, padre o tutor, no solo un botón de aceptar.
- Leyes de protección de datos y de menores del país (en EE.UU. COPPA; en la UE, el RGPD con edad mínima de consentimiento digital; en Argentina, la ley de protección de datos personales y la normativa sobre niñez). Confirmar cuáles aplican.
- Recoger lo mínimo, no usar los datos para publicidad, y permitir que el adulto vea, exporte y borre todo.
- La retención y el borrado de la base común (sección 15) se endurecen: períodos más cortos, borrado a pedido que alcance a las copias, y sin texto libre del niño en el registro.

### Seguridad del contenido

- Contenido preaprobado por profesionales; sin respuestas libres de un modelo sin revisión y filtros antes de cualquier texto que vea el niño.
- Sin chat abierto con desconocidos ni con otros niños.
- **Protocolo ante señales de riesgo** (tristeza intensa, daño, abuso): la app no resuelve; avisa a un adulto responsable y orienta a un profesional o a líneas de ayuda locales.
- La app no diagnostica ni reemplaza una terapia, y lo dice claramente.

### Alcance del coaching ontológico

El coaching ontológico es una práctica de acompañamiento centrada en el lenguaje, las emociones y el cuerpo. No es psicoterapia y no conviene presentarlo como tal. Para una versión infantil, el contenido y las actividades deberían revisarlos un profesional con formación en infancia (psicología o psicopedagogía), y hay que definir qué casos la app no cubre.

### Diseño de producto sugerido

- **Dos perfiles:** el del niño (actividades) y el del adulto (panel, ajustes, consentimiento, compras).
- Sesiones cortas con pausas, límites de tiempo definidos por el adulto, sin rachas.
- Éxito medido por bienestar y aprendizaje (por ejemplo, autoevaluación guiada, observación del adulto), no por tiempo de uso.

### Qué revisar antes de empezar

- [ ] Rango de edad del público.
- [ ] Quién usa, quién decide y quién paga (familia, colegio, coach).
- [ ] Qué hace la app hoy y cómo guarda los datos.
- [ ] Qué profesionales validan el contenido y el protocolo de riesgo.
- [ ] Qué leyes aplican y quién las revisa.
