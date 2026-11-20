# Instrucciones para asistentes de IA

Este repositorio es **Builder Passport**, la plataforma de registro, equipos, proyectos y jueces del Builder Challenge de ZEEK.

## Antes de escribir código

1. Lee completo [`docs/PLAN.md`](docs/PLAN.md). Es la fuente de verdad del proyecto.
2. Lee el issue de GitHub de la tarea en la que estás trabajando.
3. Revisa quién es el responsable de la parte que vas a tocar (tabla de abajo).
4. Lee las 10 entradas más recientes de [`docs/avances/`](docs/avances/) para saber qué hicieron los demás.

## Equipo

| Persona | GitHub | Rol | Responsable de |
| --- | --- | --- | --- |
| Esteban | `Psybre` | PM | Plan, tablero, decisiones de alcance |
| Diego Díaz | `ddm140207` | Líder técnico | Setup, base de datos, autenticación y roles, perfiles, equipos, CI y despliegue |
| Derling Torres | `Derling07` | Frontend y diseño | Sistema visual y pantallas de participante |
| José Julián Gutiérrez | pendiente | Full stack | Proyectos, galería, jueces y calificación |
| Katheryn Méndez | `KaMendez04` | Backend | Panel de admin, invitaciones, exportaciones |
| Jeshua Venegas | `Jeshua130` | QA | Pruebas, reportes de bugs, prueba de carga |

El detalle de qué tablas y rutas son de cada persona está en "Qué es de cada quien" dentro de `docs/PLAN.md`. Si tu tarea toca la parte de otra persona, dilo para que se le pida revisión en el PR.

## Reglas

- Usa exactamente los nombres de tablas, campos, estados, roles y rutas de `docs/PLAN.md`.
- No agregues funcionalidades de la sección "Fuera de alcance" ni cambies el stack.
- Valida toda regla de negocio en el servidor, nunca solo en la interfaz.
- Textos de la interfaz en español de Costa Rica, con voseo.
- Nunca escribas claves, contraseñas ni URLs de producción en el código: van en variables de entorno.
- Si la tarea contradice el plan o le falta información, detente y avisa en vez de suponer.
- Antes de dar una tarea por terminada, revisa la "Definición de terminado" del plan.

## Registro de avances

Al terminar una tarea, en el **mismo PR** del código, crea un archivo nuevo en `docs/avances/` para que el equipo y sus IA sepan qué se hizo.

- **Nombre:** `AAAA-MM-DD-usuario-tema.md`, por ejemplo `2026-11-22-Derling07-pantalla-perfil.md`.
- **Formato:** la plantilla de [`docs/avances/README.md`](docs/avances/README.md). Máximo 25 líneas.
- **Nunca** edites ni borres la entrada de otra persona. Si algo cambió, crea una entrada nueva que lo explique.
- **Nunca** pongas claves, contraseñas ni datos personales de participantes.

## Comandos

- `pnpm install` · instalar dependencias
- `pnpm dev` · levantar en local
- `pnpm lint` · `pnpm typecheck` · `pnpm test` · revisar antes de abrir un PR
- `pnpm db:generate` · `pnpm db:migrate` · migraciones de base de datos
