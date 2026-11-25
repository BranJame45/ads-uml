---
tipo: entregable
entregable: Flujo de Caso de Uso y Arquitectura de Vista
caso_de_uso: UC11. Registrar ingreso de mercancía
estado: finalizado (Arquitectura Estricta)
fecha: 2026-11-23
---
# UC11: Registrar Ingreso de Mercancía

## Arquitectura de la Interfaz (Visión UX/UI)

Flujo transaccional de abastecimiento directo (sin órdenes de compra). Actor: **Encargado de Almacén**.

### 1. Interfaz Principal: `<Gestión de Ingresos de Mercancía>`
- **Bloque Superior:** Botón primario **`[+ Nuevo Ingreso]`** (`btnNuevoIngreso`).
- **Bloque Central (Historial de Ingresos - `RF-106`):** DataGrid con los ingresos registrados (`N° Ingreso`, `Fecha`, `Proveedor`, `Comprobante Proveedor`, `Costo Total`) y botón por fila **`[Ver Detalle]`** (`btnVerDetalle`).

### 2. Interfaz Secundaria: `<Modal Detalle de Ingreso>`
- Muestra los datos del proveedor, comprobante e ítems del ingreso seleccionado (`listadoDetalleIngreso`) y botón **`[Cerrar]`** (`btnCerrar`).

### 3. Interfaz Terciaria: `<Modal Formulario de Ingreso>`
- **Buscador de Proveedor (`RF-099`):** `txtBuscarProveedor` y botón **`[Buscar Proveedor]`** (`btnBuscarProveedor`, sin `isset` por R4) + `lblProveedorSeleccionado`.
- **Datos de Cabecera (`RF-098`, `RF-102`):** `txtFecha` (actual por defecto) y `txtNumComprobanteProv` (guía o factura de compra).
- **Agregar Ítems al Detalle (`RF-100`, `RF-101`):** `cboItem` (insumos y productos activos), `txtCantidad`, `txtCostoUnitario`, botón **`[Agregar Ítem]`** (`btnAgregarItem`).
- **Grilla Temporal de Detalle:** Ítems agregados en sesión (`Ítem`, `Cantidad`, `Costo Unitario`, `Subtotal`, botón `[Eliminar Ítem]`) y `lblCostoTotal` calculado automáticamente.
- **Pie del Modal:** **`[Confirmar Ingreso]`** (`btnConfirmarIngreso`) y **`[Cancelar]`** (`btnCancelar`).

### 3. Comportamiento del Flujo
1. **Escenario Base (`RF-106`):** Lista el historial de ingresos con fecha, proveedor y costo total.
2. **Nuevo Ingreso:** Presiona `[+ Nuevo Ingreso]`, carga los ítems activos (`Producto` de tipo Insumo y Terminado) y abre `<Modal Formulario de Ingreso>`.
3. **Buscar Proveedor (`RF-099`):** Filtra proveedores activos por razón social o RUC y selecciona uno.
4. **Gestionar Detalle en Sesión (`RF-100`, `RF-101`):** Agrega o elimina ítems en el arreglo temporal de sesión, recalculando `subtotal` y `costo_total`.
5. **Cancelación (`RF-105`):** Si presiona `[Cancelar]`, limpia la sesión temporal y cierra el modal sin modificar stock.
6. **Confirmación (`RF-103`, `RF-104`):** Genera correlativo `num_ingreso`, inserta en `Ingreso_Mercancia` y `Detalle_Ingreso`, suma `stock_fisico` de cada ítem en `Producto`, limpia sesión y recarga el historial.

---

## Trazabilidad de Requerimientos
| RF | Cubierto por |
| :--- | :--- |
| RF-098 Formulario con proveedor / fecha / lista de ítems | `<Modal Formulario de Ingreso>` |
| RF-099 Buscar y seleccionar proveedor por razón social o RUC | `case (boton <Buscar Proveedor>)` |
| RF-100 Agregar múltiples ítems al detalle | `case (boton <Agregar Ítem>)` en sesión temporal |
| RF-101 Calcular automáticamente el costo total | Recálculo de `subtotal` y `costo_total` |
| RF-102 Registrar número de comprobante del proveedor | `txtNumComprobanteProv` → `num_comprobante_prov` |
| RF-103 Sumar automáticamente el stock físico al confirmar | `UPDATE Producto SET stock_fisico = stock_fisico + ?` |
| RF-104 Guardar ingreso y generar número autoincrementado | `SELECT MAX(num_ingreso)+1` + `INSERT INTO Ingreso_Mercancia` |
| RF-105 Cancelar sin afectar el stock | Rama `boton <Cancelar>` + limpiar sesión |
| RF-106 Mostrar historial de ingresos con fecha/proveedor/total | Grilla principal en `<Gestión de Ingresos de Mercancía>` |
