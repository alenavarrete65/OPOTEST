# OpoTest — checklist antes de publicar

No hay tests automatizados (montar Playwright/CI para un único archivo HTML sin
proyecto Node ni repositorio con integración continua es más infraestructura de
la que compensa ahora mismo). Esta lista es el sustituto práctico: un repaso de
5-10 minutos antes de subir un cambio a producción, para pillar roturas antes de
que las note un usuario.

## Antes de nada
- [ ] `index.html` no da errores en la consola del navegador al cargar (F12 → Console).
- [ ] Sube el número de `CACHE_NAME` en `service-worker.js` si has tocado
      `index.html`, `manifest.json` o los iconos (si no, algunos usuarios seguirán
      viendo la versión vieja cacheada).

## Cuenta e inicio de sesión
- [ ] Entrar con una cuenta aprobada (rol lectura) carga el banco de preguntas.
- [ ] Entrar con una cuenta aprobada (rol admin) puede añadir/editar preguntas.
- [ ] Una cuenta nueva (sin aprobar) ve el mensaje de "pendiente de aprobación" y
      no puede entrar al banco.
- [ ] Cerrar sesión y volver a entrar no rompe nada (recarga bien el progreso).

## Tests y simulacro
- [ ] Hacer un test normal corto, responder todas, corregir: la corrección y el
      contador de aciertos son correctos.
- [ ] Iniciar un simulacro: el cronómetro cuenta hacia atrás y el aviso de los
      últimos 5 minutos suena y vibra (puedes forzarlo bajando `SIM_AVISO_SEGUNDOS`
      un momento para probarlo, y devolverlo a `5*60` después).
- [ ] Cerrar la pestaña a mitad de un test y volver a abrirla: ofrece "continuar"
      con las respuestas ya dadas.
- [ ] Modo audio: el botón 🔊 de una pregunta suelta la lee bien, y el modo
      audio general de la barra de herramientas encadena varias preguntas.
- [ ] El selector de velocidad del modo audio cambia el ritmo de la voz.

## Progreso
- [ ] La racha, el objetivo diario y el mapa del temario reflejan lo que acabas
      de practicar.
- [ ] El recordatorio inteligente aparece cuando corresponde (prueba forzando
      `UMBRAL_OLVIDO_DIAS` a 0 un momento) y el botón "Repasarlo ahora" lleva al
      tema correcto.
- [ ] "Exportar informe (PDF)" descarga un PDF legible, sin texto cortado ni
      símbolos raros.
- [ ] La comparativa entre dispositivos muestra al menos el dispositivo actual
      tras terminar un test.

## Panel de administración (solo cuenta dueña)
- [ ] El resumen de cuentas (total, aprobadas, pendientes, preguntas en el banco)
      carga sin quedarse en "Cargando…".
- [ ] El buscador por email y el filtro por estado funcionan.
- [ ] Aprobar/revocar una cuenta de prueba se refleja al momento.
- [ ] Si has añadido un segundo UID a `SUPER_ADMIN_UIDS` (en `index.html` y en
      `firestore.rules`), esa cuenta de respaldo también puede aprobar/revocar
      cuentas y dar el rol admin, no solo la cuenta original.

## Asistente legal ("⚖️ Asistente legal", dentro de Tests)
- [ ] Como admin, subir un PDF pequeño con título: la IA lo transcribe y aparece
      en la lista de "Documentos guardados" con su nº de caracteres/trozos.
- [ ] Con una cuenta de lectura (con su propia clave de Gemini pegada), elegir
      ese documento guardado y hacer una pregunta sobre su contenido: la
      respuesta se ciñe al texto del documento (y dice que no lo sabe si
      preguntas algo que no está en él, en vez de inventar).
- [ ] Con "Subir mi propio PDF" (sin guardarlo), preguntar algo sobre un PDF
      cualquiera y comprobar que responde sin necesidad de que el admin lo suba
      antes.
- [ ] Marcar dos documentos guardados a la vez (checkboxes) y hacer una
      pregunta que cruce ambos: la respuesta indica de cuál sale cada dato.
- [ ] "🕘 Historial reciente" muestra las últimas preguntas/respuestas al final
      de la pestaña, y "Borrar historial" lo vacía.
- [ ] Cada respuesta del asistente muestra arriba "📄 Documento: …" con el
      título guardado (o el nombre del PDF, en modo "Subir mi propio PDF").
- [ ] Borrar un documento guardado como admin: desaparece de la lista y ya no
      sale como opción al preguntar.
- [ ] Probar con un PDF largo (bastantes páginas): se sube por trozos (verás el
      contador "Trozo X/Y") en vez de fallar o cortarse a mitad.

## Revisar pregunta con IA (solo admin)
- [ ] Abrir la vista previa de una pregunta desde el Banco y pulsar
      "🤖 Revisar con IA": aparece una respuesta corta debajo, sin bloquear el
      resto del modal.
- [ ] Probar con una pregunta a la que le cambias la correcta a una opción
      equivocada a propósito: la IA debería avisar de que algo no cuadra.
- [ ] "🤖 Revisar en bloque" (junto a "Rellenar explicaciones"): filtrar por
      categoría, poner una tanda pequeña (p. ej. 3) y darle a "Revisar":
      aparece el progreso, los resultados uno a uno, y "Detener" corta el
      proceso a mitad sin romper nada. "Ver pregunta" abre cada una.

## Reportar pregunta (cualquier cuenta aprobada)
- [ ] Al corregir un test, el botón "🚩 Reportar esta pregunta" aparece junto
      al de "Explícamelo".
- [ ] Enviar un reporte (con y sin motivo escrito): se ve "Enviando y
      revisando con IA…" y luego "✓ Reportada".
- [ ] Como admin, en "Banco de preguntas" aparece arriba el panel
      "🚩 Preguntas reportadas" con el motivo del usuario y el veredicto de la
      IA; "Ver pregunta" la abre y "✓ Marcar resuelto" la quita de la lista.
- [ ] La pestaña "Banco de preguntas" en la navegación muestra un 🚩 con el
      número de reportes abiertos, sin tener que entrar a mirar.
- [ ] "Ver resueltos" despliega los ya resueltos, y "Borrar todos los
      resueltos" los quita de Firestore (no toca las preguntas en sí).
- [ ] Sin las reglas de `reportes` desplegadas en Firestore, reportar debe dar
      un error claro, no quedarse colgado.

## Uso de IA y revisión en bloque
- [ ] Tras hacer alguna pregunta a la IA (asistente, revisar, generar...), el
      panel de claves de IA muestra "📊 Hoy has hecho N llamada(s)".
- [ ] En "🤖 Revisar en bloque", revisar una tanda pequeña y luego repetir la
      misma categoría: la segunda vez empieza por preguntas distintas a la
      primera (prioriza las nunca revisadas), no por las mismas de siempre.
- [ ] "Tamaño de tus datos" muestra líneas de "Documentos legales" y
      "Reportes de preguntas" cuando ya has visitado esas pestañas antes.

## Tamaño de tus datos
- [ ] El bloque "Tamaño de tus datos" aparece (encima de "Copia de seguridad") con las
      barras y sin errores en la consola.
- [ ] Con la cuenta admin salen 3 barras y la lista "Lo que más ocupa"; con una cuenta de
      lectura salen solo 2 barras y sin lista.
- [ ] Tras guardar una pregunta con imagen, las cifras se actualizan solas en un par de segundos.
- [ ] "🧹 Vaciar caché y liberar espacio": tras confirmar, recarga sola y el banco/temario se
      vuelven a descargar bien (con conexión). Tu progreso, un test a medias y las claves de
      IA/Telegram siguen intactos después de usarlo.
- [ ] Con algún cambio sin sincronizar a propósito (edita una pregunta estando en avión), el
      botón avisa del riesgo antes de dejarte continuar.

## Copia de seguridad
- [ ] "Exportar copia (JSON)" descarga un archivo con las preguntas.
- [ ] Importar esa misma copia no cambia el número de preguntas del banco.
- [ ] El aviso de "han pasado X días desde tu última copia" desaparece nada más
      exportar una nueva.

## Multidispositivo (si tienes forma de probarlo)
- [ ] Un cambio hecho en el banco desde un dispositivo aparece en otro al recargar.
- [ ] El tema (oscuro/claro/automático por hora) se recuerda en cada dispositivo
      por separado.

## Firestore
- [ ] Si has tocado `firestore.rules`, las has vuelto a desplegar
      (`firebase deploy --only firestore:rules` o pegadas en la consola) — un
      cambio de reglas no sirve de nada hasta que se publica. Esto incluye las
      reglas nuevas de `legalDocs` la primera vez que las subas.
