# Instrucciones para agentes

## Lectura al inicio de cada sesión

Antes de analizar o modificar el proyecto, lee estos archivos en este orden:

1. `memory-bank/projectbrief.md` para entender el negocio, el problema y los objetivos.
2. `memory-bank/techContext.md` para conocer el stack, las decisiones técnicas y las restricciones.
3. `memory-bank/progress.md` para conocer el estado actual y los próximos pasos.

Si alguno no existe o está desactualizado, indícalo y no inventes la información que falta. Lee también el `README` de la carpeta en la que vayas a trabajar.

## Regla de desarrollo

Antes de modificar código, scripts, esquemas o configuración en `agents/`, `data/`, `infra/`, `internal/`, `mcps/`, `packages/`, `scripts/`, `services/`, `shared/`, `uis/` o `workflows/`, carga y sigue `.agents/skills/development-validation/SKILL.md`. Esta regla se aplica a todo cambio de implementación dentro de esas carpetas.

## Reglas del dominio TrackFlow

- Usa `CONTEXT.md` como fuente de verdad del negocio. Distingue los hechos del briefing de las propuestas y supuestos documentados en el banco de memoria o en una skill.
- TrackFlow opera almacenes en Los Ángeles y Zaragoza con SGA distintos: uno es software comercial y otro una hoja de cálculo avanzada. No asumas cuál corresponde a cada almacén; el briefing no lo especifica.
- No presentes inventario como global, sincronizado o en tiempo real: el briefing indica que no existe esa visibilidad compartida.
- En análisis de inventario, separa cantidad registrada por el sistema de conteo físico. No inventes causas, tolerancias, conversiones, políticas locales ni procedimientos de ajuste; solicita datos faltantes o deriva al personal responsable.
- No modifiques existencias ni ejecutes acciones sobre sistemas operativos. Las skills de este repo solo pueden describir acciones de lectura, salvo que una futura integración y autorización explícita definan otro alcance.

## Ubicación de aplicaciones

- El sitio web público de TrackFlow va en `uis/website`; su ruta de inicio es `/` dentro de esa aplicación y su marca y contenido deben basarse en `CONTEXT.md`.
- La aplicación interna va en `uis/backoffice`, separada del sitio público. Sus capacidades internas se organizan dentro de ese proyecto; `/` es su ruta de inicio, no una carpeta del monorepo.
- Las APIs y workers backend van en `services/`. Sigue el README de cada carpeta y documenta cada aplicación o servicio en su propia subcarpeta.
- No crees un segundo árbol de frontend/API en la raíz ni aplicaciones paralelas en rutas inventadas. Si el trabajo solo pide documentar la arquitectura, no crees las aplicaciones: deja constancia de dónde vivirían cuando se implementen.

## Flujo obligatorio antes de cada commit

Completa estos pasos en orden antes de ejecutar cada commit:

1. Revisa `git status` y distingue tus cambios de cualquier cambio preexistente. No descartes ni sobrescribas cambios ajenos.
2. Revisa el diff de los archivos afectados y confirma que el alcance es intencional; comprueba que no incluye secretos ni archivos protegidos.
3. Ejecuta las pruebas, el lint o la validación más específicos disponibles para el cambio. Si no existen o no se pueden ejecutar, deja constancia.
4. Actualiza `memory-bank/progress.md` si el estado o los próximos pasos han cambiado.
5. Prepara solo los archivos previstos, revisa el diff staged y confirma que no haya cambios ajenos o protegidos. Solo entonces realiza el commit.

No hagas commits salvo que el desarrollador lo solicite.

## Rutas protegidas

No modifiques las siguientes rutas sin confirmación explícita del desarrollador para la ruta concreta:

- `.git/`: metadatos internos del repositorio.
- `.devcontainer/`: configuración del entorno de desarrollo.
- `CONTEXT.md` y `company-choice.md`: contexto y elección de empresa.
- `data/raw/` y `data/eval/`: datos fuente y conjuntos de evaluación.
- Archivos que contengan credenciales, claves privadas, certificados o secretos, incluidos archivos de entorno locales.

La confirmación debe ser directa; una petición general de implementar una funcionalidad no autoriza cambios en estas rutas. Si un cambio solicitado parece requerirlos, explica por qué y pide autorización antes de editarlos.