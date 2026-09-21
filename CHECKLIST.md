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
      cambio de reglas no sirve de nada hasta que se publica.
