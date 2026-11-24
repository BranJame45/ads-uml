---
tipo: entregable
entregable: Flujo de Caso de Uso y Arquitectura de Vista
caso_de_uso: UC14. Gestionar movimientos de inventario
estado: finalizado (Arquitectura Estricta)
fecha: 2026-11-23
---
# UC14: Gestionar Movimientos de Inventario (Kardex y Anulación)

## Arquitectura de la Interfaz (Visión UX/UI)

Módulo de auditoría y corrección de errores de inventario. Actor: **Encargado de Almacén**. Nunca borra registros físicamente; toda corrección se realiza anulando el movimiento y revirtiendo automáticamente su impacto en `Producto.stock_fisico`.

### 1. Interfaz Principal: `<Gestión de Movimientos de Inventario>`
- **Bloque Superior (Filtros - `RF-123`):** `cboTipoMovimiento` (`Todos / Ingreso / Venta / Producción / Merma / Reserva`), `txtItem`, `txtFechaInicio`, `txtFechaFin` y botón **`[Filtrar]`** (`btnFiltrar`).
- **Bloque Central (DataGrid - `RF-122`):** Lista movimientos en orden cronológico descendente (`Fecha`, `Tipo`, `Ítem`, `Cantidad`, `Stock Resultante`, `Estado`, `Acciones`).
- **Botones por fila:** **`[Ver Detalle / Kardex]`** (`btnVerDetalle`) y **`[Anular]`** (`btnAnularMovimiento`).

### 2. Interfaz Secundaria: `<Modal Detalle y Kardex>`
- Muestra los datos completos del movimiento (`RF-124`: tipo, fecha, usuario, referencia, cantidad, stock resultante) y el historial Kardex del ítem (`RF-125`), más botón **`[Cerrar]`** (`btnCerrar`).

### 3. Interfaz Terciaria: `<Modal Anular Movimiento>`
- Muestra resumen del movimiento seleccionado, campo **`txtMotivoAnulacion`** (obligatorio, `RF-126`) y botones **`[Confirmar Anulación]`** (`btnConfirmarAnulacion`) y **`[Cancelar]`** (`btnCancelar`).

### 4. Comportamiento del Flujo
1. **Escenario Base (`RF-122`):** Lista todos los movimientos en orden cronológico descendente.
2. **Filtro (`RF-123`):** Filtra por tipo, ítem o rango de fechas sin `isset` (R4).
3. **Detalle y Kardex (`RF-124`, `RF-125`):** Muestra el detalle del movimiento y el Kardex del ítem para verificar el stock actual en función de sus movimientos.
4. **Anulación (`RF-126` a `RF-128`):** Verifica que el movimiento esté activo (`estado = 'Activo'`), exige motivo de anulación, revierte el efecto en `Producto.stock_fisico` (si fue entrada descuenta; si fue salida suma), y actualiza `Movimiento_Inventario` registrando `estado = 'Anulado'`, `motivo_anulacion`, fecha y usuario.

---

## Trazabilidad de Requerimientos
| RF | Cubierto por |
| :--- | :--- |
| RF-122 Listar movimientos en orden cronológico descendente | Grilla principal + `ORDER BY fecha DESC` |
| RF-123 Filtrar por tipo / ítem / rango de fechas | `case (boton <Filtrar>)` |
| RF-124 Mostrar detalle del movimiento seleccionado | `<Modal Detalle y Kardex>` (`obtenerDetalleMovimiento`) |
| RF-125 Mostrar stock actual en función a movimientos (Kardex) | `<Modal Detalle y Kardex>` (`obtenerKardexItem`) |
| RF-126 Solicitar anulación indicando motivo | `<Modal Anular Movimiento>` (`txtMotivoAnulacion`) |
| RF-127 Revertir automáticamente efecto en stock | `UPDATE Producto SET stock_fisico = stock_fisico +/- ?` |
| RF-128 Registrar anulación con fecha, usuario y motivo en BD | `UPDATE Movimiento_Inventario SET estado='Anulado', motivo_anulacion=?, ...` |
