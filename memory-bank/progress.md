# Progress

## Estado actual

- Repositorio base de la plantilla 4Geeks Academy; el contexto ya está personalizado para TrackFlow.
- La elección de empresa prioriza Operaciones de almacén y Tecnología.
- La idea inicial es una plataforma interna y un asistente que consulte procedimientos; no hay funcionalidades implementadas todavía.
- Se añadió y revisó `inventory-discrepancy-review`, una skill de solo lectura para resumir discrepancias de inventario para revisión humana. Sus casos defensivos se identifican como salvaguardas, no como hechos confirmados del negocio.
- Se alinearon las reglas y el banco de memoria con las limitaciones conocidas: dos SGA distintos, sin mapeo documentado por almacén ni visibilidad global en tiempo real.
- Se documentó la ubicación prevista del producto: web pública en `uis/website`, backoffice interno en `uis/backoffice` y APIs/workers en `services/`. No se crearon aplicaciones; el stack sigue pendiente.
- Se está trabajando en la rama `feature/agent-memory-bank`.
- Esta carpeta documenta el contexto inicial del proyecto; aún no hay stack definitivo ni decisiones de arquitectura registradas.

## Próximos pasos previstos

1. Acordar un primer caso de uso y definir qué queda dentro y fuera del alcance.
2. Identificar las fuentes de procedimientos y datos disponibles, junto con permisos y restricciones de acceso.
3. Elegir el stack y documentar las decisiones de arquitectura necesarias para ese caso de uso.
4. Implementar una primera versión acotada y definir cómo se validará la calidad de sus respuestas o resultados.
