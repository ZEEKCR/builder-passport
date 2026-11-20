# Plan técnico · Builder Passport

Última actualización: 19 de noviembre de 2026 · Responsable: Esteban (`Psybre`)

## Resumen

Builder Passport tiene que estar en producción el **1 de diciembre de 2026**, con registro, perfiles, equipos, proyectos y jueces funcionando para el Builder Challenge del Innovation & Impact Day.

- **Qué es:** la plataforma donde los participantes crean su perfil, forman equipos de 3 a 6 personas y entregan sus proyectos. Los jueces los revisan y califican, y ZEEK exporta toda la información para patrocinadores.
- **Fechas:** el challenge corre del 2 al 11 de diciembre de 2026. Las entregas cierran el 11 de diciembre a las 23:59, hora de Costa Rica. El Demo Day es el 12 de diciembre.
- **Escala esperada:** entre 100 y 150 participantes, unos 30 a 40 equipos, entre 5 y 10 jueces.
- **A futuro:** es la base del Builder Passport de ZEEK, una plataforma permanente para varios eventos, con insignias por participación. Por eso el modelo de datos se diseña por evento desde el primer día.
- **Repositorio:** `ZEEKCR/builder-passport`. Tareas en el Project "Builder Passport" de la organización ZEEKCR.

## Cómo usar este plan

Este documento es la **fuente de verdad** del proyecto, para personas y para asistentes de IA.

- Vive en el repo como `docs/PLAN.md`. Cualquier cambio al plan se hace por pull request y lo aprueba Esteban o Diego.
- Si el código y este plan no coinciden, manda el plan. Si el plan está mal, se corrige el plan primero.
- Si algo no está definido aquí, **no se inventa**: se pregunta en el grupo y se agrega al plan.

### Reglas para asistentes de IA

Si eres un asistente de IA trabajando en este repositorio:

1. Lee este plan completo antes de proponer o escribir código.
2. Usa exactamente los nombres de tablas, campos, estados, roles y rutas definidos aquí.
3. No agregues funcionalidades que estén en "Fuera de alcance". No cambies el stack.
4. Toda regla de negocio se valida en el servidor, nunca solo en la interfaz.
5. Los textos de la interfaz van en español de Costa Rica, con voseo ("Creá tu perfil", "Unite a un equipo").
6. Si una tarea contradice este plan o le falta información, detente y dilo en vez de suponer.
7. Al terminar una tarea, revisa la "Definición de terminado".

### Glosario

| Término | Significado |
| --- | --- |
| Evento | Una edición de un challenge o hackathon. Hoy existe uno: Builder Challenge IID 2026 |
| Participante | Persona registrada que puede unirse a un equipo |
| Equipo | Grupo de 3 a 6 participantes de un mismo evento |
| Lead | El participante que creó el equipo, o a quien pasó ese rol |
| Proyecto | Lo que entrega un equipo. Uno por equipo por evento |
| Borrador (`draft`) | Proyecto que todavía no se envió |
| Enviado (`submitted`) | Proyecto que cumple los requisitos y fue enviado |
| Cierre | Fecha y hora después de la cual nada se puede editar |
| Juez | Persona invitada por un admin para revisar y calificar proyectos |
| Admin | Organizador de ZEEK con acceso total |

## Alcance

Todo lo marcado como obligatorio tiene que estar en producción el 1 de diciembre.

| Funcionalidad | Prioridad | Fecha límite |
| --- | --- | --- |
| Registro, login y recuperación de contraseña | Obligatorio | 1 dic |
| Perfil (ver y editar) y perfil público | Obligatorio | 1 dic |
| Consentimiento para compartir datos con patrocinadores | Obligatorio | 1 dic |
| Equipos: crear, unirse con código, salir, lista de "busco equipo" | Obligatorio | 1 dic |
| Proyectos: crear y editar borrador | Obligatorio | 1 dic |
| Proyectos: enviar y cierre automático | Obligatorio | 1 dic |
| Galería de proyectos | Obligatorio | 1 dic |
| Invitación de jueces y vista de jueces | Obligatorio | 1 dic |
| Calificación de jueces por criterios | Obligatorio | 1 dic |
| Panel de admin y exportación de participantes y equipos | Obligatorio | 1 dic |
| Exportación de proyectos y notas de jueces | Comodín | 11 dic |
| Modo oscuro | Opcional | Sin fecha |

**Comodín** significa que se intenta tener el 1 de diciembre, pero si no llega se termina antes del 11 de diciembre sin afectar el lanzamiento.

**Fuera de alcance:** insignias, historial del pasaporte entre eventos, contador de commits, Build Receipt, landing page del evento, transmisión en vivo dentro de la plataforma (solo se muestra el link), chat o mensajes entre usuarios, notificaciones push, login con Google u otras redes, subida de videos.

## Stack técnico

| Capa | Herramienta |
| --- | --- |
| Framework | Next.js 16 (App Router) + React 19 + TypeScript |
| Estilos | Tailwind CSS 4 |
| Base de datos | Postgres en Neon (producción) y PGlite en local |
| ORM y migraciones | Drizzle ORM + drizzle-kit |
| Autenticación | Better Auth, con correo y contraseña |
| Correo | Resend (plan gratis), cuando el dominio esté conectado |
| Archivos | Vercel Blob, solo para fotos de perfil |
| Hosting | Vercel |
| Errores | Sentry (plan gratis) |
| Pruebas | Vitest (unitarias) y Playwright (de punta a punta) |
| Calidad | ESLint, Prettier, Husky, commitlint, gitleaks |
| Paquetes | pnpm, Node 22 |

- **Variables de entorno:** `DATABASE_URL`, `BETTER_AUTH_SECRET`, `APP_URL`, `RESEND_API_KEY`, `MAIL_FROM`, `BLOB_READ_WRITE_TOKEN`, `SENTRY_DSN`, `SEED_ADMIN_EMAIL`. Se documentan en `.env.example`. Nunca se suben valores reales.
- **URL de la app:** siempre desde `APP_URL`, nunca escrita en el código. Hasta que el dominio esté conectado se usa la URL gratis de Vercel.
- **Horas:** todo se guarda en UTC (`timestamptz`) y se muestra en hora de Costa Rica (`America/Costa_Rica`).

## Modelo de datos

El perfil es global y todo lo demás cuelga de un evento. La próxima hackathon será otro registro en `events`, y las insignias se agregarán después con una tabla nueva.

| Tabla | Campos | Reglas |
| --- | --- | --- |
| `users` | id, email (único), name, role (`participant`, `judge`, `admin`), created_at | Las tablas de sesión y cuentas las crea Better Auth |
| `profiles` | user_id, username (único), photo_url, bio (máx. 280), city, organization, skills (lista), github_url, linkedin_url, portfolio_url, looking_for_team (booleano) | 1 a 1 con `users`. Se crea al registrarse |
| `consents` | id, user_id, type (`share_with_sponsors`), accepted (booleano), accepted_at | Se guarda en el registro. Se puede cambiar desde el perfil |
| `events` | id, name, slug, starts_at, submissions_close_at, status (`upcoming`, `open`, `closed`), live_url | Hoy uno solo. Cierre: `2026-12-12T05:59:59Z` (11 dic 23:59 en Costa Rica) |
| `teams` | id, event_id, name, description, invite_code (único, 8 caracteres), created_by, created_at, archived_at | Pertenece a un evento |
| `team_members` | team_id, user_id, role (`lead`, `member`), joined_at | Único por (user_id, evento): una persona, un equipo por evento |
| `projects` | id, team_id (único por evento), event_id, name, problem, solution, repo_url, demo_url, video_url, slides_url, status (`draft`, `submitted`), submitted_at, updated_at | Un proyecto por equipo |
| `invitations` | id, email, role (`judge`, `admin`), token (único), expires_at, accepted_at, created_by | Para jueces y admins. Vence a los 7 días |
| `judge_assignments` | event_id, user_id | Qué jueces califican qué evento |
| `criteria` | id, event_id, name, description, order | Criterios de calificación del evento. Se cargan con un script |
| `scores` | id, project_id, judge_id, criterion_id, value (1 a 5), comment, updated_at | Único por (project_id, judge_id, criterion_id) |

- **Fotos de perfil:** Vercel Blob, máximo 2 MB, solo JPG, PNG o WebP. Si falta el token, el perfil funciona sin foto.
- **Videos:** nunca se suben. Solo se guarda el link (YouTube, Loom o Drive).

## Autenticación y acceso por rol

Nadie comparte contraseñas. Cada persona tiene su propia cuenta y el rol vive en `users.role`. No existe una "contraseña de juez".

| Rol | Cómo obtiene la cuenta | Qué puede hacer |
| --- | --- | --- |
| Participante | Se registra solo en `/registro` con nombre, correo, contraseña y la casilla de consentimiento | Editar su perfil, crear o unirse a un equipo, editar y enviar el proyecto de su equipo, ver la galería |
| Juez | Un admin lo invita por correo desde `/admin/jueces`. El juez abre el link `/invitacion/[token]` y crea su contraseña | Ver proyectos enviados del evento asignado y calificarlos. No puede unirse a equipos |
| Admin | El primer admin se crea con el script `pnpm seed:admin`, usando `SEED_ADMIN_EMAIL`. Los demás los invita un admin, igual que a los jueces | Todo: ver y editar participantes, equipos y proyectos, mover personas entre equipos, invitar jueces, exportar |

- **Si el correo de un juez ya tiene cuenta de participante:** al aceptar la invitación, su rol cambia a `judge`. Si estaba en un equipo, primero un admin lo saca del equipo.
- **Contraseñas:** mínimo 8 caracteres. Límite de intentos de login con el rate limit que trae Better Auth.
- **Protección de rutas:** cada ruta revisa el rol en el servidor. Un participante que entra a `/jueces` o `/admin` ve un 404.
- **Mientras el dominio no esté conectado:** no se pueden enviar correos a cualquier dirección. Por eso, el panel de admin muestra un botón **"Copiar link"** en cada invitación y genera links de recuperación de contraseña, para mandarlos a mano por WhatsApp. Cuando el dominio esté conectado, los mismos links salen también por correo.

## Reglas de negocio

Todas estas reglas se validan en el servidor.

### Participación individual y equipos

1. Una persona se puede registrar sola, sin equipo. Para entregar un proyecto, tiene que estar en un equipo.
2. Quien no tiene equipo puede activar "busco equipo" en su perfil. Aparece en `/buscar-equipo` con sus habilidades.
3. Los admins ven esa lista y pueden asignar a cualquier persona a un equipo con cupo.
4. Cualquier participante sin equipo puede crear uno y queda como `lead`. Se genera un código de invitación de 8 caracteres.
5. Los demás se unen con ese código o con el link `/equipo/unirse/[código]`.
6. Una persona solo puede estar en un equipo por evento.
7. Un equipo tiene máximo 6 miembros. No se aceptan nuevos miembros si ya hay 6.
8. **No existen equipos de 2 para entregar:** un equipo de 1 o 2 personas puede existir solo mientras se arma. Para enviar el proyecto necesita entre 3 y 6.
9. Cualquier miembro puede salir. Si sale el lead, el rol pasa al miembro más antiguo. Si no queda nadie, el equipo se archiva (`archived_at`).
10. El lead puede regenerar el código de invitación y sacar miembros.
11. Los cambios de integrantes se permiten hasta el cierre. Después del cierre, nadie entra ni sale.
12. Si un equipo con proyecto enviado baja de 3 miembros, el proyecto vuelve a `draft` y el equipo ve un aviso.
13. Jueces y admins no pueden pertenecer a equipos.

### Proyectos

1. Cada equipo tiene un solo proyecto, que nace en `draft` cuando el equipo lo crea. Cualquier miembro lo puede editar.
2. Para enviarlo (`submitted`) se exige: nombre, problema, solución, `repo_url` válida y equipo de 3 a 6 personas.
3. Después de enviarlo se puede seguir editando y reenviando hasta el cierre. `submitted_at` guarda el último envío.
4. Después de `submissions_close_at`, ningún proyecto se puede editar, enviar ni crear. La comparación usa la hora del servidor.
5. Un proyecto que sigue en `draft` al cierre no participa y no aparece a los jueces.

### Galería

1. Antes del cierre, cada participante ve solo el proyecto de su equipo. Jueces y admins ven todos los enviados.
2. Después del cierre, todos los usuarios con sesión ven la galería de proyectos enviados.

### Jueces y calificación

1. Un juez solo ve proyectos `submitted` del evento que tiene asignado. Puede leer todo, nunca editar.
2. Califica cada proyecto en cada criterio del 1 al 5, con un comentario opcional. Puede cambiar sus notas hasta que un admin cierre la calificación.
3. Cada juez ve solo sus propias notas. Los participantes nunca ven notas.
4. El admin ve el promedio por proyecto y por criterio, y cuántos jueces calificaron cada proyecto.
5. Los criterios se cargan en la tabla `criteria` con un script. Los criterios oficiales son: Problema, Solución, Ejecución técnica, Impacto y Presentación (cómo el equipo comunica su proyecto: pitch, demo, video y slides).

## Pantallas

Todo se diseña primero para celular.

| Pantalla | Ruta | Quién la ve |
| --- | --- | --- |
| Inicio con info del challenge, fechas y link del live | `/` | Todos |
| Registro | `/registro` | Visitantes |
| Login | `/login` | Visitantes |
| Recuperar contraseña | `/recuperar` y `/restablecer/[token]` | Visitantes |
| Aceptar invitación de juez o admin | `/invitacion/[token]` | Invitados |
| Mi perfil | `/perfil` | Participante |
| Perfil público | `/u/[username]` | Usuarios con sesión |
| Mi equipo: crear, unirse, ver miembros, salir | `/equipo` | Participante |
| Unirse con link | `/equipo/unirse/[código]` | Participante |
| Personas que buscan equipo | `/buscar-equipo` | Participante y admin |
| Mi proyecto: editar y enviar | `/proyecto` | Miembros del equipo |
| Galería de proyectos | `/proyectos` | Según reglas de galería |
| Detalle de proyecto | `/proyectos/[id]` | Según reglas de galería |
| Lista de proyectos para calificar | `/jueces` | Juez |
| Calificar un proyecto | `/jueces/[projectId]` | Juez |
| Panel de admin | `/admin` | Admin |
| Admin: participantes, equipos, proyectos, jueces, exportar | `/admin/participantes`, `/admin/equipos`, `/admin/proyectos`, `/admin/jueces`, `/admin/exportar` | Admin |

## Exportación de datos

ZEEK tiene que poder descargar la información en un clic y en cualquier momento, porque los patrocinadores pueden querer seguir trabajando con los proyectos o financiarlos.

- **Dónde:** `/admin/exportar`, un botón por archivo.
- **Formato:** CSV en UTF-8 con BOM, para que Excel muestre bien las tildes. Nombre del archivo con fecha: `participantes-2026-12-11.csv`.
- **Participantes** (obligatorio, 1 dic): nombre, correo, username, ciudad, organización, habilidades, links, equipo, consentimiento.
- **Equipos** (obligatorio, 1 dic): nombre, integrantes con correo, cantidad, estado del proyecto.
- **Proyectos** (comodín): nombre, equipo, integrantes con correo, problema, solución, links, estado, fecha de envío, y columnas vacías para notas.
- **Notas de jueces** (comodín): proyecto, juez, criterio, nota, comentario, y promedio por proyecto.
- **Plan B de los jueces:** si la calificación en la plataforma falla, los jueces califican sobre el archivo de proyectos en Excel.
- **Para patrocinadores:** un filtro "solo con consentimiento" que excluye a quien no aceptó compartir sus datos.

## Diseño e identidad visual

El tema principal es claro (dirección "Problem First" del deck creativo). El modo oscuro queda preparado, pero solo se activa si sobra tiempo.

- **Colores como variables CSS** desde el primer día, para poder cambiar de tema sin tocar componentes.
- **Paleta ZEEK:** violeta `#6335ED`, verde `#01E576`, blanco `#F7F7F7`, negro `#161616`. Degradados suaves en azul eléctrico, cian, violeta y rosa para acentos.
- **Tipografías:** Geist para títulos y texto, IBM Plex Mono para etiquetas y datos.
- **Estilo:** minimalista y editorial, con mucho espacio. Evitar neón, ilustraciones complejas y apariencia de conferencia corporativa.
- **Accesibilidad:** contraste AA, todos los campos con etiqueta, navegable con teclado.

## Equipo y responsables

| Persona | GitHub | Rol | Responsable de | Disponibilidad |
| --- | --- | --- | --- | --- |
| Esteban | `Psybre` | PM | Prioridades, fechas, decisiones, contacto con ZEEK y Mindful Tech | Flexible |
| Diego Díaz | `ddm140207` | Líder técnico | Setup, base de datos, autenticación y roles, perfiles, equipos, revisión de PRs, despliegue | Flexible |
| Derling Torres | `Derling07` | Frontend y diseño | Sistema visual, todas las pantallas de participante, adaptación a celular | Flexible |
| José Julián Gutiérrez | `josejulian1604` | Full stack | Proyectos (borrador, envío, cierre), galería, jueces y calificación | Fines de semana |
| Katheryn Méndez | `KaMendez04` | Backend | Panel de admin, invitaciones, exportaciones | Fines de semana |
| Jeshua Venegas | `Jeshua130` | QA | Casos de prueba, reportes de bugs, prueba de carga, apoyo al seguimiento del tablero | Entre semana |

### Qué es de cada quien

Cada parte del código tiene un responsable. Si un cambio toca algo de otra persona, se le avisa y se le pide revisión en el PR.

| Persona | GitHub | Tablas | Rutas y partes del código |
| --- | --- | --- | --- |
| Diego Díaz | `ddm140207` | `users`, `profiles`, `consents`, `events`, `teams`, `team_members` | Configuración del proyecto, esquema y migraciones, Better Auth, protección de rutas, lógica de servidor de `/registro`, `/login`, `/recuperar`, `/perfil`, `/equipo`, `/buscar-equipo`, CI y despliegue |
| Derling Torres | `Derling07` | Ninguna | Sistema visual (colores, tipografías, componentes base) y la interfaz de `/`, `/registro`, `/login`, `/recuperar`, `/perfil`, `/u/[username]`, `/equipo`, `/buscar-equipo` |
| José Julián Gutiérrez | `josejulian1604` | `projects`, `criteria`, `scores`, `judge_assignments` | `/proyecto`, `/proyectos`, `/proyectos/[id]`, `/jueces`, `/jueces/[projectId]`, cierre automático de entregas, script de criterios |
| Katheryn Méndez | `KaMendez04` | `invitations` | `/admin` y todas sus subrutas, `/invitacion/[token]`, invitación de jueces y admins, exportaciones CSV, script `seed:admin` |
| Jeshua Venegas | `Jeshua130` | Ninguna | Pruebas de punta a punta con Playwright, casos de prueba, prueba de carga, reportes de bugs con la etiqueta `bug` |
| Esteban | `Psybre` | Ninguna | Este plan, el tablero y las decisiones de alcance. Revisa los PRs de Diego |

**Respaldo:** José Julián conoce la autenticación y la base de datos desde el primer fin de semana, por si Diego no puede seguir. Al menos dos personas saben desplegar.

## Calendario

Solo hay dos fines de semana antes del lanzamiento (21–22 y 28–29 de noviembre). Ahí se concentra el trabajo de Katheryn y José Julián.

| Milestone | Fecha | Qué tiene que estar |
| --- | --- | --- |
| Arranque | 20 nov | Reunión de arranque. Diego empieza el setup |
| Semana 1 | 22 nov | Setup desplegado en Vercel, esquema de base de datos, autenticación con roles, sistema visual base, perfiles |
| Semana 2 | 29 nov | Equipos, proyectos, galería, jueces y calificación, panel de admin con exportaciones. QA completo |
| Lanzamiento | 1 dic | Todo lo obligatorio en producción, bugs críticos corregidos, prueba de carga hecha, admins cargados |
| Comodín | 11 dic | Exportación de proyectos y notas de jueces, si no llegó antes. Jueces invitados entre el 8 y el 10 de diciembre |

**También antes del 1 de diciembre:** resolver el acceso al dominio y conectarlo a Vercel (a más tardar el 26 o 27 de noviembre), configurar Resend con ese dominio, y respaldar la base de datos antes del challenge y antes del cierre de entregas.

## Cómo trabajamos

La plataforma tiene que estar lista el 1 de diciembre, pero este proyecto también es una prueba: queremos ver cómo trabajamos juntos y si este método nos funciona para los próximos proyectos de ZEEK.

**Uso de IA:** está permitido y recomendado usar IA (Cursor, Claude, Copilot u otras) para avanzar más rápido. La única condición es entender lo que se sube: cada quien responde por su código en la revisión.

### Flujo de trabajo

1. Tomar un issue asignado y moverlo a **En progreso** en el tablero.
2. Crear la rama desde el issue (enlace "Create a branch") o con el formato `feature/12-perfil`, con el número del issue.
3. Hacer commits pequeños con formato convencional: `feat: ...`, `fix: ...`, `chore: ...`, `docs: ...`.
4. Agregar en el mismo PR una entrada en `docs/avances/` con el resumen de lo que se hizo (ver "Registro de avances").
5. Abrir un pull request hacia `main` con **`Closes #12`** en la descripción. Sin eso, el tablero no se actualiza solo.
6. Esperar 1 aprobación. Diego revisa la mayoría; los PRs de Diego los revisa Esteban.
7. Al unirse a `main`, Vercel despliega solo y la tarea pasa a **Hecho**.

### Registro de avances

La carpeta `docs/avances/` guarda un archivo corto por cada tarea terminada: qué se hizo, qué archivos toca, qué se decidió y qué necesitan saber los demás. Así cada persona y su IA saben cómo va el proyecto antes de empezar algo nuevo.

- Un archivo por tarea, en el mismo PR del código, con el nombre `AAAA-MM-DD-usuario-tema.md`.
- Se usa la plantilla de `docs/avances/README.md`. Máximo 25 líneas.
- Nadie edita ni borra la entrada de otra persona.
- Antes de empezar una tarea, se leen las 10 entradas más recientes.

### Definición de terminado

Una tarea está terminada cuando:

- Cumple lo que dice el issue y este plan.
- Las reglas de negocio se validan en el servidor.
- Funciona en celular y en computadora.
- Pasa lint, typecheck y tests en el CI.
- Tiene pruebas para las reglas de negocio que toca.
- Se revisó en el despliegue de prueba (preview) de Vercel.
- Tiene su entrada en `docs/avances/`.
- El PR fue aprobado y unido a `main`.

### Reglas

- Nadie sube directo a `main`: la regla del repo lo bloquea.
- Si te trabas más de medio día, mueve la tarea a **Bloqueado** y avisa en el grupo.
- Nunca subir claves ni contraseñas: van en variables de entorno.
- Check-in corto por WhatsApp 2 o 3 veces por semana: qué hice, qué sigue, qué me bloquea.

**Bot de seguimiento:** un asistente de IA revisa el repo cada día, le manda un resumen a Esteban y puede comentar en los PRs. Sus comentarios salen con la cuenta `Psybre` y empiezan con "🤖". Nunca aprueba PRs.

## Primeros pasos de Diego

El objetivo es que el sábado 21 los demás ya tengan una base sobre la cual trabajar.

- [ ] Crear el proyecto Next.js 16 con TypeScript, Tailwind 4 y pnpm, con ESLint, Prettier, Husky, commitlint y gitleaks
- [ ] Conectar Drizzle con PGlite en local y crear el esquema completo de "Modelo de datos"
- [ ] Configurar Better Auth con registro, login, roles y protección de rutas
- [ ] Crear el script `pnpm seed:admin` y el script de criterios de calificación
- [ ] Crear el proyecto en Vercel y la base en Neon, y desplegar la primera versión
- [ ] Configurar el CI de GitHub Actions (lint, typecheck, tests)
- [ ] Escribir en el README cómo levantar el proyecto en local
- [ ] Crear el layout base con variables de color, para que Derling arranque con el diseño

## Decisiones pendientes

- [ ] Resolver el acceso al dominio para poder conectarlo
- [ ] Lista de jueces con sus correos, para invitarlos entre el 8 y el 10 de diciembre (las invitaciones vencen a los 7 días)
- [ ] Confirmar que la galería se abre a todos solo después del cierre
