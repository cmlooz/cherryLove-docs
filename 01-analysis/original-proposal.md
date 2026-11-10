---
id: AN-000
status: approved
approved_by: Camilo
approved_at: 2026-11-09
note: Original proposal, verbatim (Spanish). Source for every REQ marked you.
---

Actua como un analista de software senior con mucha atención al detalle que está en control de todo y que me ayudará a crear el mejor prompt posible en cada situación.
Lee esta propuesta de app, crea los requerimientos y sin asumir nada, me preguntas como proceder recomendando una opción desde tu perspectiva como senior, vamos a dejar absolutamente todo detallado y documentado incluyendo el ux/ui, todo debe ser aprobado por mi y guardaremos en un repo aparte los documentos generados, vamos a tener un analisis, plan de aprendizaje, plan de implementación, plan de pruebas, evidencias. Antes de hacer algo definiremos los requerimientos, roles, actuadores, acciones y si tienes algo extra que me recomiendes y podamos hacer genial.

La app se llama CherryLove por lo que ese juego de palabras inspira el sistema en todo.

Quiero tener un reto personal de programación en el que ya tengo en mente dos apps que te voy a indicar ahora pero básicamente quiero aprender varias cosas mientras desarrollo el proyecto.
1, es una app de parejas con varios modulos (cada uno es listado con gestión con opción de visualizar detalles desde click), que son los siguientes:
Con opción de trabajar offline y sincronizar con conexión

a. cartas diarias (simple pero poderosa), modulo con 2 pantallas (react)

Cada día muestra si hay la carta del dia, sino una aleatoria de las pasadas, pero la misma a los dos usuarios de la pareja donde vemos el contenido, quien de los y cuando fue creado ese mensaje.
Se puede hacer:
Cada usuariuo puede Guardar mensajes en local (JSON o SQLite) para sincronizar si esta offline.
Va haber una pantalla principal para mostrar el mensaje del dia pero tambien hay una pantalla para crear/editar mensajes y ver los que cada uno creó antes.
Programar desbloqueo por fecha, se puede crear mensajes para publicar luego.

b. Álbum interactivo de recuerdos, modulo con 2 pantallas (react)

Galería scroll infinito automatico sobre las fotos de cada recuerdo (recuerdos agrupados por fecha, fechas con una sola foto pasan a la sigueinte) hasta que el usuario interactura con la foto, de fotos con:
Descripciones
Funcionalidad:
Al tocar una foto → toggle historia o anécdota (descripcion).
Al visualizar una foto → si tiene relación con una canción suena de fondo.
Botón star en la esquina de la foto → click para agregar recuerdo a favoritos.
Pantalla para crear los recuerdos donde podemos guardar las fotos o videos desde: el link de un servidor externo o como opción subir el archivo y mantener una referencia en local que este disponible en el entorno de la pareja, campo de texto para descripcion de la foto, campo de fecha de la foto, selector de canciones agregadas (opcional, solo 1), pin de ubicación (opcional).
ve lo creaod por la pareja, editar y eliminar unicamente las fotos que el usuario ha creado en una lista dedse donde se despliega el formulario de creacion.

c. Reproductor de “nuestras canciones”

Pantalla donde se reproduce la lista de canciones importantes
Cada canción puede tener:
Nota personal
Galeria de recuerdos relacionados mientras suena la cancion
Botón star en la esquina del reproductor → click para agregar canción a favoritos.

Pantalla para crear las canciones donde podemos guardar los archivos desde: el link de un servidor externo, tambien spotify o youtube, etc o subir el archivo y mantener una referencia en local que este disponible en el entorno de la pareja,  campo de texto para descripcion de la canción, selector de recuerdos (opcional, multiple).

d. Mapa de lugares importantes

Muestra sitios donde han estado juntos
Cada punto tiene un recuerdo asociado:
Fotos
Historia
Fecha
Canción

Botón star en la esquina del visualizador → click para agregar ubicación a favoritos.

Resumen de ubicaciones como dashboard del modulo, al dar click sobre 1 ubicaci3n se abre el detalle antes mencionado.

e. App tipo “diario de amor” (notebook)

Puedes escribir entradas:
Cosas que sientes
Momentos especiales
La pareja puede leerlas dentro de la app

Pantalla de listado de notas con autor y fecha de creacion, al dar click se abre el lector.

f. Mis favoritos
Lista de todos los recuerdos (todas las fotos de la misma fecha se agrupan), canciones, lugares, notas donde al hacer click vemos el detalle de cada elemento.

g. Mi cuenta
Cambiar foto de perfil
Ver nombre, fecha de cumpleaños y pareja
Cambiar contraseña

Al entrar a la app veremos un dashboard con el timeline de los favoritos y "{couple.user1.name} y {couple.user2.name} {couple.daystogheter}" un contador de días juntos.
Cada uno edita/borra lo suyo pero ve lo de ambos, se guarda log de todo

2, API gestora y app administradora de parejas, accesos y permisos (django)
Todos los metodos CRUD de los modulos de las apps de los usuarios
Cada petición valida si el usuario es de la pareja, nunca mostramos datos de otras parejas.
Gestion de recursos de las parejas
Persistencia de los datos en base de datos
Administrar las relaciones entre parejas y usuarios (solo 2 usuarios por pareja, solo 1 pareja por usuario, sin cambio de pareja por usuario), enviando invitaciones a los correos de los usuarios y que llenen el form con sus datos (nombre, fecha de nacimiento, fecha de inicio de relacion, si uno ya lo llena al otro no se le pide).
Dashboard de parejas activas (registro de login y elementos creados y visitados por semana y mes).
Registro de logs con filtro por tipo.

3, APP para compilar android y iOs
Mismas funcionalidades que la versión web
Con opción de trabajar offline y sincronizar con conexión

4, PostgresQL como motor de base de datos, cada capa del sistema debe desconocer las demas y poder integrar o uno o el otro sin ningun problema y si luego me creo otra API u otra UI debería todo funcionar sin problemas, quiero la menor cantidad de dependencias posibles.
Si luego se implementan microservicios o distintas APIS el sistema debería ser capaz de soportarlo.

Las aplicaciones deben aplicar los principios de alta seguridad, rapida respuesta, alto volumen de capacidad, arquitectura escalable, limpia, a prueba de errores, trazabilidad de acciones, principios solid, escalable, legible, siempre buscando el mejor código no solo el que funcione, optimizadas, en base a la documentación oficial.
Tengo mas modulos en mente pero por ahora haremos estos pensando en implementarlos luego dejaremos las bases bien estructuradas.

En base a las apps que hagan esto, quiero aprender a programar con React para hacer una web responsive, ReactNative para hacer una versión mobile y Django con Python para toda la administración del sitio y rol de API. Tengo algo de experiencia con Vue, con python si he trabajado poco, pero no django, quiero pasar de 0 a experto en estos 2 frameworks.
en si la app y los lenguajes/frameworks son el canal pero lo importante que si quiero aprender, conocer y creer en expertice es en la arquitectura de software, quiero ser arquitecto con conocimentos avanzados y poder decidir cuando sirve mas una u otra estructura de proyectos, tengo 0 conocimientos sobre arquitectura, solo he sido desarrollador sobre proyectos ya en curso, en proyectos nuevos no sabría como empezar a distribuir las cargas en controladores, interfaces, middlewares, modelos, etc, y en sí como deberían estar los archivos. Quiero aprender a hacer proyectos escalables, con buen manejo de errores, baja complejidad de base de datos y alta velocidad de respuesta, capaz de un entorno con grandes cantidades de registros, excelente gestión de recurso del servidor, que sean faciles de leer y debuguear y sobretodo donde se reutilice la mayor cantidad de codigo posible, que se tengan buenos protocolos de seguridad y de red en la app en como se comunica el front con el back, donde hayan roles y permisos bien definidos.
Vamos a crear un plan de aprendizaje y tres planes de implementación (1 por app) por fases y pasos en los que: cada implementación tiene su entrada de conocimientos previos para proceder en el aprendizaje, el plan de implementación debe incluir 1 commit por paso describriendo lo realizado, 1 PR por fase, una prueba unitaria y con el plan de aprendizaje vamos a crar 1 jupiternotebook por lección que se base en la documentación oficial respectiva y que incluya ejercicios por paso y un test por fase.

Siempre con el objetivo de implementar el mejor algoritmo y el codigo mas limpio y legible, siempre que se pueda tipado, priorizando el uso de validaciones antes que try/catch.
Nunca asumir y siempre validar de nuevo, preguntar cuando sea necesario.

Necesito que cremos los agentes y skills necesarios para llevar el proyecto a cabo con la mejor calidad.

Expreso muchas cosas aquí por lo que vamos a necesitar diferentes sesiones para todos estos requerimientos por lo que antes de hacer cualquier cosa revisa que se puede paralelizar para crear su respectiva sesión y su contexto.
