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

## Autenticación (usuario/contraseña) y aprobación por administrador

- La app usa Firebase Authentication con correo y contraseña. Cada persona que
  entra tiene su propio banco de preguntas, aislado del de cualquier otra
  (las reglas de Firestore lo garantizan a nivel de servidor, no solo en la
  interfaz).
- Para usarla en otro dispositivo, entra con el mismo correo y contraseña, no
  hace falta ningún enlace ni código.
- Si olvidas la contraseña, el botón "¿Olvidaste tu contraseña?" te envía un
  correo de recuperación (lo gestiona Firebase automáticamente).
- **Cuentas nuevas quedan pendientes de aprobación.** Cualquiera puede pulsar
  "Crear cuenta nueva", pero hasta que el administrador (tú) la apruebe desde
  el panel de administración dentro de la app, esa persona solo ve la pantalla
  "Cuenta pendiente de aprobación" y no puede leer ni escribir ninguna
  pregunta — esto lo garantizan las reglas de Firestore, no solo la interfaz.
- **Conviértete en administrador (una sola vez, imprescindible):**
  1. Entra en la app con tu propia cuenta (créala si no la tienes ya).
  2. Ve a la consola de Firebase → **Firestore Database** → pestaña **Datos** →
     colección `users` → abre el documento con tu `uid` (se crea solo la
     primera vez que entras).
  3. Cambia a mano los campos `aprobado` a `true` y `esAdmin` a `true`, y
     guarda.
  4. Recarga la app: ahora verás un bloque **"Panel de administración"** debajo
     del banco de preguntas, con la lista de cuentas registradas.
- **Aprobar gente nueva:** cuando alguien se registre, aparecerá en "Pendientes"
  dentro del panel de administración; pulsa **Aprobar** y ya podrá entrar. Para
  quitarle el acceso más adelante, pulsa **Revocar acceso** junto a su correo.
- El rol de administrador (`esAdmin`) solo se puede cambiar a mano desde la
  consola de Firebase, nunca desde la app — así nadie puede dárselo a sí mismo.
- Cuando llegue el momento de vender la app a terceros, cada comprador crea su
  propia cuenta desde la pantalla de acceso y tú la apruebas desde el panel —
  el modelo de datos (`users/{uid}/testQuestions/...` + `users/{uid}.aprobado`)
  ya está pensado para eso, no haría falta ningún cambio de estructura.

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

## Cosas pendientes / ideas para más adelante

- [x] Sincronización incremental del banco de preguntas (implementada: ver
      apartado de arriba).
- [x] Cuenta de administrador que aprueba manualmente las cuentas nuevas que se
      registren (implementada: ver apartado "Autenticación (usuario/contraseña)
      y aprobación por administrador").
- [ ] Panel de administración más completo (ver cuántas preguntas tiene cada
      usuario) si la app se abre a más gente.
- [ ] Cobro (Stripe u otra pasarela) si se decide vender el acceso.
- [ ] Firebase Hosting como alternativa/respaldo a GitHub Pages (ya está
      preparado en `firebase.json`, solo faltaría ejecutar `firebase deploy`).
