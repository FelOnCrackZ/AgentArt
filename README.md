# AgentArt

> La agenda cultural de tu ciudad.

AgentArt es un portal web donde organizadores locales publican conciertos, ferias y talleres, y el público los descubre, los filtra y guarda sus favoritos. Cada evento pasa por una moderación antes de publicarse.

## Contenido

1. [Integrantes del proyecto](#1-integrantes-del-proyecto)
2. [Idea de desarrollo web elegida](#2-idea-de-desarrollo-web-elegida)
3. [Requerimientos funcionales](#3-requerimientos-funcionales)
4. [Requerimientos no funcionales y Definition of Done](#4-requerimientos-no-funcionales-y-definition-of-done)
5. [Modelado de la base de datos](#5-modelado-de-la-base-de-datos)

---

## 1. Integrantes del proyecto

- Felipe Nuñez
- Cristian Arango
- Miguel Patiño
- Sharon Zapata
- Jhoan Chavez

---

## 2. Idea de desarrollo web elegida

**Agenda de eventos culturales locales.** Un portal donde los organizadores publican conciertos, ferias y talleres, y el público los descubre y marca su interés.

### ¿Qué problema resuelve?

La oferta cultural local suele estar dispersa en publicaciones sueltas, afiches y grupos de mensajería, lo que dificulta encontrarla y compararla. AgentArt la reúne en un solo lugar, con información ordenada y confiable, y le da a los organizadores un canal propio para llegar a más gente.

### ¿Cómo funciona?

- **El público** explora el listado, filtra por fecha, categoría, ciudad y precio, busca por texto, consulta el detalle con imágenes y ubicación, y guarda favoritos. También puede compartir un evento en redes o agregarlo a su calendario.
- **Los organizadores** se registran y crean, editan o eliminan sus eventos, con uno o varios afiches.
- **El administrador** revisa cada evento antes de publicarlo y lo aprueba o rechaza indicando el motivo.

### Análisis inicial (las 5 preguntas)

| Pregunta | Respuesta |
|---|---|
| 1. Usuarios y crecimiento | 5.000 visitantes mensuales, duplicándose cada ~3 meses. |
| 2. Read o write-heavy | Muy read-heavy: cerca de 99 % lecturas y 1 % escrituras. Solo los organizadores publican. |
| 3. Datos críticos | Nunca perder eventos publicados ni cuentas de organizadores. Se tolera desfase en contadores (visitas, "me interesa") y recomendaciones. |
| 4. Latencia | Listado y detalle < 200 ms. Publicar un evento con imagen: 2 a 3 s. |
| 5. Costo | Páginas cacheadas en CDN, imágenes en almacenamiento de objetos y una base de datos pequeña. Casi todo el tráfico no toca el servidor. |

### Estimación de costos

| Fase | Componentes | USD/mes | COP/mes |
|---|---|---|---|
| Lanzamiento (5.000 visitantes) | Hosting estático con CDN gratis, base de datos y almacenamiento de imágenes en nivel gratuito | 0 | 0 |
| ~9 meses (40.000 visitantes) | Plan de pago de la base de datos; con CDN casi nada llega a ella | 25 | ≈ 88.000 |

Aparte va el dominio (COP 45.000 a 105.000 al año, estimado). Total realista: entre COP 0 y 100.000 al mes. El costo principal es el tiempo de moderación.

### Tecnologías sugeridas (aún no claras todavía)

| Capa | Tecnología |
|:---:|---|
| Frontend | React |
| Backend | Node.js (posiblemente) |
| Base de datos | PostgreSQL o MySQL |
| Despliegue | Docker y GitHub Actions |

---

## 3. Requerimientos funcionales

### Requisitos

| ID | Requisito |
|---|---|
| RF-01 | Listar y filtrar eventos por fecha, categoría, ciudad y precio. |
| RF-02 | Ver el detalle del evento (descripción, ubicación, imágenes). |
| RF-03 | Registrar organizadores e iniciar sesión. |
| RF-04 | Crear, editar y eliminar los eventos propios. |
| RF-05 | Subir imágenes o afiches del evento. |
| RF-06 | Marcar "me interesa" y guardar favoritos. |
| RF-07 | Buscar eventos por texto. |
| RF-08 | Compartir en redes y agregar el evento al calendario. |
| RF-09 | Moderación: el administrador aprueba o rechaza eventos. |
| RF-10 | Registro e inicio de sesión de usuarios del público. |
| RF-11 | Mostrar eventos similares en el detalle. |
| RF-12 | Contar las visitas de cada evento. |

### Épicas e historias de usuario

Las historias usan el formato *Como* [usuario] *quiero* [acción] *para* [beneficio]. Prioridad: Alta, Media o Baja. Estimación en puntos de historia (escala Fibonacci). El sprint indica en qué iteración se desarrolla cada una.

| Épica | Descripción | Pts |
|---|---|---|
| **EP-01 · Descubrimiento de eventos** | El público encuentra eventos con listados, filtros, búsqueda y detalle. | 21 |
| **EP-02 · Cuentas y autenticación** | Registro, inicio de sesión y recuperación de acceso para cada tipo de usuario. | 17 |
| **EP-03 · Gestión de eventos** | Los organizadores publican y administran sus eventos e imágenes. | 24 |
| **EP-04 · Moderación** | El administrador revisa y decide sobre los eventos. | 10 |
| **EP-05 · Interacción del público** | Favoritos, compartir y calendario. | 10 |

#### EP-01 · Descubrimiento de eventos

| Código | Historia | Prioridad | Pts | Sprint |
|:---:|---|:---:|:---:|:---:|
| HU-01 | **Listado de eventos.** Como visitante, quiero ver un listado de eventos próximos, para descubrir qué hay en mi ciudad.<br><br>*Criterios de aceptación*<br>• Solo se muestran eventos aprobados, no eliminados y con fecha vigente.<br>• Cada tarjeta muestra título, fecha, ciudad, precio e imagen principal.<br>• Carga en menos de 200 ms (desde caché/CDN). | Alta | 3 | 3 |
| HU-02 | **Filtros de eventos.** Como visitante, quiero filtrar por fecha, categoría, ciudad y precio, para encontrar eventos que se ajusten a lo que busco.<br><br>*Criterios de aceptación*<br>• Los filtros se pueden combinar.<br>• Un evento gratuito tiene `precio = 0` y se filtra como "gratis".<br>• Si no hay resultados, se muestra un mensaje claro.<br>• La URL conserva los filtros para poder compartirla. | Alta | 5 | 3 |
| HU-03 | **Detalle del evento.** Como visitante, quiero ver la descripción, dirección, horario, precio e imágenes de un evento, para decidir si asisto.<br><br>*Criterios de aceptación*<br>• Las imágenes se ordenan según el campo `orden`, con la principal primero.<br>• Se muestra el organizador y la cantidad de interesados.<br>• La página es indexable por buscadores (HTML semántico). | Alta | 5 | 3 |
| HU-04 | **Búsqueda por texto.** Como visitante, quiero buscar eventos por palabras clave, para encontrar uno específico sin recorrer todo el listado.<br><br>*Criterios de aceptación*<br>• La búsqueda cubre título y descripción, con índice de texto completo.<br>• Se puede combinar con los filtros. | Media | 5 | 3 |
| HU-22 | **Eventos similares.** Como visitante, quiero ver eventos parecidos en el detalle, para descubrir más opciones.<br><br>*Criterios de aceptación*<br>• Se sugieren eventos vigentes y aprobados de la misma categoría y ciudad.<br>• Máximo 4, con lista cacheada; puede estar levemente desactualizada. | Baja | 3 | 4 |

#### EP-02 · Cuentas y autenticación

| Código | Historia | Prioridad | Pts | Sprint |
|:---:|---|:---:|:---:|:---:|
| HU-10 | **Registro de organizadores.** Como organizador, quiero crear una cuenta con mis datos (nombre, correo, teléfono, descripción), para poder publicar eventos.<br><br>*Criterios de aceptación*<br>• El correo es único y la contraseña se guarda como hash.<br>• Hay protección contra spam (por ejemplo, captcha). | Alta | 3 | 1 |
| HU-11 | **Inicio y cierre de sesión.** Como organizador, quiero iniciar y cerrar sesión de forma segura, para gestionar mis eventos.<br><br>*Criterios de aceptación*<br>• Solo veo y edito mis propios eventos. | Alta | 3 | 1 |
| HU-17 | **Acceso del administrador.** Como administrador, quiero acceder a un panel protegido, para moderar contenido.<br><br>*Criterios de aceptación*<br>• Es una cuenta distinta a las de organizador y usuario.<br>• Las cuentas de administrador se crean manualmente. | Alta | 3 | 1 |
| HU-07 | **Registro del público.** Como persona del público, quiero crear una cuenta con correo y contraseña, para guardar mis eventos de interés.<br><br>*Criterios de aceptación*<br>• El correo es único.<br>• La contraseña se guarda como hash.<br>• Si el correo ya existe, se muestra un error claro. | Media | 5 | 4 |
| HU-24 | **Recuperación de contraseña.** Como usuario u organizador, quiero restablecer mi contraseña por correo, para recuperar el acceso si la olvido.<br><br>*Criterios de aceptación*<br>• El enlace expira en 24 horas y es de un solo uso.<br>• La respuesta no revela si el correo existe. | Media | 3 | 4 |

#### EP-03 · Gestión de eventos (organizador)

| Código | Historia | Prioridad | Pts | Sprint |
|:---:|---|:---:|:---:|:---:|
| HU-12 | **Crear evento.** Como organizador, quiero publicar un evento con título, descripción, fechas, precio, dirección, categoría y ciudad, para que el público lo conozca.<br><br>*Criterios de aceptación*<br>• Se validan los campos obligatorios; `fecha_fin` no puede ser anterior a `fecha_inicio`.<br>• El evento queda con `estado_moderacion` = "pendiente" (valores: pendiente, aprobado, rechazado).<br>• Publicar con imagen toma entre 2 y 3 s. | Alta | 5 | 1 |
| HU-13 | **Subir imágenes.** Como organizador, quiero subir uno o varios afiches y definir cuál es la principal y su orden, para que el evento sea atractivo.<br><br>*Criterios de aceptación*<br>• Solo se aceptan formatos y tamaños permitidos.<br>• Solo una imagen puede ser la principal (índice único parcial sobre `es_principal`). | Alta | 8 | 1 |
| HU-14 | **Editar evento.** Como organizador, quiero modificar los datos de un evento mío, para corregir errores o actualizar información.<br><br>*Criterios de aceptación*<br>• Si cambio datos sensibles, el evento vuelve a moderación. | Alta | 3 | 2 |
| HU-15 | **Eliminar evento.** Como organizador, quiero eliminar un evento mío, para que deje de mostrarse al público.<br><br>*Criterios de aceptación*<br>• Se pide confirmación.<br>• La eliminación es lógica (campo `eliminado`), no se pierde el registro. | Media | 2 | 2 |
| HU-16 | **Estado de mis eventos.** Como organizador, quiero ver si mis eventos están pendientes, aprobados o rechazados, y el motivo del rechazo, para corregirlos y volver a enviarlos.<br><br>*Criterios de aceptación*<br>• El motivo registrado en la moderación es visible para el organizador. | Media | 3 | 2 |
| HU-23 | **Contador de visitas.** Como organizador, quiero ver cuántas visitas tiene mi evento, para medir su alcance.<br><br>*Criterios de aceptación*<br>• Se incrementa `contador_visitas` por lotes o en segundo plano, sin frenar la página cacheada.<br>• Se tolera un desfase en el valor mostrado. | Media | 3 | 3 |

#### EP-04 · Moderación (administrador)

| Código | Historia | Prioridad | Pts | Sprint |
|:---:|---|:---:|:---:|:---:|
| HU-18 | **Cola de moderación.** Como administrador, quiero ver la cola de eventos por moderar, para revisarlos en orden.<br><br>*Criterios de aceptación*<br>• Se ordenan por `fecha_creacion` (los más antiguos primero). | Alta | 3 | 2 |
| HU-19 | **Aprobar evento.** Como administrador, quiero aprobar un evento, para que se publique en el portal.<br><br>*Criterios de aceptación*<br>• Se crea un registro en Moderación con decisión, fecha y administrador.<br>• El evento aparece en listados y detalle (con la caché actualizada). | Alta | 3 | 2 |
| HU-20 | **Rechazar evento.** Como administrador, quiero rechazar un evento indicando el motivo, para que el organizador sepa qué corregir.<br><br>*Criterios de aceptación*<br>• El motivo es obligatorio.<br>• El evento no se muestra al público. | Alta | 2 | 2 |
| HU-21 | **Historial de moderación.** Como administrador, quiero ver las decisiones tomadas sobre cada evento, para tener trazabilidad.<br><br>*Criterios de aceptación*<br>• Se listan las decisiones por evento con fecha, administrador y motivo. | Baja | 2 | 2 |

#### EP-05 · Interacción del público

| Código | Historia | Prioridad | Pts | Sprint |
|:---:|---|:---:|:---:|:---:|
| HU-08 | **Marcar favorito.** Como usuario registrado, quiero marcar un evento como favorito ("me interesa"), para encontrarlo después fácilmente.<br><br>*Criterios de aceptación*<br>• No se puede marcar el mismo evento dos veces (clave compuesta usuario + evento).<br>• Se registra la fecha en que se marcó.<br>• El contador de interesados se actualiza con desfase tolerado (consistencia eventual). | Media | 3 | 4 |
| HU-09 | **Gestionar favoritos.** Como usuario registrado, quiero ver mi lista de favoritos y quitar los que ya no me interesan, para mantenerla ordenada.<br><br>*Criterios de aceptación*<br>• Se ordena por fecha del evento.<br>• Si un evento fue eliminado o rechazado, deja de aparecer. | Media | 2 | 4 |
| HU-05 | **Compartir en redes.** Como visitante, quiero compartir un evento en redes sociales, para invitar a otras personas.<br><br>*Criterios de aceptación*<br>• Cada evento tiene una URL única con título e imagen para la vista previa. | Baja | 2 | 4 |
| HU-06 | **Agregar al calendario.** Como visitante, quiero agregar un evento a mi calendario, para no olvidarlo.<br><br>*Criterios de aceptación*<br>• Se genera un archivo `.ics` o un enlace a Google Calendar con título, fechas y dirección. | Media | 3 | 4 |

### Planificación por sprints

Supuestos: sprints de 2 semanas y capacidad aproximada de 20 puntos por sprint. Total: 82 puntos en 4 sprints. Son estimaciones iniciales que se ajustan con el equipo.

| Sprint | Historias (puntos) | Pts | Objetivo |
|---|---|---|---|
| 1 · Cuentas y publicación de eventos | HU-10 (3), HU-11 (3), HU-17 (3), HU-12 (5), HU-13 (8) | 22 | Un organizador puede registrarse, iniciar sesión y publicar un evento con imágenes. |
| 2 · Moderación y gestión | HU-18 (3), HU-19 (3), HU-20 (2), HU-14 (3), HU-15 (2), HU-16 (3), HU-21 (2) | 18 | El administrador modera, y el organizador edita, elimina y sigue el estado de sus eventos. |
| 3 · Descubrimiento público | HU-01 (3), HU-02 (5), HU-03 (5), HU-04 (5), HU-23 (3) | 21 | El público lista, filtra, busca y ve eventos con caché/CDN y SEO. Primer lanzamiento utilizable. |
| 4 · Usuarios y extras | HU-07 (5), HU-08 (3), HU-09 (2), HU-05 (2), HU-06 (3), HU-22 (3), HU-24 (3) | 21 | Favoritos, compartir, calendario, eventos similares y recuperación de contraseña. |

**Dependencias clave**

- HU-12 y HU-13 requieren que las categorías y ciudades ya estén cargadas (sin ellas no se puede crear un evento).
- Ningún evento es visible al público hasta que existe la moderación (Sprint 2).
- El Sprint 3 es el hito mínimo para publicar el portal.

### Trazabilidad: requisito funcional → historias de usuario

Sirve para comprobar que cada requisito tiene historias que lo cubren y que ninguna historia queda "huérfana".

| Requisito | Historias de usuario |
|---|---|
| RF-01 | HU-01, HU-02 |
| RF-02 | HU-03 |
| RF-03 | HU-10, HU-11, HU-24 |
| RF-04 | HU-12, HU-14, HU-15 |
| RF-05 | HU-13 |
| RF-06 | HU-08, HU-09 |
| RF-07 | HU-04 |
| RF-08 | HU-05, HU-06 |
| RF-09 | HU-16, HU-17, HU-18, HU-19, HU-20, HU-21 |
| RF-10 | HU-07, HU-24 |
| RF-11 | HU-22 |
| RF-12 | HU-23 |

### Decisiones adoptadas y cambios

Supuestos tomados donde faltaba una decisión. Se pueden cambiar; si cambian, se ajustan las historias indicadas.

| # | Tema | Decisión |
|:---:|---|---|
| 1 | Cuentas del público | Se mantiene la entidad USUARIO. Nuevo RF-10 (registro e inicio de sesión de usuarios) y RNF-06 aplica también a ellos. Si se elimina, cambian HU-07 a HU-09. |
| 2 | Recomendaciones | Se definen como "eventos similares" (misma categoría y ciudad) en el detalle. Nuevo RF-11 (HU-22). Se calcula por consulta y se cachea; no requiere entidad nueva. |
| 3 | Contador de visitas | Nuevo RF-12 (HU-23). Se agrega `contador_visitas` (entero, por defecto 0) a EVENTO. |
| 4 | Estado de moderación | `estado_moderacion` solo admite pendiente, aprobado o rechazado. |
| 5 | Imagen principal | Índice único parcial en IMAGEN sobre `es_principal` por evento. |
| 6 | Precio gratuito | Se representa con `precio = 0`. |
| 7 | Ubicación | Solo dirección; el mapa se resuelve con un enlace a Maps, sin latitud ni longitud. |
| 8 | Búsqueda | Índice de texto completo sobre título y descripción. |
| 9 | Administradores | Se crean manualmente, no hay registro público de administradores. |
| 10 | Recuperación de contraseña | Con un servicio de autenticación gestionado no necesita tablas nuevas; si es propia, requiere una tabla de tokens. |

---

## 4. Requerimientos no funcionales y Definition of Done

### Requisitos no funcionales

| ID | Atributo | Requisito |
|---|---|---|
| RNF-01 | Rendimiento | Listado y detalle < 200 ms; publicar con imagen en 2 a 3 s. Se logra con caché y CDN. |
| RNF-02 | Escalabilidad | Absorber picos de tráfico (por ejemplo, un festival) y duplicar el tráfico cada ~3 meses sin reescribir el sistema. |
| RNF-03 | Durabilidad | Respaldo de la base de datos y de las imágenes. Los eventos publicados y las cuentas no se pierden. |
| RNF-04 | Consistencia | Consistencia eventual aceptable en contadores (visitas, "me interesa") y recomendaciones. |
| RNF-05 | Disponibilidad | ≥ 99 %. Si cae el backend, las páginas cacheadas siguen visibles. |
| RNF-06 | Seguridad | Autenticación de organizadores, usuarios y administradores; validación de archivos subidos y protección contra spam. |
| RNF-07 | SEO y accesibilidad | Páginas indexables y HTML semántico, porque el descubrimiento viene de buscadores. |
| RNF-08 | Costo | Casi cero en la fase inicial, con pago por uso. |

### Definition of Done (DoD)

Una historia de usuario se considera **terminada** solo si cumple todos los puntos siguientes.

**Código y calidad**
- [ ] Cumple todos sus criterios de aceptación.
- [ ] El código fue revisado y aprobado mediante *pull request* por al menos otro integrante.
- [ ] Pasa el linter y las pruebas automatizadas, y no rompe funcionalidades existentes.
- [ ] Tiene pruebas para su lógica principal y para los casos de error más importantes.

**Rendimiento y seguridad**
- [ ] Listado y detalle responden en menos de 200 ms con caché (RNF-01).
- [ ] Valida los datos de entrada y los archivos subidos; no expone contraseñas ni datos de otras cuentas (RNF-06).
- [ ] Respeta los permisos por rol (público, organizador, administrador).

**Experiencia de usuario**
- [ ] Se ve y funciona bien en celular y en escritorio.
- [ ] Usa HTML semántico y cumple lo básico de accesibilidad: textos alternativos, contraste y navegación con teclado (RNF-07).
- [ ] Muestra mensajes claros ante errores y estados vacíos.

**Entrega**
- [ ] Está integrada en la rama principal y desplegada en el entorno de pruebas.
- [ ] La base de datos cambia solo mediante migraciones versionadas.
- [ ] La documentación (README, modelo de datos, variables de entorno) está actualizada.
- [ ] Fue demostrada al equipo y aceptada como completa.

---

## 5. Modelado de la base de datos

Diagrama Entidad-Relación 

<img width="1422" height="921" alt="modelo_ER_AgentArt" src="https://github.com/user-attachments/assets/f77cfe0a-de82-4e62-a7e5-2ab44196f6f0" />


### Reglas del modelo

- Un **organizador** publica muchos eventos; cada evento pertenece a una **categoría** y a una **ciudad**.
- Un evento tiene varias **imágenes**, y solo una puede ser la principal.
- El **correo** es único en organizadores, usuarios y administradores; el nombre de la **categoría** también es único.
- Cada decisión de **moderación** relaciona un evento con el administrador que la tomó. Un evento puede tener varias decisiones (por ejemplo, un rechazo y luego una aprobación).
- **Favorito** es la tabla intermedia entre usuarios y eventos; su clave primaria compuesta evita marcar dos veces el mismo evento.
- La eliminación de eventos es lógica (`eliminado`), para no perder información.
- `contador_interesados` y `contador_visitas` admiten consistencia eventual.
