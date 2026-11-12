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

**Transversales para todo, sobre todo para llevarlo a Contabo:** `deployment-strategies`, `deployment-validation`, `cicd-automation`, `shell-scripting`, `observability-monitoring`, `security-scanning`, `unit-testing`, `tdd-workflows`, `documentation-generation`, `incident-response`. Para organizar el trabajo: `conductor` (seguimiento por fases) y `agent-teams` (varios agentes en paralelo).

**Salvedad:** `cloud-infrastructure` y `kubernetes-operations` apuntan a AWS, Azure, GCP y Kubernetes. Contabo es un servidor VPS, así que solo sirven si se usa Terraform o Kubernetes allí; hay que confirmarlo al ver el servidor.

---

## 15. Plan paso a paso: base común (registro de eventos)

**Qué es:** un único lugar donde cada sistema anota qué pasó (mostrado, abierto, saltado, terminado, respondido, comprado, error) y cómo terminó. Alimenta las 8 ideas. No cambia ningún comportamiento del producto.

**Supuestos a confirmar al ver Contabo:** hay una base de datos relacional en el servidor (si no, PostgreSQL es el punto de partida razonable); los sistemas pueden enviar eventos por HTTP o escribirlos directo a la base.

### Fase 0. Reconocimiento (solo lectura, sin cambiar nada)

Plugins: `c4-architecture`, `cloud-infrastructure` (solo para leer), `observability-monitoring`.

1. Inventariar qué corre en Contabo: servicios, contenedores, bases de datos, puertos, versiones.
2. Medir recursos libres: CPU, memoria, disco, y backups existentes.
3. Listar los sistemas que van a emitir eventos (agente, tienda, feed, app móvil, tablero, envíos) y en qué lenguaje están.
4. Anotar cómo se despliega hoy y quién tiene acceso.
5. Revisar qué datos personales ya se guardan y con qué consentimiento.

**Listo cuando:** hay un mapa de una página del servidor y la lista de sistemas emisores.

### Fase 1. Modelo de datos

Plugins: `database-design`, `database-migrations`, `data-validation-suite`.

1. Definir el esquema mínimo de un evento:

| Campo | Contenido |
|---|---|
| `event_id` | Identificador único (también sirve para no duplicar) |
| `occurred_at` | Fecha y hora con zona horaria |
| `actor_id` | Identificador seudónimo de la persona (no el dato real) |
| `actor_type` | Cliente, empleado, contacto |
| `event_type` | Mostrado, abierto, saltado, terminado, respondido, comprado, error |
| `object_type` y `object_id` | Qué fue: ítem, producto, reporte, mensaje, lección |
| `channel` | Web, app, WhatsApp, email |
| `context` | Solo lo necesario: tipo de conexión, hora local, modo silencio |
| `outcome` | Resultado posterior, se completa al cerrar |
| `schema_version` | Versión del esquema, para poder evolucionarlo |

2. Tablas de apoyo: consentimientos por persona y canal, asignaciones a grupos de control, zona horaria por contacto.
3. Partir la tabla de eventos por mes, con índices por persona y por fecha, para que siga rápida con volumen.
4. Escribir las migraciones, con vuelta atrás, y probarlas en una copia antes de tocar la base real.

**Listo cuando:** las migraciones corren y se deshacen limpias en una copia.

### Fase 2. Entrada de eventos

Plugins: `backend-development`, `api-scaffolding`, `python-development` o el lenguaje que use el sistema.

1. Un único punto de entrada que recibe eventos de a lotes.
2. Cada evento se valida contra el esquema; los inválidos se rechazan y se cuentan.
3. Guardado con cola: la persona o el sistema nunca esperan al registro, y si el registro falla el producto sigue funcionando.
4. Idempotencia: reenviar el mismo evento no lo duplica.
5. Una pequeña librería para que cada sistema emita eventos con una línea de código.
6. Autenticación por sistema emisor y límite de velocidad.

**Listo cuando:** un sistema de prueba envía mil eventos y llegan exactos, sin duplicados, aun reiniciando el servicio a mitad.

### Fase 3. Privacidad y consentimiento

Plugins: `security-compliance`, `hr-legal-compliance`, `backend-api-security`.

1. La persona se identifica con un seudónimo (hash con clave secreta guardada fuera de la base).
2. No se guarda el texto de mensajes en el registro; las conversaciones siguen en su propio almacén y el registro solo guarda señales derivadas (por ejemplo, "pregunta repetida: sí").
3. Los eventos se guardan solo si hay consentimiento cuando el canal lo exige.
4. Retención definida: eventos crudos por un período acotado (propuesta: 13 meses) y agregados sin identificar, por más tiempo.
5. Procedimiento para borrar o exportar los datos de una persona a pedido.
6. Texto de política de privacidad actualizado.

**Listo cuando:** se puede borrar a una persona de prueba y no queda rastro identificable.

### Fase 4. Grupos de control

Plugins: `backend-development`, `business-analytics`.

1. Asignación estable: `hash(actor_id + nombre_del_experimento) mod 100`; la misma persona cae siempre en el mismo grupo.
2. Cada asignación se registra, con el experimento y la fecha.
3. Los porcentajes (control 10 a 20 %) se configuran por experimento sin tocar código.

**Listo cuando:** una persona de prueba recibe siempre el mismo grupo y los porcentajes se cumplen en una muestra grande.

### Fase 5. Primer informe

Plugins: `business-analytics`, `data-engineering`, `database-cloud-optimization`.

Consultas y un panel mínimo que respondan:

- Eventos por día y por sistema.
- Eventos rechazados y por qué.
- Retraso entre que ocurre un evento y se guarda.
- Tiempo de carga de reportes, por reporte (alimenta la idea 7).
- Mensajes sin resolver, por canal (alimenta la idea 4).

**Listo cuando:** el panel muestra datos reales de al menos un sistema durante una semana.

### Fase 6. Despliegue y operación

Plugins: `deployment-strategies`, `deployment-validation`, `cicd-automation`, `shell-scripting`, `observability-monitoring`, `incident-response`, `security-scanning`.

1. Despliegue con revisión previa y vuelta atrás.
2. Copias de seguridad probadas con una restauración real.
3. Alertas: cola atascada, rechazos que suben, disco al 80 %, retraso de guardado alto.
4. Tarea nocturna de retención (borra lo vencido y resume lo demás).
5. Explorar el consumo de recursos para ver si el registro cabe junto a lo que ya corre.
6. Documentar con `documentation-generation`: cómo emitir eventos, el esquema y cómo operarlo.

**Listo cuando:** una restauración desde la copia funciona y las alertas disparan en una prueba controlada.

### Pruebas en todas las fases

Plugins: `unit-testing`, `tdd-workflows`. Pruebas del esquema, de la validación, de la idempotencia y de la carga. La librería de emisión se prueba con un servicio caído.

### Organización del trabajo

Plugins: `conductor` (seguir cada fase con su criterio de "listo") y `agent-teams` (fases 3 y 4 pueden avanzar en paralelo con la 2 una vez fijado el esquema).

### Riesgos principales

| Riesgo | Qué hacer |
|---|---|
| El registro frena el producto | Cola y guardado asíncrono; el producto nunca espera |
| Se junta más de lo permitido | Solo el campo que hace falta; sin texto de mensajes; retención corta |
| El servidor se queda sin disco o memoria | Medir en Fase 0 y 5; particionar por mes; archivar lo viejo |
| El esquema queda corto | `schema_version` y migraciones con vuelta atrás |
| Eventos duplicados o perdidos | Idempotencia y conteo de rechazados |

### Primera semana, versión mínima

1. Fase 0 completa.
2. Esquema y migración de la tabla de eventos.
3. Entrada de eventos con validación.
4. Un solo sistema emisor, el más simple, y el informe de eventos por día.

Con esto ya se ve si el registro funciona antes de invertir en el resto.
