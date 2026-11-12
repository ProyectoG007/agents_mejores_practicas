# Pendiente: tablero interno que precalcula lo que cada persona abre

**Estado:** diseño hecho, implementación en espera.
**Depende de:** acceso al servidor Contabo ("contab") y a las bases de datos de los sistemas internos.
**Fecha de la nota:** 2026-11-12
**Origen:** idea 7 del video de TikTok (predecir cuándo abre la app cada persona y tenerla lista). Ver `agente-deteccion-abandono.md`.

## Objetivo

Que los reportes y pantallas que cada persona del equipo usa todos los días estén listos cuando los necesita, sin esperas, y que la pantalla de inicio muestre primero lo que esa persona suele mirar a esa hora.

Ejemplo: ventas abre "ventas de ayer por vendedor" todos los días a las 9; administración abre "facturas pendientes" los lunes. A las 8:50 esos reportes ya están calculados y abren al instante.

## Diseño

```
Registro de uso ──► Detector de hábitos ──► Lista de precálculo (reporte, hora, persona)
                                                   │
                                                   ▼
Planificador (fuera de horas de carga) ──► Calcula y guarda en caché
                                                   │
Usuario abre el reporte ──► ¿caché vigente? ── sí ──► responde al instante
                                    │ no
                                    └──► calcula en el momento y refresca la caché
```

### 1. Registro de uso

Cada vez que alguien abre un reporte se guarda: persona o rol, reporte, filtros usados, hora, día de la semana y cuánto tardó en cargar. Es solo uso interno; se informa al equipo.

### 2. Detector de hábitos

Un proceso periódico (por ejemplo, cada noche) mira las últimas 4 a 6 semanas:

- Un reporte entra en la lista de precálculo si una persona lo abrió en al menos 3 de las últimas 5 semanas en la misma franja (misma hora, mismo día de la semana o todos los días).
- Se guardan sus filtros más usados; si usa varios, se precalcula la combinación más frecuente.
- Para personas con poco historial se usa el patrón de su rol.

Reglas simples; no hace falta un modelo.

### 3. Planificador de precálculo

- Corre unos 10 a 30 minutos antes de la hora habitual, y fuera de los picos de carga.
- Si el reporte es pesado, se calcula antes y se refresca poco antes de la hora.
- Prioriza los reportes lentos y los más usados; los de carga instantánea no se precalculan.
- Si dos personas usan el mismo reporte con los mismos filtros, se calcula una sola vez.

### 4. Caché y vigencia

- Cada resultado guarda de qué datos depende y hasta cuándo es válido.
- Si los datos de origen cambiaron después del cálculo, el reporte se marca como desactualizado y se vuelve a calcular al abrirlo (o se muestra con la hora del último cálculo).
- Siempre se muestra "actualizado a las HH:MM" para que nadie decida con datos viejos sin saberlo.

### 5. Pantalla de inicio personalizada

El inicio muestra primero los reportes que esa persona abre a esa hora, y debajo los demás. La persona puede fijar o quitar los que quiera; lo manual siempre gana sobre lo detectado.

## Cómo medir si sirve

- Tiempo de carga de los reportes habituales, antes y después.
- Porcentaje de aperturas servidas desde caché.
- Aciertos del precálculo contra cálculos que nadie abrió (no debe haber mucho desperdicio).
- Carga del servidor en horarios pico.
- Opinión del equipo después de 2 semanas.

## Límites

- Los permisos mandan: cada persona solo recibe precálculos de lo que ya puede ver.
- Una caché no puede mostrar datos de otra persona ni de otra empresa.
- No usar el registro para evaluar el rendimiento de las personas; es solo para acelerar el sistema.
- Dejar visible la hora del último cálculo.

## Orden de construcción

1. Registro de uso y de tiempos de carga de los reportes.
2. Informe de los reportes más lentos y más usados (ya sirve para decidir antes de construir más).
3. Caché de resultados con vigencia y la leyenda "actualizado a las…".
4. Detector de hábitos y planificador de precálculo.
5. Pantalla de inicio personalizada.

## Qué revisar cuando haya acceso a Contabo

- [ ] Qué sistemas internos y reportes existen, y con qué están hechos.
- [ ] Qué reportes tardan más y cuántas personas los usan.
- [ ] Dónde se guarda hoy el uso y si hay registros para calibrar.
- [ ] Si ya hay caché (Redis, base de datos u otra) o hay que sumarla.
- [ ] Recursos libres de Contabo para el precálculo en horas de poca carga.
- [ ] Cómo se manejan hoy los permisos por usuario y por rol.
