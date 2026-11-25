---
name: inventory-discrepancy-review
description: 'Use when TrackFlow warehouse staff need to review inventory overages, shortages, count mismatches, missing or invalid data, duplicate records, or time gaps between a system snapshot and a physical count. Classify the case and prepare a human-review summary.'
user-invocable: true
disable-model-invocation: false
---

# Revisión de discrepancias de inventario

## Objetivo

Revisar un registro de conteo físico frente a la cantidad registrada por el sistema para un SKU en un almacén de TrackFlow, calcular la diferencia y preparar un resumen para revisión humana. Esta skill no reconcilia ni modifica inventario.

## Relación con el briefing

El briefing confirma que Los Ángeles y Zaragoza usan SGA distintos (uno es software comercial y el otro una hoja de cálculo avanzada), que no existe visibilidad global del inventario en tiempo real y que las discrepancias son frecuentes y se detectan tarde. No especifica qué SGA corresponde a cada almacén ni define reglas de ajuste.

Los casos de duplicados, fechas ausentes, conteos repetidos o cantidades anómalas incluidos más abajo son salvaguardas para manejar inputs ambiguos; no representan problemas confirmados en TrackFlow. No afirmes que hayan ocurrido salvo que el usuario aporte evidencia.

## Inputs

El usuario proporciona un registro con estos campos:

| Campo | Requerido | Descripción |
| --- | --- | --- |
| `warehouse` | Sí | Almacén: Los Ángeles o Zaragoza. No lo infieras a partir de la fuente del sistema. |
| `sku` | Sí | Identificador del producto contado. |
| `system_quantity` | Sí si hay un único registro | Cantidad registrada para ese almacén. Si hay varios candidatos, no elijas uno: proporciona todos en `additional_system_records`. |
| `physical_count` | Sí si hay un único conteo | Cantidad observada en el conteo físico. Si hay conteos repetidos distintos, no elijas uno: proporciona todos en `repeated_physical_counts`. |
| `unit` | Sí | Unidad usada en ambas cantidades; debe ser la misma. |
| `counted_at` | Sí | Fecha y hora del conteo, preferiblemente ISO 8601. |
| `system_source` | Sí para un registro único | Sistema o registro del que se obtuvo la cantidad; en `additional_system_records`, indica la fuente de cada candidato. No asumas cuál SGA corresponde al almacén. |
| `system_as_of` | No | Fecha y hora a la que corresponde la cantidad del sistema, si está disponible. |
| `notes` | No | Contexto adicional aportado por el usuario, como lote o incidencia observada. |
| `additional_system_records` | No | Otros registros candidatos para el mismo almacén y SKU, con identificador, cantidad, unidad y fecha si se conocen. |
| `repeated_physical_counts` | No | Conteos repetidos, cada uno con cantidad, unidad y fecha si se conocen. |
| `movement_context` | No | Recepciones, picking, envíos, devoluciones o traslados informados por el usuario, con fechas y cantidades si están disponibles. |

Las cantidades deben ser números finitos. `physical_count` no puede ser negativo. Un `system_quantity` negativo es una anomalía no documentada en el briefing: clasifícala como `review_required`, informa la diferencia solo como provisional y solicita revisión; no lo trates como saldo válido ni como hecho esperado del negocio. No conviertas unidades ni completes valores ausentes por inferencia. Las marcas de tiempo deben conservarse tal como fueron proporcionadas: no infieras la zona horaria. La ausencia de una fecha de actualización impide afirmar que el saldo del sistema es actual.

## Procedimiento

1. Valida `warehouse`, `sku`, unidades, las cantidades presentes y que `counted_at` sea interpretable. Se requiere un valor único de sistema/conteo o una lista explícita de candidatos para el campo ambiguo. Si falta esa información, hay una cantidad no numérica/no finita, el conteo físico es negativo, `counted_at` no es interpretable o las unidades no coinciden, devuelve `insufficient_data`, enumera el problema y solicita aclaración.
2. Busca ambigüedades: almacén o SKU dudoso, más de un registro candidato del sistema, conteos físicos repetidos que no coinciden, o movimientos informados entre `system_as_of` y `counted_at`. No elijas registros, promedies conteos ni compenses movimientos por tu cuenta.
3. Si hay varios conteos repetidos idénticos, informa que coinciden y usa ese valor común. Si difieren, no escojas ni promedies uno. Si solo un lado (sistema o conteo) tiene varios candidatos, calcula deltas por candidato y márcalos como provisionales. Si ambos lados tienen varios valores sin emparejamiento explícito, no calcules deltas combinados.
4. Calcula `difference = physical_count - system_quantity` solo para un par único confirmado y con unidades coincidentes. Un resultado positivo es sobrante físico frente al sistema; uno negativo es faltante físico; cero significa que las cantidades coinciden. Los candidatos múltiples nunca producen una diferencia única.
5. Determina el estado:
	- `insufficient_data`: falta o no es válido un dato obligatorio, o no se pueden comparar las unidades.
	- `review_required`: se puede hacer un cálculo provisional, pero hay un registro ambiguo/duplicado, conteos repetidos distintos, un saldo del sistema negativo, falta o invalidez de `system_as_of`, o un desfase temporal/movimiento que impide confirmar que los datos representan el mismo momento. Conserva los deltas provisionales por separado si pueden calcularse.
	- `no_discrepancy`: diferencia cero y no hay ambigüedades que impidan comparar los datos.
	- `discrepancy_found`: diferencia distinta de cero y no hay ambigüedades que impidan comparar los datos.
	- Asigna `case_type` de forma determinista: `balanced` para `no_discrepancy`; `physical_overage` o `physical_shortage` según el signo de una diferencia comparable; `incomplete_or_invalid` para `insufficient_data`; `anomalous_system_balance` para un saldo de sistema negativo; y `ambiguous_or_not_comparable` para las demás revisiones requeridas.
6. Si `system_as_of` falta, es inválido, no tiene zona horaria suficiente para compararlo o representa un momento distinto a `counted_at`, indícalo y usa `review_required`. Compara fechas con zona horaria solo cuando esta se haya proporcionado; no la infieras. No declares que un dato está “obsoleto” según un umbral inventado.
7. Resume hechos separados de hipótesis. Puedes mencionar como causas posibles una recepción, devolución, picking, envío, daño, pérdida o registro tardío solo como hipótesis; no afirmes ninguna causa sin evidencia incluida en los inputs.
8. Indica el siguiente paso seguro: pedir los datos faltantes, confirmar el registro correcto, repetir/validar el conteo o derivar al responsable del almacén según el procedimiento local. No inventes prioridades, SLA ni políticas.

## Formato de salida

Devuelve un resumen con estos campos:

```text
status: no_discrepancy | discrepancy_found | insufficient_data | review_required
case_type: balanced | physical_overage | physical_shortage | incomplete_or_invalid | anomalous_system_balance | ambiguous_or_not_comparable
warehouse: <valor proporcionado o desconocido>
sku: <valor proporcionado o desconocido>
system_quantity: <valor y unidad, si es válido>
physical_count: <valor y unidad, si es válido>
difference: <delta único confirmado; no calculada si hay varios candidatos>
system_source: <valor proporcionado o desconocido>
counted_at: <valor proporcionado o desconocido>
system_as_of: <valor proporcionado o no proporcionado>
issues: <datos faltantes, inválidos, duplicados, fechas o unidades no comparables>
candidate_system_records: <valores/fuentes y deltas provisionales por separado si el conteo es único; no sumar ni elegir>
repeated_physical_counts: <valores y deltas provisionales por separado si el registro de sistema es único; no promediar>
possible_causes: <hipótesis claramente marcadas, o ninguna>
next_step: <acción de revisión humana; nunca ajustar inventario>
```

## Criterios de aceptación

- Usa solo datos proporcionados y separa hechos de causas posibles; no inventa umbrales, conversiones, fuentes, fechas o políticas.
- Para unidades comparables, calcula exactamente `physical_count - system_quantity`, conserva el signo y la unidad, y clasifica el signo correctamente.
- Si faltan datos obligatorios, hay valores inválidos (incluido un conteo físico negativo) o unidades incompatibles, devuelve `insufficient_data` y no presenta una diferencia calculada. Excepción: un `system_quantity` negativo es una anomalía que devuelve `review_required` con diferencia provisional.
- Si hay ambigüedad, conteos repetidos distintos, duplicados, saldos del sistema negativos o desfase/movimientos que impiden confirmar comparabilidad, devuelve `review_required`; no promedia, selecciona ni reconcilia automáticamente.
- Si existen varios registros candidatos o conteos repetidos distintos, presenta sus valores por separado; nunca los sumes, promedies ni elijas uno como definitivo sin confirmación explícita.
- Si hay varios candidatos en un solo lado, deja `difference` como no calculada y coloca los deltas provisionales individuales en el campo de candidatos correspondiente. Si ambos lados tienen varios valores sin emparejamiento explícito, no calcules deltas combinados.
- Si `counted_at` falta o no es interpretable, devuelve `insufficient_data`; si falta o no es válida la fecha del snapshot del sistema, devuelve `review_required` y marca cualquier cálculo como provisional.
- Solo devuelve `no_discrepancy` si las cantidades comparables coinciden y no hay ambigüedades; solo devuelve `discrepancy_found` si difieren y son comparables.
- Informa la fuente y las fechas disponibles sin afirmar que los datos son en tiempo real ni decidir que están obsoletos con un umbral no proporcionado.
- No modifica existencias, llama APIs, crea incidencias ni envía alertas. La decisión y cualquier ajuste corresponden al personal autorizado.

## Casos de comprobación

En todos los casos, los campos obligatorios no mencionados se consideran válidos y presentes, salvo cuando se indique lo contrario.

| Caso | Inputs relevantes | Resultado esperado |
| --- | --- | --- |
| Coinciden | Sistema 24 y conteo 24, misma unidad, `system_as_of` igual a `counted_at` | `no_discrepancy`, diferencia `0`. |
| Sobrante físico | Sistema 24 y conteo 26, misma unidad, `system_as_of` igual a `counted_at` | `discrepancy_found`, `physical_overage`, diferencia `+2`. |
| Faltante físico | Sistema 24 y conteo 21, misma unidad, `system_as_of` igual a `counted_at` | `discrepancy_found`, `physical_shortage`, diferencia `-3`. |
| Falta cantidad | `physical_count` ausente | `insufficient_data`; solicita el conteo y no calcula diferencia. |
| Unidades incompatibles | Sistema 2 cajas, conteo 24 unidades; sin conversión proporcionada | `insufficient_data`; no convierte ni calcula diferencia. |
| Registros candidatos duplicados | Candidatos de sistema 24 y 25; conteo físico único 21; no se sabe si son duplicados o ubicaciones distintas | `review_required`; `difference` no calculada, informa deltas provisionales `-3` y `-4`, no elige ni suma. |
| Recuentos distintos | Sistema único 24; conteos repetidos 21 y 23 | `review_required`; informa deltas provisionales `-3` y `-1`; no promedia. |
| Desfase temporal | Sistema 24 a las 08:00, conteo 21 a las 09:30, sin historial de movimientos | `review_required`, diferencia provisional `-3`; no afirma que el saldo esté obsoleto ni que el desfase sea la causa. |
| Fecha del sistema desconocida | `system_as_of` ausente, cantidades válidas 24 y 21 | `review_required`, diferencia provisional `-3`; solicita la fecha del snapshot y mantiene el resultado como provisional hasta revisión humana. |
| Fecha del conteo inválida | `counted_at` ausente o no interpretable | `insufficient_data`; no calcula diferencia. |
| Saldo de sistema negativo | Sistema `-2`, conteo 3, misma unidad y `system_as_of` igual a `counted_at` | `review_required`, `anomalous_system_balance`, diferencia provisional `+5`; señala el saldo anómalo. |
| Valor inválido | Conteo físico `-1`, texto no numérico o infinito | `insufficient_data`; identifica el valor y no calcula diferencia. |

Si una situación no está cubierta, usa `review_required`, explica qué no se puede determinar con los datos disponibles y solicita revisión humana. Esta skill clasifica y prepara la revisión; no puede resolver físicamente una causa ni sustituir las reglas de cada almacén.