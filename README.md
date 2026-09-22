# OpoTest — notas de mantenimiento

Banco de preguntas de test para oposiciones, extraído de la pestaña "Tests" de
Operación Baeza para vivir como app independiente. PWA estática (HTML + JS, sin
build ni frameworks), con los datos guardados en Firebase Firestore y acceso por
usuario/contraseña (Firebase Authentication), pensada para poder crecer en el
futuro hacia una plataforma de test multiusuario.

## Datos del proyecto

- **Proyecto Firebase:** _(anota aquí el ID de tu proyecto nuevo, distinto del de Operación Baeza)_
- **Consola Firebase:** _(anota aquí el enlace, algo tipo `https://console.firebase.google.com/project/TU-PROYECTO`)_
- **Dónde vive la web:** GitHub Pages, repo: _(anota aquí la URL de tu repo)_
- **URL pública de la app:** _(anota aquí tu enlace de GitHub Pages, algo tipo
  `https://tu-usuario.github.io/opotest/`)_

## Archivos del proyecto

| Archivo               | Para qué sirve                                                |
|------------------------|----------------------------------------------------------------|
| `index.html`           | Toda la app (HTML + JS); los estilos viven aparte en `style.css` |
| `style.css`            | Todo el CSS de la app (separado de `index.html` para que sea más manejable) |
| `manifest.json`        | Metadatos de la PWA (nombre, iconos, colores)                  |
| `service-worker.js`    | Caché offline del "app shell"                                  |
| `icon-192.png` / `icon-512.png` | Iconos de la app                                       |
| `icon-maskable-192.png` / `icon-maskable-512.png` | Iconos «maskable» (Android los recorta en círculo u otras formas; sin marco) |
| `apple-touch-icon.png` | Icono para la pantalla de inicio de iPhone/iPad          |
| `firestore.rules`      | Reglas de seguridad de la base de datos (se pegan en la consola de Firebase, no en GitHub Pages) |
| `firebase.json`        | Solo necesario si algún día despliegas con Firebase Hosting en vez de (o además de) GitHub Pages |

## Puesta en marcha (una sola vez)

1. **Crea un proyecto de Firebase nuevo** en https://console.firebase.google.com
   (dale un nombre distinto al de Operación Baeza, por ejemplo "opotest").
2. **Añade una app web** dentro del proyecto (icono `</>`) y copia el objeto de
   configuración que te da (`apiKey`, `authDomain`, `projectId`, etc.).
3. Pega esos datos en `index.html`, buscando `const FIREBASE_CONFIG = {...}` y
   sustituyendo los valores `"PEGA_AQUI..."`.
4. **Activa Firestore**: en el menú lateral, Firestore Database → Crear base de
   datos (modo producción, la región que te propongan por defecto está bien).
5. **Activa Authentication**: en el menú lateral, Authentication → Sign-in method
   → habilita el proveedor **Correo electrónico/contraseña**.
6. **Publica las reglas de seguridad**: Firestore Database → pestaña Reglas →
   pega el contenido de `firestore.rules` de este proyecto → Publicar.
7. Sube estos archivos a un repositorio de GitHub nuevo (por ejemplo `opotest`) y
   activa **GitHub Pages** en Settings → Pages → rama `main`, carpeta `/ (root)`.
8. Abre la URL de GitHub Pages, pulsa **"Crear cuenta nueva"**, mete tu correo y
   una contraseña: esa es ya tu cuenta para entrar desde cualquier dispositivo.

## Autenticación (usuario/contraseña)

- La app usa Firebase Authentication con correo y contraseña. Cada persona que
  entra tiene su propio banco de preguntas, aislado del de cualquier otra
  (las reglas de Firestore lo garantizan a nivel de servidor, no solo en la
  interfaz).
- Para usarla en otro dispositivo, entra con el mismo correo y contraseña, no
  hace falta ningún enlace ni código.
- Si olvidas la contraseña, el botón "¿Olvidaste tu contraseña?" te envía un
  correo de recuperación (lo gestiona Firebase automáticamente).
- Cuando llegue el momento de vender la app a terceros, cada comprador simplemente
  crea su propia cuenta desde la pantalla de acceso — el modelo de datos
  (`users/{uid}/testQuestions/...`) ya está pensado para eso, no haría falta
  ningún cambio de estructura.

## Banco de Tests

- **Almacenamiento**: cada pregunta se guarda como un documento independiente en
  Firestore, dentro de `users/{tu uid}/testQuestions/`. Esto es importante porque
  Firestore limita cada documento a 1 MB — guardando cada pregunta por separado,
  el banco puede crecer a miles de preguntas sin problema.
- Con el mismo criterio, los documentos legales del asistente de IA (ver más abajo)
  viven en `users/{tu uid}/legalDocs/` (metadatos) + `.../legalDocs/{docId}/chunks/`
  (el texto, troceado igual que las preguntas para no chocar con ese límite de 1 MB).
- Para transcribir fotos automáticamente con IA usa Google Gemini (modelo
  `gemini-3.6-flash` por defecto). Necesitas tu propia clave API gratuita de
  https://aistudio.google.com/apikey.
- Esa clave **se guarda solo en el navegador de cada dispositivo** (no viaja a
  Firebase, no es compartida), así que hay que volver a pegarla si usas la app
  desde el móvil y el ordenador.
- Si algún día Google cambia el nombre del modelo y da error 404 "no longer
  available", cambia el nombre en el campo "Modelo" de esa pestaña (comprueba el
  nombre vigente en https://ai.google.dev/gemini-api/docs/models).
- También puedes escribir las preguntas a mano sin usar ninguna IA (botón
  "Escribir manualmente"), sin coste ni clave API.

## Añadir preguntas al banco (a mano, desde foto, o varias desde un PDF)

- **Como admin**, la pestaña **Añadir pregunta** (dentro de Tests) es donde entra toda pregunta
  nueva, por tres métodos que se eligen arriba con un selector: **A mano**, **Desde foto (IA)** y
  **Varias desde PDF (IA)**. Antes "Generar con IA" era una pestaña aparte; ahora es uno de estos
  tres métodos, para no repartir en dos sitios distintos lo que en el fondo es la misma tarea.
- Los bloques que no usas todo el rato (la clave API de Gemini, la explicación de "cómo funciona"
  la generación por PDF) están plegados por defecto en secciones desplegables — pulsa sobre el
  título para abrirlas. Se recuerdan abiertas o cerradas mientras estés en esa pestaña.
- **A mano**: el formulario de siempre (categoría, tema/subtema, enunciado, opciones…), sin nada
  de IA de por medio.
- **Desde foto (IA)**: sube una o varias capturas de una pregunta ya respondida y la IA la
  transcribe (enunciado, opciones, cuál era la correcta) directamente en el mismo formulario de
  abajo, para que la revises y la guardes. Solo tiene sentido para preguntas de opción múltiple.
- **Varias desde PDF (IA)**: el generador masivo de antes (temario, examen o simulacro → varias
  preguntas de golpe, con su propia lista de revisión separada). Ver el apartado siguiente para el
  detalle completo.

## Generar preguntas desde un PDF (temario, exámenes, simulacros)

- **Como admin**, dentro de **Añadir pregunta** (pestaña de Tests), eligiendo el método "Varias
  desde PDF (IA)" arriba, puedes subir un PDF — temario oficial, un examen ya corregido o un
  simulacro de test— y pedirle a la IA (Gemini, la misma clave/modelo que usas para transcribir
  fotos) que proponga varias preguntas de golpe para la categoría que elijas, en vez de una a una.
- Eliges la **categoría de destino**, cuántas preguntas pedir (hasta 60 por PDF) y, si quieres,
  instrucciones adicionales en texto libre (p. ej. "solo del Tema 4", "prioriza las que aparecen
  falladas en el examen"). La IA usa el temario que ya tienes configurado en esa categoría para
  intentar clasificar cada pregunta por tema/subtema automáticamente.
- **Nada se guarda solo**: las preguntas generadas aparecen en una lista de revisión con casilla
  "incluir" marcada por defecto. Puedes pulsar "Revisar / editar" en cualquiera para corregir el
  enunciado, las opciones, la correcta, el tema/subtema o la explicación antes de guardarla, o
  "Descartar" para tirarla. El botón "Guardar las marcadas" añade de golpe al banco todas las que
  sigan con la casilla activada.
- Si el PDF es un temario explicativo (sin preguntas), la IA redacta preguntas nuevas basadas en
  ese contenido; si el PDF ya trae preguntas de examen con su corrección, las transcribe tal cual.
  En ambos casos puede generar menos de las pedidas si el documento no da para tantas — mejor eso
  que preguntas inventadas.
- Los PDF escaneados como imagen (sin texto seleccionable) pueden fallar o dar peor resultado; si
  eso pasa, prueba con una versión con texto seleccionable, o pasa antes el PDF por un OCR.
- **Troceado automático de PDFs largos**: para simulacros de 80-100 preguntas, mandar el PDF entero
  en una sola llamada puede truncarse o fallar. Con la casilla "Trocear PDFs largos automáticamente"
  activada (por defecto), si el PDF principal supera el nº de páginas que indiques en "Páginas por
  llamada" (20 por defecto), se divide en varios documentos más pequeños por rango de páginas y se
  hace una llamada encadenada a la IA por cada trozo, acumulando los candidatos según van llegando
  (verás un aviso "Trozo X/Y" y cada trozo se va guardando ya en el borrador local, así que si un
  trozo falla a mitad de camino no se pierde lo generado en los anteriores). El PDF de respuestas/
  corrección (segundo archivo, opcional) se manda entero en cada llamada, sin trocear. Esta función
  depende de la librería `pdf-lib`, cargada desde un CDN externo (`cdn.jsdelivr.net`): si no hay
  conexión a ese CDN, la casilla se oculta y el PDF se manda entero como antes.
- **Contador de tiempo**: mientras genera, junto al botón aparece cuántos segundos lleva la llamada
  en curso (y, si está troceando, qué trozo está procesando), para que se note que sigue viva en
  PDFs largos que tardan más.
- **Reintentar sin perder la configuración**: si la llamada falla (sin red, JSON mal formado por el
  modelo, error de la API…), aparece un aviso con el motivo y un botón "Reintentar" que vuelve a
  lanzar la generación con el mismo PDF, categoría, cantidad e instrucciones que ya tenías puestos,
  sin tener que rellenar nada de nuevo. Si el fallo fue a mitad de un troceado, el reintento vuelve
  a empezar desde el primer trozo (las preguntas ya generadas de trozos anteriores no se pierden —
  quedan en la lista de revisión —, pero podrían volver a generarse y quedar marcadas como posible
  duplicado 🔁, lo cual está pensado para eso: revísalas y descarta la copia de más si es el caso).
- Igual que con la transcripción por foto, la clave API se guarda solo en el navegador de cada
  dispositivo.
- **Examen y respuestas en documentos separados**: si el simulacro no trae ya la solución
  incluida, puedes subir un segundo PDF opcional con la plantilla de respuestas/corrección. En
  ese caso la IA deja de redactar preguntas nuevas: transcribe cada pregunta del primer PDF
  literalmente (enunciado y opciones tal cual) y busca su respuesta correcta en el segundo PDF,
  emparejando por número de pregunta. Si para alguna pregunta no encuentra su número en el
  documento de respuestas, la marca con ⚠ como "sin respuesta correcta detectada" en vez de
  adivinar — esas quedan bloqueadas para el guardado rápido y solo se pueden guardar entrando a
  "Revisar / editar" y marcando tú la opción correcta a mano.
- **Detección de posibles duplicados**: cada pregunta generada se compara automáticamente contra
  las que ya tienes en el banco (de esa misma categoría) y contra las demás del propio lote
  recién generado. Si el parecido es muy alto (mismo examen subido dos veces, preguntas
  repetidas dentro del propio simulacro…) se marca con 🔁, se muestra con qué pregunta existente
  coincide y el porcentaje de parecido, y se desmarca por defecto para que no se cuele sola con
  el botón de guardado rápido. Puedes revisarla y guardarla igualmente si el parecido es casual.
- **Borrador automático**: las preguntas generadas y aún sin guardar en el banco se guardan solo
  en este dispositivo (no en Firebase) mientras las revisas, así que si cierras la pestaña o se
  va la conexión a medio revisar, al volver a abrir "Añadir pregunta" → "Varias desde PDF (IA)"
  las recuperas tal cual las dejaste. El borrador desaparece en cuanto guardas o descartas todas
  las preguntas pendientes.
- **Nombre del PDF de origen**: cada pregunta generada guarda el nombre del archivo del que salió
  (campo `origenArchivo`). Se ve en la tarjeta de revisión, y también en el Banco de preguntas (bajo
  las estadísticas de cada pregunta) una vez guardada; el buscador del Banco también busca por ese
  nombre de archivo, así que si detectas que un examen concreto tenía errores puedes escribir su
  nombre (o parte de él) en "Buscar" para localizar y limpiar solo esas preguntas.

## Temario (temas y subtemas)

- Cada categoría de test (Test teoría, Test inglés, Psicotécnicos, Ortografía,
  Gramática) tiene su propio temario: una lista de temas y, dentro de cada uno,
  sus subtemas. Test teoría y Test inglés vienen ya rellenos; Psicotécnicos,
  Ortografía y Gramática se dejan sin temas a propósito (solo se puede
  practicar "Todo" de esa categoría).
- Al añadir una pregunta (a mano o transcrita con IA) o al editarla desde el
  Banco de preguntas, puedes clasificarla por **tema y subtema** (con la
  opción "Todos / sin especificar" siempre disponible). Al configurar un test
  en "Practicar", eliges con casillas qué temas y subtemas concretos quieres
  incluir (o dejas "Todos los temas" marcado para usar toda la categoría).
- **Como admin**, en la pestaña **Temario** (dentro de Tests) puedes añadir,
  renombrar, reordenar (▲▼) y borrar temas y subtemas de cualquier categoría,
  para ir organizando el temario a tu gusto según avances. Los cambios se
  guardan en Firestore (`users/{tu uid}/config/temario`) y los ve cualquier
  cuenta que practique con tu banco.
- Si ya tenías `firestore.rules` publicadas de una versión anterior de la app
  (antes de que existiera esta pestaña), vuelve a pegar el contenido actualizado
  de `firestore.rules` en la consola de Firebase → Firestore Database → Reglas
  → Publicar, para que la nueva ruta `users/{tu uid}/config/{docId}` tenga
  permisos (si no, el temario no se podrá guardar ni leer).

## Practicar: pantalla "Hoy", test rápido y objetivo diario

- La portada de **Practicar** es ahora la pantalla **Hoy**: un anillo con las preguntas que llevas
  hoy frente a tu **objetivo diario** (se cambia con los botones − / +, de 5 en 5), tu racha de días
  y un botón grande **Test rápido de hoy**.
- **Test rápido**: 10 preguntas en modo instantáneo y sin temporizador, mezcladas automáticamente:
  ~40 % de tus fallos, ~30 % de repasos que te tocan hoy, ~20 % de las más flojas (acierto < 60 %
  con al menos 2 intentos), el resto de preguntas nuevas y, si aún faltan, las que llevas más
  tiempo sin repasar.
- **Repasar mis fallos / Solo nuevas**: atajos en la pantalla Hoy y opciones en el primer paso del
  asistente. En el paso de temas hay además un selector *Todas · Mis fallos · Nuevas* para
  combinarlo con una categoría y unos temas concretos. Una pregunta cuenta como "fallo" mientras su
  **último intento** fue un fallo (campo `stats.ultimo`); en preguntas antiguas, sin ese dato, cuenta
  como fallo si alguna vez se falló. En cuanto la aciertas deja de estar en la lista.
- **Pantalla de resultados**: junto a la nota muestra la comparación con el test anterior, cuántas
  preguntas has **recuperado** (las que fallabas antes y ahora aciertas), cómo vas con el objetivo
  de hoy y los botones **Repetir solo las falladas** y **Otro test rápido**.
- **Racha**: un día suma a la racha cuando respondes al menos tantas preguntas como tu objetivo
  diario (antes bastaba con empezar un test). Las preguntas de opción múltiple dejadas en blanco no
  cuentan para el objetivo. La racha antigua se conserva: se migra sola la primera vez.
- **Progreso personal sincronizado**: la racha, el objetivo y el historial de los últimos 40 tests
  se guardan en `users/{tu uid}/progreso/resumen`, así que son los mismos en móvil y ordenador.
  **Hay que volver a publicar `firestore.rules`** (Firebase → Firestore Database → Reglas →
  pegar → Publicar) para que esa ruta tenga permisos. Mientras no se publiquen, la app funciona
  igual pero solo guarda el progreso en cada dispositivo (verás un aviso en la consola del
  navegador, nada más).
- Limitación que ya existía: las estadísticas por pregunta (`stats`) las guarda solo la cuenta
  admin en el banco compartido. Una cuenta que solo practica ve sus fallos/nuevas durante la sesión,
  pero esos datos no se conservan al recargar; su racha, objetivo e historial sí se guardan.

## 🎓 Simulacro de examen

En la pantalla Hoy, botón **🎓 Simulacro de examen**. Copia la estructura del cuestionario oficial (Cabos y
Guardias 2026, tipo A) y pone la nota como en el examen real:

- **Examen de conocimientos**, por secciones y en este orden: Ortografía (5 frases = 20 palabras),
  Gramática (20 frases), Conocimientos generales (100 preguntas, ordenadas por temario) y Lengua inglesa (20).
  Los números y el **tiempo** son editables (el cuestionario oficial no indica el tiempo: pon el de tu
  convocatoria; por defecto 100 min). Las preguntas de reserva no se incluyen.
- **Psicotécnico**: 80 preguntas, 55 minutos.
- **Nota**: acierto +1, blanco 0, fallo −0,33. Ortografía y gramática, cada una por separado, **aptas con 5
  fallos o menos**. El psicotécnico es sobre 30: (aciertos − 0,33 × fallos) × 0,375. En conocimientos se
  muestran los puntos de teoría e inglés y si ortografía y gramática son aptas.
- **Reparto de teoría por temas**: por defecto el del examen oficial (las 100 preguntas de teoría del
  cuestionario 2026 clasificadas por los 35 temas del temario: p. ej. Derechos Humanos 12, Derecho Procesal 9,
  Guardia Civil 9, Constitución 5…). Con **Mi reparto en %** eliges el porcentaje de cada tema y el nº de
  preguntas de teoría; el botón **⚖️ Reforzar mis puntos débiles** parte del reparto oficial y sube el peso de
  los temas que peor llevas (flojo ×3, sin ver ×2, en camino ×1,8, dominado ×1) para que lo retoques a mano.
  Cada tema muestra cuántas preguntas tienes en el banco frente a las que pide; si no hay suficientes, avisa y
  completa con otros temas. Los temas se reconocen por su número (1., 4.1, 15…), así que reordenar el temario
  no rompe el reparto.
- Los simulacros no entran en la gráfica de evolución de los tests normales: tienen su propia lista en
  **Progreso → Tus simulacros** (nota, máximo y apto/no apto, con comparación con el anterior).
- En el **Banco → Salud del banco**, cada tema de teoría muestra «tu banco / lo que pide el examen oficial».
- Si cierras la app a mitad de un simulacro, se recupera desde Hoy con el resto de tests a medias.

## Salud del banco y preguntas por tema (Banco, solo admin)

Arriba del **Banco de preguntas** hay un panel plegable **🩺 Salud del banco y preguntas por tema**:

- **Cosas por arreglar**: cuántas preguntas no tienen tema asignado, no tienen explicación, no tienen
  respuesta correcta definida o tienen menos de 2 opciones. Cada línea tiene un botón **Ver** que filtra
  el banco por ese problema (con un aviso "Filtrando… ✕ Quitar filtro").
- **Preguntas por tema**: para cada categoría con temario, una fila por tema con el número de preguntas y
  una barra proporcional; los temas con 0 preguntas salen en rojo. Al tocar una fila se filtra el banco
  por ese tema.
- **🤖 Rellenar explicaciones** (botón de la cabecera del Banco): recorre las preguntas sin explicación
  (con enunciado y respuesta correcta), hasta 30 por tanda, y pide a la IA una explicación de cada una.
  Tú revisas y editas los textos y guardas solo las que te valgan; nada se guarda solo.

## Más herramientas para estudiar

- **Cuenta atrás al examen**: en la pantalla Hoy hay un selector de **Fecha del examen**. Con fecha puesta
  muestra los días que faltan, cuántos temas llevas dominados y el ritmo de preguntas/día que necesitas
  para ver todo lo nuevo y arreglar tus fallos a tiempo (y si tu objetivo diario se queda corto).
- **Gráfica de evolución** (Progreso): % de acierto de tus últimos 20 tests, con la media de los últimos 5
  frente a los 5 anteriores. Pasa el ratón (o mantén pulsado) sobre un punto para ver la fecha y la nota.
- **🧩 Preguntas rebeldes**: las que has fallado 3 veces seguidas, o con 4+ intentos aciertas ≤ 25 % y
  ahora estás fallando. Tienen atajo en Hoy, alcance en el asistente y filtro en el paso de temas. Al
  responderlas en un test sale un aviso.
- **⭐ Dudosas y 📝 notas personales**: cada pregunta del test tiene una estrella para marcarla como dudosa
  (se repasan luego desde Hoy o el asistente) y, una vez respondida, un botón para escribir tu propia nota
  o regla mnemotécnica (con el formato enriquecido). Son personales de cada cuenta y se guardan en tu
  progreso, no en el banco compartido.
- **Retomar un test a medias**: el test en curso se guarda en el dispositivo (respuestas, modo y tiempo
  restante). Si cierras la app o cambias de pestaña, en Hoy aparece "Tienes un test a medias" con
  Continuar / Descartar. Caduca a los 3 días.
- **Durante el test**: 🎯 modo concentración (oculta cabecera y pestañas), A− / A+ para el tamaño de la
  letra de las preguntas (se recuerda), 📳 vibración al responder en modo instantáneo (móviles
  compatibles; los iPhone no la soportan) y atajos de teclado en escritorio: **1-4 o A-D** responden a la
  pregunta sin contestar más alta que se ve en pantalla.
- **📸 Hacer foto ahora** en el modo de varias fotos: abre la cámara del móvil directamente.

Las notas, las dudosas y la fecha del examen viajan en `users/{tu uid}/progreso/resumen`, así que para
que se sincronicen entre dispositivos hay que tener publicadas las reglas de `firestore.rules`
(sin ellas funcionan igual, pero solo en cada dispositivo).

## Varias fotos → varias preguntas

En **Añadir pregunta → Desde foto (IA)** ahora hay dos modos (selector arriba):

- **Varias preguntas (1 foto = 1 pregunta)**, el modo por defecto: subes hasta **5 fotos** (se pueden
  elegir de golpe o ir añadiéndolas), cada una con UNA pregunta y su retroalimentación visible. La IA
  transcribe cada foto por separado (una llamada por foto, con el mismo respaldo Gemini → OpenRouter →
  proveedor extra de siempre): enunciado, opciones, respuesta correcta deducida de la corrección, y la
  retroalimentación copiada en la explicación. Las fotos se reducen a 1600 px antes de enviarlas.
- **1 pregunta (varias capturas)**: el modo anterior, donde todas las imágenes son una sola pregunta.

Las preguntas transcritas van a la misma lista de **"Revisa antes de guardar"** que "Desde PDF"
(puedes editarlas, cambiar tema/subtema, descartarlas o guardar todas las marcadas); no se guarda nada
en el banco hasta que lo confirmas, y avisa si alguna se parece a otra que ya tienes. Si una foto falla,
las demás siguen y la fallida se queda en la lista para reintentarla con un clic. El límite de 5 está en
la constante `MAX_FOTOS_LOTE` de `index.html`.

## Repaso espaciado

Cada vez que respondes una pregunta se calcula cuándo te toca volver a verla (`stats.nivel` y
`stats.prox`): si la fallas, mañana; cada acierto seguido la aleja más (3, 7, 14, 30 y 60 días).
La pantalla Hoy muestra **Repaso de hoy** con las preguntas que te tocan (las atrasadas se acumulan)
y cuántas hay para mañana; también es un alcance del asistente y un filtro del paso de temas. Las
preguntas antiguas, sin fecha de repaso, solo entran si están falladas; en cuanto las practiques
una vez ya tendrán su calendario.

## Mapa del temario

En **Progreso**, cada tema del temario (categorías con temas: teoría, inglés…) es una casilla de color:
**Dominado** (de las preguntas que has practicado, al menos el 85 % las tienes bien ahora y has visto
como mínimo el 70 % del tema), **En camino**, **Flojo** (menos del 60 % bien) y **Sin ver**. Arriba
se resume cuántos temas llevas dominados. Al tocar una casilla se abre el asistente de Practicar ya
con ese tema elegido.

## "Explícamelo" con IA

Al responder una pregunta (modo instantáneo, o al corregir un test) aparece **🤖 Explícamelo**
siempre que ese dispositivo tenga alguna clave de IA configurada (Gemini, OpenRouter u otro
proveedor, en *Añadir pregunta*). Usa el mismo sistema de respaldo entre proveedores que el resto de
la app y muestra una explicación breve, marcada como generada por IA para que la contrastes con el
temario. Si eres admin, **💾 Guardar como explicación** la guarda en la pregunta (pide confirmación
si ya tenía una) y desde ese momento se ve con el formato enriquecido como cualquier otra.

## Asistente de dudas legales (pestaña "⚖️ Asistente legal")

Disponible tanto para el admin como para cualquier cuenta aprobada, con dos formas de usarlo:

- **Documento guardado**: el admin sube un PDF (una ley, el reglamento, un tema del temario…) desde
  esta misma pestaña; la IA lo transcribe una única vez a texto plano (troceando el PDF por rango de
  páginas si es largo, igual que "Generar con IA", para no toparse con el límite de tokens de salida
  de una sola llamada) y lo guarda en Firestore, troceado a su vez en documentos por debajo de 1 MiB
  cada uno (`users/{ADMIN_UID}/legalDocs/{docId}` + subcolección `chunks/`). Cualquier cuenta
  aprobada puede entonces elegir ese documento de una lista y preguntarle dudas, sin volver a gastar
  cuota transcribiendo el PDF cada vez. **Recomendado subir un documento por ley/tema en vez de uno
  solo gigante con todo junto**: cada pregunta manda el texto completo del documento elegido como
  contexto a la IA, así que documentos más pequeños gastan menos tokens por pregunta.
- **Subir mi propio PDF**: cualquier usuario puede en su lugar subir un PDF puntual y preguntar sobre
  ese archivo concreto; no se guarda en ningún sitio, se manda directo a la IA como en "Añadir
  pregunta → Varias desde PDF".

Cada cuenta necesita su propia clave de IA (Gemini/OpenRouter) pegada en el mismo panel de siempre —
así el consumo de cada usuario no depende de la cuota del admin. El admin puede borrar un documento
guardado desde la misma pestaña (borrado permanente, sin papelera).

Como puede haber más de una cuenta con permiso para subir documentos (la tuya y, si la has añadido,
la de `SUPER_ADMIN_UIDS`), la lista de "Documentos guardados" muestra quién subió cada uno (su email)
y un resumen arriba con el total y cuántos ha subido cada cuenta. Los documentos guardados antes de
este cambio aparecen como "sin registrar", porque ese dato no existía todavía cuando se subieron.

## Revisar una pregunta con IA (control de calidad, solo admin)

En la vista previa de cualquier pregunta del banco, el botón **🤖 Revisar con IA** manda el
enunciado, las opciones, la marcada como correcta y la explicación a la IA, y le pide que señale
posibles fallos: respuesta mal marcada, enunciado ambiguo, dos opciones que podrían ser válidas a la
vez, o que falte información para responder. Es solo una segunda opinión — no cambia nada
automáticamente, el admin decide si corrige algo tras leer la respuesta.

## Explicaciones con formato

Al escribir la explicación de una pregunta hay una barra con **negrita**, <u>subrayado</u>,
resaltado y tres colores (rojo, verde, azul), con vista previa. El texto se guarda como string normal
con marcas: `**negrita**`, `__subrayado__`, `==resaltado==`, `{rojo:texto}` (también `verde` y
`azul`). Las explicaciones antiguas se ven igual. En Telegram se conservan negrita y subrayado; el
resto de formatos se envía como texto normal.

## Tamaño de tus datos (medidor de espacio)

En la parte de abajo de la app (encima de "Copia de seguridad") hay un bloque **"Tamaño de tus
datos"**. A diferencia de Operación Baeza, aquí **no hay un único documento gigante**: cada pregunta
se guarda en su propio documento de Firestore (`users/{admin}/testQuestions/{id}`), y el límite de
Firestore es de **1 MiB por documento**, no por colección. Por eso el banco entero (3.000-4.000
preguntas o más) no choca con ese techo. Lo que sí puede llenarse, y lo que mide el bloque:

- **Almacenamiento de este dispositivo** (`localStorage`, ~5 MB según el navegador): la app guarda
  en él una copia completa del banco, con las imágenes de Psicotécnicos en base64, para abrir sin
  conexión. Es el límite más probable si el banco crece con muchas imágenes. Si se llena, la app ya
  mostraba un aviso (⚠); ahora además ves el porcentaje **antes** de llegar ahí.
- **Pregunta más pesada** (solo admin): una pregunta con muchas imágenes podría acercarse a 1 MiB,
  y entonces Firestore no la guardaría. Debajo aparece la lista de las 5 preguntas que más pesan.
- **Tu progreso** (`users/{uid}/progreso/resumen`, también 1 MiB): racha, notas personales,
  simulacros… Tiene topes internos, pero el de las notas (hasta 1.000 notas de hasta 1.500
  caracteres) es más holgado que 1 MiB, así que en teoría podría llegar; en la práctica queda lejos.
  Es el único documento "personal" que crece.

Los avisos pasan a **ámbar al 60 %** y a **rojo al 85 %**, con un consejo según lo que se llene. Las
cifras de Firestore son una estimación (el JSON en UTF-8, sin contar unos cientos de bytes de nombres
de campo). Se recalcula solo poco después de cambiar el banco o el progreso.
Código: sección «TAMAÑO DE TUS DATOS» de `index.html` (`renderStorageBox()`), estilos `.tam-*`
en `style.css`.

Debajo del aviso hay un botón **"🧹 Vaciar caché y liberar espacio"**, disponible para cualquier
cuenta (es un ajuste de este dispositivo, no del banco compartido). Borra la copia local del banco y
del temario más la caché de archivos de la app (`caches` + desregistra el service worker), y recarga
la página; todo eso se vuelve a descargar solo en cuanto haya conexión. **No** toca tu progreso, un
test a medias, un borrador de preguntas generadas por IA sin guardar, ni tus claves de IA/Telegram. Si
hay cambios del banco sin sincronizar todavía, avisa del riesgo de perderlos antes de dejarte
continuar. Código: `limpiarCacheApp()` en `index.html`.

## Cómo publicar un cambio

1. Edita los archivos que necesites (normalmente `index.html`).
2. Si tocas el `index.html`, `manifest.json`, `service-worker.js` o los iconos,
   sube **la versión del caché** en `service-worker.js`:
   ```js
   const CACHE_NAME = 'opotest-v2'; // sube el número cada vez que despliegues
   ```
   Si no lo haces, los móviles que ya tengan la PWA instalada pueden tardar en
   ver los cambios porque siguen sirviendo la copia cacheada antigua.
3. Sube los cambios a GitHub:
   ```bash
   git add .
   git commit -m "Descripción breve del cambio"
   git push
   ```
4. Espera 1–2 minutos a que GitHub Pages despliegue, y comprueba en una pestaña
   de incógnito (para saltarte la caché del navegador) que se ve el cambio.

## Copia de seguridad de tus datos

Dentro de la app, debajo del banco de preguntas, hay un bloque **"Copia de
seguridad"** con dos botones:

- **Exportar copia (JSON):** descarga todas tus preguntas a un archivo.
- **Importar copia (JSON):** restaura las preguntas desde un archivo exportado
  anteriormente. **Sustituye todo lo que hubiera**, así que úsalo con cuidado.

## Costes (Firebase, plan Blaze de pago por uso)

Firebase tiene dos planes: **Spark** (gratis, con topes diarios fijos) y
**Blaze** (pago por uso, pero incluye gratis esos mismos topes y solo cobra lo
que pase de ahí). Con el volumen que planteas, te conviene tener el proyecto en
Blaze desde ya (hay que añadir una tarjeta, pero eso no significa que vayas a
pagar): así, si algún día un día concreto superas el tope gratuito, la app sigue
funcionando en vez de bloquearse, y lo que pases de más cuesta céntimos.

**Ahora mismo (tú solo, varios dispositivos):**
- Firestore regala 50 000 lecturas, 20 000 escrituras y 1 GiB de almacenamiento
  gratis **al día**. Con 3 000-4 000 preguntas guardadas (unos pocos MB en
  total), estás muy por debajo del límite de almacenamiento.
- Desde la versión con **sincronización incremental** (ver más abajo), abrir la
  app ya NO descarga el banco completo cada vez: solo trae las preguntas nuevas
  o modificadas desde la última vez. Aunque abras la app muchas veces al día en
  varios dispositivos, prácticamente nunca vas a acercarte a las 50 000
  lecturas/día — y aunque algún día las superases, en Blaze no pasa nada: lo
  que pase del tope gratis cuesta **$0,06 por cada 100 000 lecturas** de más.
- Authentication (usuario/contraseña) es gratis hasta 50 000 usuarios activos al
  mes, así que para ti solo no tiene coste.
- **Estimación realista para tu uso personal: $0/mes**, con margen de sobra.

**En el futuro (si la vendes a terceros):**
- El coste escala con el número de usuarios × veces que abren la app × preguntas
  nuevas/modificadas que traen cada vez (gracias a la sincronización incremental,
  ya NO es "preguntas totales del banco" en cada apertura, solo lo que haya
  cambiado). Con 5 000-6 000 preguntas por usuario, esto es mucho más barato que
  con el diseño anterior: la primera apertura en cada dispositivo sí trae el
  banco completo una vez, pero las siguientes aperturas normalmente son de
  pocas lecturas (o ninguna, si no ha habido cambios).
- Vender la app con cobros (suscripción, pago único) necesitaría además una
  pasarela de pago (Stripe es la opción más habitual), que no viene incluida en
  Firebase.

## Sincronización incremental (cómo funciona)

Para que abrir la app muchas veces al día no gaste lecturas de Firestore de
más, cada pregunta guarda un campo `actualizado` (fecha de la última vez que
se creó/editó/practicó) y este dispositivo recuerda en qué momento sincronizó
por última vez (`localStorage`). En cada apertura:

1. Se carga primero la copia local (instantáneo, funciona incluso sin
   internet).
2. Se pide a Firestore solo lo que tenga `actualizado` posterior a la última
   sincronización de este dispositivo — no el banco entero.
3. Lo nuevo se combina con la copia local y se vuelve a guardar.

Los borrados son "blandos" (la pregunta se marca con `borrado:true` en vez de
eliminarse de verdad) para que también se recojan como una actualización más
en el siguiente paso 2; el dispositivo los quita de su copia local al verlos.

La primera vez que abras la app en un dispositivo nuevo (o la primera vez tras
instalar esta mejora) sí se descarga el banco completo una única vez, para
tener una base fiable; a partir de ahí, las aperturas son incrementales.

## Tipos de pregunta y corrección (añadido)

- **Orden al practicar**: en "Practicar" hay un selector "Orden de las preguntas":
  *Desordenado (aleatorio)* (como antes) o *En orden (del 1º tema al último)*,
  que recorre las preguntas seleccionadas por tema → subtema → fecha de alta,
  útil para repasar sistemáticamente todos los casos antes del examen.
- **Psicotécnicos con imagen**: al añadir una pregunta de Psicotécnicos (o de
  cualquier categoría de opción múltiple) puedes subir una o varias imágenes
  como enunciado; se comprimen a JPEG en el propio dispositivo antes de
  guardarse (ojo: Firestore limita cada documento a 1 MB, así que no subas
  fotos ni demasiado grandes ni en gran número por pregunta).
- **Ortografía**: la pregunta son 4 palabras sueltas; al crearla marcas cuáles
  están mal escritas (pueden ser 0, algunas o las 4). Al practicar, hay que
  marcar "Bien escrita"/"Mal escrita" en las 4 (no se admite dejarlo en blanco).
- **Gramática**: la pregunta es una frase; al crearla marcas si está bien o
  mal. Al practicar, hay que marcar "Está bien"/"Está mal" (tampoco blanco).
- **Puntuación al corregir un test** (se calcula por categoría, incluso si el
  test mezcla varias):
  - *Teoría* e *Inglés*: acierto +1, fallo −0,33, blanco 0.
  - *Psicotécnicos*: acierto +1, fallo −0,33, blanco 0, y se aplica la fórmula
    `(aciertos − fallos×0,33) × 0,375` (pensada para 80 preguntas = 30 puntos).
  - *Ortografía* y *Gramática*: no hay nota numérica ni opción en blanco; cada
    palabra (ortografía) o frase (gramática) marcada al revés cuenta como
    fallo, y el resultado es **Apto** (0–5 fallos) o **No apto** (6 o más
    fallos) — el límite se cuenta por separado en cada una de las dos.

## Cosas pendientes / ideas para más adelante

- [x] Sincronización incremental del banco de preguntas (implementada: ver
      apartado de arriba).
- [x] Clasificación de preguntas por tema y subtema + panel de administración
      del temario (implementado: ver apartado "Temario" de arriba).
- [x] Orden/desorden de las preguntas al practicar, preguntas de Psicotécnicos
      con imagen, y tipos de pregunta de Ortografía/Gramática con su propia
      corrección (implementado: ver apartado de arriba).
- [x] Generación masiva de preguntas desde un PDF (temario, exámenes,
      simulacros) con pantalla de revisión antes de guardar (implementado:
      ver apartado "Generar preguntas desde un PDF" de arriba).
- [x] Detección de posibles duplicados y borrador local al generar con IA
      (implementado: ver apartado de arriba).
- [x] Trocear PDFs muy largos en varias llamadas a la IA para simulacros
      grandes, con contador de tiempo/trozo y botón de reintento sin perder
      la configuración (implementado: ver apartado "Generar preguntas desde
      un PDF" de arriba).
- [x] Guardar el nombre del PDF de origen en cada pregunta generada, para
      poder filtrar/limpiar el banco por examen (implementado: ver apartado
      de arriba, buscable desde el Banco de preguntas).
- [ ] Ajustar el nº de preguntas de cada tema de Teoría al peso real que le da
      Jefatura de Enseñanza en el examen oficial (pendiente: Alejandro tiene
      que pasar esos porcentajes por tema).
- [ ] Panel de administración simple (ver cuántos usuarios hay, cuántas
      preguntas tiene cada uno) si la app se abre a más gente.
- [ ] Cobro (Stripe u otra pasarela) si se decide vender el acceso.
- [ ] Firebase Hosting como alternativa/respaldo a GitHub Pages (ya está
      preparado en `firebase.json`, solo faltaría ejecutar `firebase deploy`).
