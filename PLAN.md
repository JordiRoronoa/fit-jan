# FitJan — plan de la app

FitJan es una app para el móvil que guarda tu plan de entrenamiento, te guía durante la sesión con temporizadores automáticos y apunta lo que haces.

No tiene servidor. Todo el plan va escrito dentro del código y todo lo que guardas se queda en el móvil.

---

## 1. Cómo se construye (igual que CuinesJan)

CuinesJan es una sola página `index.html` publicada con GitHub Pages desde la rama `main`. FitJan copia esa forma:

| Pieza | CuinesJan | FitJan |
|---|---|---|
| Repo | `JordiRoronoa/cuines-jan` | `JordiRoronoa/fit-jan` |
| Dirección | `jordiroronoa.github.io/cuines-jan/` | `jordiroronoa.github.io/fit-jan/` |
| Código | `index.html` con HTML, CSS y JS juntos, sin compilar | Igual |
| Instalar en el móvil | `manifest.webmanifest` + carpeta `icons/` | Igual |
| Guardar datos | `localStorage` con prefijo `rq.` | `localStorage` con prefijo `fj.` |
| Navegación | Barra de pestañas abajo, pestaña en `#hash` | Igual |
| Funciona sin internet | No | **Sí**, con `sw.js` (nuevo) |

**Novedad: `sw.js`.**
Un service worker es un pequeño script que guarda la app en el móvil. Sirve para que FitJan abra aunque no tengas cobertura en casa o en el parque.
- Pide `index.html` primero a internet y, si no hay conexión, usa la copia guardada. Así las actualizaciones llegan sin quedarse atascadas.
- Los iconos y las fuentes se guardan y se usan desde la copia.

**Archivos finales:**

```
FitJan/
  index.html             la app entera
  manifest.webmanifest   nombre, colores e iconos para instalarla
  sw.js                  copia offline
  icons/                 icon.svg, icon-192/512.png, maskable-192/512.png, apple-touch-icon.png
  README.md
  PLAN.md                este documento
```

---

## 2. Pestañas

Barra inferior con 4 pestañas:

| Pestaña | Para qué |
|---|---|
| **Hoy** | Pantalla de inicio. Dice qué sesión toca y tiene el botón grande «Empezar». |
| **Plan** | Las 4 sesiones completas, con ejercicios, series, objetivo y última marca. Aquí marcas una sesión como hecha a mano. |
| **Calendario** | Mes con los días entrenados. Tocas un día para ver, añadir o quitar. |
| **Guía** | Reglas del plan (progresión, descansos, comida, anclaje seguro), ajustes y copia de seguridad. |

El entrenamiento en marcha no es una pestaña. Es una pantalla completa que se abre al pulsar «Empezar» y tapa la barra, para que no la toques sin querer.

---

## 3. Funciones

### 3.1 Plan de entrenamiento (escrito en el código)

Son las 4 sesiones de la rotación: **Torso A → Pierna A → Torso B → Pierna B**, con las superseries ya emparejadas.

Cada ejercicio muestra:
- Nombre, músculos y 2–3 frases de técnica (cómo ponerte, qué notar).
- Largo de las cintas del TRX (largo, medio o corto).
- Series y rango de repeticiones. Ejemplo: 4 × 8–15.
- **Última vez:** lo que hiciste. Ejemplo: 12 · 11 · 10 con chaleco de 10 kg.
- **Próxima vez:** el objetivo. Puedes cambiarlo a mano.
- Botón «Ver vídeo», que abre una búsqueda de YouTube con el nombre del ejercicio.

### 3.2 Marcar sesiones hechas y calendario

- **Al acabar un entrenamiento en marcha,** el día se marca solo en el calendario.
- **En Hoy y en Plan,** cada sesión tiene el botón «Marcar como hecha hoy». Si te olvidaste otro día, «Otro día…» abre el selector de fecha.
- **En el calendario,** cada día entrenado lleva una etiqueta de color con la sesión (TA, PA, TB, PB). Al tocar un día ves lo que hiciste, serie a serie, y tienes:
  - «Quitar»: borra ese entrenamiento, con confirmación.
  - «Añadir»: marca una sesión ese día.
- Debajo del mes: sesiones de esta semana y semanas seguidas entrenando.

### 3.3 Rotación «hoy toca»

La app mira la última sesión hecha y te propone la siguiente de la rotación. Puedes elegir otra si quieres. Si quitas un día del calendario, la propuesta se recalcula.

### 3.4 Entrenamiento en marcha (la parte principal)

**Idea:** pulsas «Empezar» una sola vez. A partir de ahí la app sigue sola y tú solo pulsas «Hecho» al acabar cada serie.

Cada serie pasa por dos estados:

1. **Trabajando.** El cronómetro sube. Ves el ejercicio, la serie y el objetivo. Un botón enorme abajo dice «Hecho».
2. **Descansando.** Una cuenta atrás baja. Ves qué viene después. Cuando llega a 0, suena un pitido y empieza la serie siguiente: vuelves al estado 1 sin tocar nada.

```
 [Empezar] → TRABAJANDO → [Hecho] → DESCANSANDO → (llega a 0) → TRABAJANDO → ...
                                       │
                                       ├─ [Pausar / Seguir]  alarga el descanso lo que quieras
                                       ├─ [+15 s]            alarga 15 segundos
                                       └─ [Ya]               se salta el descanso
```

**Pantalla mientras descansas** (boceto):

```
 ┌──────────────────────────────────┐
 │ Torso A          Bloque 1/4   ✕  │
 │                                  │
 │            ⟳ 0:24                │  ← cuenta atrás grande
 │                                  │
 │  Siguiente: Remo TRX · serie 2/4 │
 │  Objetivo 12 reps · chaleco 10kg │
 │ ──────────────────────────────── │
 │  Serie hecha: Flexiones 1/4      │
 │  Reps  [ − ]  12  [ + ]          │  ← ya viene relleno con el objetivo
 │  Kg    [ − ]  10  [ + ]          │
 │ ──────────────────────────────── │
 │  [ Pausar ]  [ +15 s ]  [ Ya ]   │
 │            Deshacer «Hecho»      │
 └──────────────────────────────────┘
```

**Apuntar repeticiones y kg:** cuando pulsas «Hecho», la serie se guarda con el objetivo ya puesto. Si hiciste otra cosa, la corriges con los botones − y + mientras descansas. No tienes que escribir nada si cumples el objetivo.

**Descansos por defecto** (se cambian en Guía → Ajustes):

| Paso | Descanso | Por qué |
|---|---|---|
| Entre los dos ejercicios de una superserie (A1 → A2) | 30 s | Trabajan músculos distintos, solo necesitas recolocarte. |
| Después de la vuelta de una superserie (A2 → A1) | 90 s | Cada músculo descansa unos 2–2,5 min en total. |
| Ejercicio grande en solitario (búlgara, pistol) | 120 s | Cansa todo el cuerpo. |
| Ejercicio pequeño en solitario | 60 s | Hombro, gemelos, abdomen. |
| Cambio de bloque | 90 s | Te da tiempo a ajustar el TRX o ponerte el chaleco. |

**Detalles que lo hacen fácil de usar:**
- **«Deshacer»:** vuelve atrás si pulsaste «Hecho» sin querer.
- **Menú «⋯»:** saltar serie, saltar ejercicio, ver técnica.
- **Ejercicios por tiempo** (plancha, hollow): el cronómetro guarda los segundos solo.
- **Ejercicios a una pierna:** una serie son las dos piernas seguidas. Apuntas las repeticiones por pierna.
- **Pitidos:** 3 pitidos cortos en los últimos 3 segundos y uno largo al empezar la serie.
- **Pantalla siempre encendida** durante el entrenamiento.
- **Sin pérdidas:** si cierras la app o se apaga el móvil, al volver sigues donde estabas.
- **Al terminar:** resumen con duración, series hechas y récords nuevos. El día se marca en el calendario.

### 3.5 Progresión automática

Aplica la regla del plan: «sube repeticiones hasta el máximo del rango y después hazlo más difícil».
- **Si en todas las series llegaste al máximo del rango,** la app pone la etiqueta «Sube dificultad» y sugiere cómo: chaleco, pies más cerca del anclaje, ritmo lento o una pierna.
- **Si no llegaste,** el objetivo de la próxima vez es +1 repetición en cada serie.
- **Siempre puedes cambiar el objetivo a mano** en la pestaña Plan.

### 3.6 Guía y ajustes

- **Guía:** progresión, esfuerzo (1–2 repeticiones en reserva), descansos, descarga cada 6–8 semanas, comida y cómo anclar la puerta.
- **Ajustes:** tiempos de descanso, sonido sí o no, vibración sí o no.
- **Copia de seguridad:** botón «Exportar» que descarga un archivo `.json` con todos tus datos, y botón «Importar» para recuperarlos.

---

## 4. Lo que añado yo (útil y barato)

| Añadido | Por qué te sirve |
|---|---|
| Funciona sin internet (`sw.js`) | Abres la app aunque no haya cobertura. |
| Copia de seguridad | Si cambias de móvil o se borra el navegador, no pierdes el historial. |
| «Hoy toca» con la rotación | No tienes que acordarte de qué sesión hiciste la última vez. |
| Peso corporal semanal con gráfica | Tu objetivo es ganar peso. Así ves si subes 1–2 kg al mes. |
| Aviso de semana de descarga | Cada 6–8 semanas te propone hacer la mitad de series. |
| Gráfica por ejercicio | Ves cómo suben tus repeticiones o tus kg. |
| Modo oscuro automático | Sigue el ajuste del móvil. |

Los tres últimos van en la fase 6. Si no los quieres, se quitan.

---

## 5. Datos guardados en el móvil

Todo usa `localStorage` con prefijo `fj.`. Hay una sola fuente de verdad: el historial. «Última vez», calendario y «hoy toca» salen de él, así que si borras un día todo se recalcula solo.

| Clave | Contenido |
|---|---|
| `fj.logs` | Lista de entrenamientos: `{ id, date: '2026-10-06', session: 'TA', source: 'live' \| 'manual', start, end, sets: [{ ex, n, reps, sec, kg }] }` |
| `fj.targets` | Objetivo de la próxima vez por ejercicio: `{ remo-neutro: { reps: [13, 12, 11], kg: 10 } }` |
| `fj.notes` | Nota libre por ejercicio. Ejemplo: «pies a 2 pasos del anclaje». |
| `fj.live` | Entrenamiento en marcha (para seguir si se cierra la app). |
| `fj.settings` | Descansos, sonido, vibración. |
| `fj.weight` | Peso corporal: `[{ date, kg }]` |
| `fj.v` | Versión de los datos, para poder cambiarlos sin romper nada. |

Lo que va escrito en el código (no se guarda):
- `EXERCISES`: id, nombre, músculos, técnica, largo del TRX, tipo (`reps` o `time`), si es por pierna, carga por defecto, cómo subir dificultad.
- `SESSIONS`: las 4 sesiones. Cada una es una lista de bloques. Un bloque es un ejercicio solo o una superserie de dos, con sus series y descansos.
- `ROTATION`: `['TA', 'PA', 'TB', 'PB']`.

### Las sesiones con sus superseries

| Sesión | Bloques |
|---|---|
| **Torso A** | Flexiones con chaleco 4 + Remo TRX neutro 4 · Pica pies elevados 3 + Dominada asistida TRX 3 · Face pull TRX 3 · Curl TRX 3 + Extensión tríceps TRX 3 |
| **Pierna A** | Búlgara con chaleco 4 (sola) · Curl femoral TRX 3 + Laterales 3 kg 3 · Hip thrust a una pierna 3 + Rodillas al pecho TRX 3 · Gemelo a una pierna 4 + Superman 4 |
| **Torso B** | Fondos entre sillas 4 + Remo TRX prono 4 · Flexiones pies elevados 3 + Jalón brazos rectos TRX 3 · Elevación en Y TRX 3 · Curl TRX 3 + Flexiones diamante 3 |
| **Pierna B** | Pistol asistida TRX 4 (sola) · Peso muerto rumano a una pierna 3 + Laterales 3 kg 3 · Zancada atrás con chaleco 3 + Plancha 3 · Gemelo a una pierna 4 + Hollow hold 4 |

---

## 6. Cómo funciona por dentro el entrenamiento en marcha

**Una cola plana.** Al empezar, la sesión se convierte en una lista de pasos en orden: `[{ ex, set, rest }, ...]`. Las superseries ya salen intercaladas. El entrenamiento en marcha es solo «en qué paso estoy» + «trabajando o descansando» + la hora a la que empezó o acaba.

```js
live = { session: 'TA', queue: [...], i: 3, phase: 'work' | 'rest',
         startedAt, restEndsAt, pausedLeft, done: [...] }
```

**Cuidado con los temporizadores en el móvil.** Cuando la pantalla se bloquea o cambias de app, el navegador para o frena el JavaScript. Una cuenta atrás que reste 1 cada segundo se queda atrasada.
- **Solución:** guardar la hora exacta en que acaba el descanso (`restEndsAt`) y calcular `restEndsAt - Date.now()` en cada repintado y al volver a la app (`visibilitychange`).
- `fj.live` se guarda en cada cambio, así un cierre no pierde nada.

**Avisos.**
- **Sonido:** se genera con Web Audio. El móvil solo deja sonar audio después de un toque, así que el botón «Empezar» lo activa.
- **Pantalla encendida:** usa la Wake Lock API. Si el móvil no la soporta, la app avisa de que bajes el bloqueo automático.
- **Vibración (Android):** vibración corta en los últimos 3 segundos y larga al empezar la serie, junto con los pitidos.
- **Con la pantalla bloqueada no hay aviso.** Sin servidor, una web no puede programar notificaciones. Por eso es importante la pantalla siempre encendida.

---

## 7. Fases de construcción

Cada fase termina con algo que ya funciona en el móvil.

| Fase | Qué se hace | Resultado |
|---|---|---|
| 1. Esqueleto | Carpeta, `index.html`, `manifest`, iconos, pestañas, `store`, datos del plan | Se instala en el móvil y muestra el plan |
| 2. Calendario | Check «Hecha hoy», calendario mensual, quitar y añadir días, «hoy toca» | Ya puedes apuntar entrenamientos |
| 3. Entrenamiento en marcha | Cola, estados trabajando y descansando, cuenta atrás por hora, pausa, +15 s, deshacer, pitidos, pantalla encendida, recuperar al reabrir | Entrenas con la app |
| 4. Repeticiones y kg | Apuntar en el descanso, última vez, próxima vez, progresión automática, editar objetivo | La app sabe en cuánto te quedaste |
| 5. Guía y seguridad | Guía, ajustes, exportar e importar, `sw.js` | Funciona sin internet y tienes copia |
| 6. Extras | Peso corporal, gráficas, aviso de descarga, modo oscuro | Ves tu progreso |
| 7. Publicar | Repo `fit-jan`, GitHub Pages en `main`, instalar en el móvil | App en el móvil |

**Comprobación de cada fase:** abrir la app en Chrome con tamaño de móvil y recorrer el flujo. En la fase 3, además: bloquear la pantalla durante un descanso y comprobar que al volver el tiempo es correcto.

---

## 8. Trampas a tener en cuenta

- **Móvil: Android.** La app se prueba con Chrome de Android y usa vibración además de pitidos.
- **Limpiar los datos de Chrome borra el historial.** Si borras «datos de sitios» o desinstalas la app, se pierde todo. Exporta una copia de vez en cuando.
- **La app pide almacenamiento persistente** (`navigator.storage.persist()`), para que Chrome no borre los datos si falta espacio.
- **Cambiar el plan más adelante:** si cambias un ejercicio de nombre, su id debe seguir igual. Si no, pierde su historial.

---

## 9. Cómo instalarla en el móvil

Abre `jordiroronoa.github.io/fit-jan/` en Chrome → menú ⋮ → «Instalar aplicación» (o «Añadir a pantalla de inicio»). Después ábrela siempre desde el icono.
