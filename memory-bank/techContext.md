# Technical Context

## Estado del stack

El repositorio es una plantilla y todavía no contiene una aplicación ejecutable ni un stack de aplicación seleccionado para el proyecto.

- `packages/shared/` contiene un paquete TypeScript de tipos compartidos con tipos de ejemplo.
- `agents/_template/` incluye una plantilla de agente en Python.
- La guía del repositorio recomienda una API centralizada con FastAPI en `services/`, pero es una recomendación de la plantilla, no una decisión de implementación confirmada.
- No hay un runner de workspace en la raíz.

## Arquitectura y decisiones conocidas

- El proyecto se organiza como monorepo por responsabilidades: interfaces en `uis/`, APIs y procesos en `services/`, datos en `data/`, agentes en `agents/` y documentación transversal en `docs/`.
- Cuando se implemente producto, el sitio público de TrackFlow pertenece en `uis/website`; la aplicación interna pertenece en `uis/backoffice`; las APIs y workers pertenecen en `services/`.
- `uis/website` y `uis/backoffice` son proyectos frontend distintos: `/` es la ruta de entrada de cada aplicación, no una carpeta nueva del monorepo. La web pública debe usar marca y contenido basados en el briefing de TrackFlow. El backoffice concentra las capacidades internas en un proyecto, siguiendo el README de `uis/`.
- `services/` agrupa servicios backend. La guía recomienda empezar con una API centralizada FastAPI y extraer workers solo cuando sea necesario; es una recomendación, no una decisión tecnológica confirmada.
- No crear un segundo árbol de frontend/API en la raíz ni duplicar estas aplicaciones en otras rutas. Antes de implementar, seguir el README de la carpeta correspondiente y documentar cada app o servicio en su subcarpeta.
- Este hito solo documenta la ubicación prevista; no pide crear las aplicaciones ni carpetas de producto.
- La empresa asignada es TrackFlow; el contexto de negocio está en `CONTEXT.md`.
- No se han documentado todavía decisiones de arquitectura de la solución ni integraciones concretas.

## Restricciones y consideraciones técnicas

- La solución debe contemplar los almacenes de Los Ángeles y Zaragoza, que usan SGA distintos: uno es software comercial y el otro una hoja de cálculo avanzada. El briefing no indica qué sistema corresponde a cada almacén; no se debe inferir ese mapeo.
- No existe visibilidad global del inventario en tiempo real. No presentar datos como actuales o sincronizados salvo que una fuente y su momento de actualización lo demuestren.
- El contexto describe un ERP antiguo, integraciones heredadas poco documentadas y datos operativos dispersos; la integración con esos sistemas puede limitar el alcance inicial.
- La skill `inventory-discrepancy-review` es una guía de análisis de solo lectura. No implica que haya API de inventario disponible ni que el agente pueda cambiar existencias, crear incidencias o enviar alertas.
- Las situaciones adicionales de la skill (por ejemplo, duplicados, fechas desconocidas o cantidades anómalas) son salvaguardas propuestas para datos ambiguos, no incidencias confirmadas por el briefing.
- No están definidos umbrales de tolerancia, conversiones de unidades, reglas locales de ajuste ni políticas de escalado. No inventarlos; pedir la fuente o derivar la decisión al personal responsable.
- El asistente propuesto necesitará fuentes autorizadas y actualizadas. Las acciones sobre sistemas no están definidas y deberían concretarse antes de implementarse.
- El soporte en español e inglés es recomendable para una solución operativa en ambos países, pero no es un requisito confirmado del alcance inicial.
- Tecnologías, proveedores, despliegue, almacenamiento, autenticación y políticas de acceso están pendientes de decisión.
