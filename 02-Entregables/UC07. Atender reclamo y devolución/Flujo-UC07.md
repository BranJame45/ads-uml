---
tipo: entregable
entregable: Flujo de Caso de Uso y Arquitectura de Vista
caso_de_uso: UC07. Atender reclamo y devolución
estado: finalizado (Arquitectura Estricta)
fecha: 2026-11-23
---
# UC07: Atender Reclamo y Devolución

## Arquitectura de la Interfaz (Visión UX/UI)

Flujo transaccional postventa. Actor: **Vendedor**. Permite buscar una venta completada por número de comprobante, revisar su detalle y registrar un reclamo con resolución de cambio de producto o devolución de dinero.

### 1. Interfaz Principal: `<Atender Reclamo y Devolución>`
- **Bloque Superior (Buscador):** `txtComprobante` (*"Ingrese N° de comprobante..."*) y botón **`[Buscar Venta]`** (`btnBuscarVenta`).
- **Bloque Central (Detalle de Venta):** Tarjeta con fecha, cliente, monto total (`lblFecha`, `lblCliente`, `lblTotal`) y grilla con los productos de la venta (`listadoDetalleVenta`).
- **Acción:** Botón **`[Registrar Reclamo / Devolución]`** (`btnRegistrarReclamo`).

### 2. Interfaz Secundaria: `<Modal Formulario de Reclamo>`
- **Inputs:**
  - `cboProducto` (ítem de la venta a reclamar)
  - `txtCantidad` (cantidad a devolver/cambiar, `> 0` y `<= cantidad vendida`)
  - `txtMotivo` (motivo del reclamo, obligatorio)
  - `cboTipoResolucion` (`Cambio de producto` / `Devolución de dinero`)
  - `lblMontoDevuelto` (calculado automáticamente si es `Devolución de dinero`)
- **Pie:** **`[Confirmar Reclamo]`** (`btnConfirmarReclamo`) y **`[Cancelar]`** (`btnCancelar`).

### 3. Comportamiento del Flujo
1. **Búsqueda (`RF-064`, `RF-065`):** Ingresa el número de comprobante y presiona `[Buscar Venta]` (sin `isset` por R4). Si existe venta completada, muestra sus datos y detalle.
2. **Apertura de Reclamo:** Presiona `[Registrar Reclamo / Devolución]`, se valida `isset` y venta seleccionada, y se abre `<Modal Formulario de Reclamo>`.
3. **Cancelación (`RF-071`):** Si presiona `[Cancelar]`, cierra el modal sin modificar stock ni registrar cambios.
4. **Confirmación (`RF-066` a `RF-070`):** Valida campos y cantidad (`<= cantidad` de la venta). Registra en `Reclamo` (incluyendo `monto_devuelto` negativo/egreso si es devolución de dinero), devuelve el producto retornado al `stock_fisico` (`+ cantidad`), y si es `Cambio de producto` descuenta el producto entregado en reemplazo (`- cantidad`).

---

## Trazabilidad de Requerimientos
| RF | Cubierto por |
| :--- | :--- |
| RF-064 Buscar venta por número de comprobante | `case (boton <Buscar Venta>)` + `SELECT ... WHERE num_comprobante = ?` |
| RF-065 Mostrar detalle de la venta (fecha/productos/cliente/total) | Tarjeta y grilla en `<Atender Reclamo y Devolución>` |
| RF-066 Registrar motivo del reclamo | `txtMotivo` en `<Modal Formulario de Reclamo>` + `INSERT INTO Reclamo` |
| RF-067 Seleccionar tipo de resolución (cambio o devolución de dinero) | `cboTipoResolucion` |
| RF-068 Devolver al stock el producto retornado | `UPDATE Producto SET stock_fisico = stock_fisico + ?` |
| RF-069 Registrar movimiento de caja negativo (devolución de dinero) si aplica | `monto_devuelto` en `INSERT INTO Reclamo` cuando es `Devolución de dinero` |
| RF-070 Guardar registro del reclamo con su resolución en BD | `INSERT INTO Reclamo (...)` |
| RF-071 Cancelar proceso sin registrar cambios | Rama `boton <Cancelar>` en `<Modal Formulario de Reclamo>` |
