---
name: development-validation
description: 'Use when adding or modifying application code, scripts, schemas, infrastructure, or workflow configuration. Require focused validation of each development change.'
user-invocable: false
disable-model-invocation: false
---

# Regla de desarrollo

## Alcance de aplicación

Aplica a cambios de implementación en `agents/`, `data/`, `infra/`, `internal/`, `mcps/`, `packages/`, `scripts/`, `services/`, `shared/`, `uis/` y `workflows/`, incluidos código, esquemas y configuración. No es necesaria para cambios exclusivamente documentales.

El agente debe cargar esta regla antes de modificar archivos dentro de ese alcance. La descripción permite su carga automática por relevancia y `AGENTS.md` raíz la exige explícitamente.

## Regla

Para cada cambio de desarrollo, ejecuta la validación más específica disponible que cubra el comportamiento afectado: prueba focalizada, lint, comprobación de tipos o compilación acotada. Si no existe una validación ejecutable o no se puede ejecutar, indícalo claramente y no presentes el cambio como verificado.

## Aplicación al dominio TrackFlow

Para cambios relacionados con almacén o inventario:

- Trata Los Ángeles y Zaragoza como ubicaciones separadas con SGA distintos. El briefing no mapea el software comercial ni la hoja de cálculo a una ubicación; no codifiques ese supuesto.
- No presupongas inventario global en tiempo real, interfaces/API existentes, formatos de integración ni datos actuales. Verifica qué fuentes están realmente implementadas.
- Distingue el saldo registrado y su fecha de actualización del conteo físico y su fecha. Prueba los estados de ambos almacenes cuando la implementación afecte a ambos; si faltan sistemas o datos para hacerlo, documenta esa limitación.
- Mantén las operaciones de lectura separadas de cualquier ajuste de stock. No añadas escrituras, umbrales, conversiones o procedimientos locales sin requisitos y autorización documentados.

En cualquier otro dominio de TrackFlow, basa los requisitos en `CONTEXT.md` y clasifica explícitamente los supuestos que el briefing no resuelva.