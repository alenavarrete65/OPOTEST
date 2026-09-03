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
| `index.html`           | Toda la app (HTML + CSS + JS en un único archivo)              |
| `manifest.json`        | Metadatos de la PWA (nombre, iconos, colores)                  |
| `service-worker.js`    | Caché offline del "app shell"                                  |
| `icon-192.png` / `icon-512.png` | Iconos de la app                                       |
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

## Generar preguntas desde un PDF (temario, exámenes, simulacros)

- **Como admin**, en la pestaña **Generar con IA** (dentro de Tests) puedes subir un PDF —
  temario oficial, un examen ya corregido o un simulacro de test— y pedirle a la IA (Gemini,
  la misma clave/modelo que usas para transcribir fotos) que proponga varias preguntas de golpe
  para la categoría que elijas, en vez de una a una.
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
- Los PDF muy largos o escaneados como imagen (sin texto seleccionable) pueden fallar o dar peor
  resultado; si eso pasa, prueba a subir el PDF por partes más cortas.
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
- [ ] Ajustar el nº de preguntas de cada tema de Teoría al peso real que le da
      Jefatura de Enseñanza en el examen oficial (pendiente: Alejandro tiene
      que pasar esos porcentajes por tema).
- [ ] Panel de administración simple (ver cuántos usuarios hay, cuántas
      preguntas tiene cada uno) si la app se abre a más gente.
- [ ] Cobro (Stripe u otra pasarela) si se decide vender el acceso.
- [ ] Firebase Hosting como alternativa/respaldo a GitHub Pages (ya está
      preparado en `firebase.json`, solo faltaría ejecutar `firebase deploy`).
