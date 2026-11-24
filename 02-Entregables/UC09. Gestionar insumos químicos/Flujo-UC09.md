---
tipo: entregable
entregable: Flujo de Caso de Uso y Arquitectura de Vista
caso_de_uso: UC09. Gestionar insumos químicos
estado: finalizado (Arquitectura Estricta)
fecha: 2026-11-23
---
# UC09: Gestionar Insumos Químicos (CRUD con Alerta de Stock)

## Arquitectura de la Interfaz (Visión UX/UI)

Mismo patrón de catálogo que `UC10`: **Grilla Principal + Modal de Formulario**. Actor: **Encargado de Almacén**. Los insumos químicos se almacenan en la tabla `Producto` discriminados por `tipo = 'Insumo'`.

### 1. Interfaz Principal: `<Gestión de Insumos>`
- **Bloque Superior:** `txtBuscador` (*"Buscar por nombre o categoría..."*), botón **`[Buscar]`** (`btnBuscar`) y botón primario **`[+ Nuevo Insumo]`** (`btnNuevoInsumo`).
- **Bloque Central (DataGrid):** Lista insumos activos (`tipo = 'Insumo' AND estado = 'Activo'`) con columnas: Nombre, Stock Actual, Unidad de Medida, Estado de Stock (`Normal` / `Crítico` resaltado cuando `stock_fisico <= stock_minimo`) y Acciones.
- **Botones por fila:** **`[Editar]`** (`btnEditarInsumo`) y **`[Dar de Baja]`** (`btnDarBajaInsumo`).

### 2. Interfaz Secundaria: `<Modal Formulario de Insumo>`
- **Inputs:**
  - `txtNombre` (Obligatorio)
  - `txtDescripcion`
  - `cboCategoria` (Obligatorio)
  - `cboUnidad` (Obligatorio)
  - `cboProveedor` (Obligatorio, proveedor asociado)
  - `txtStockMinimo` (Obligatorio)
  - `txtCostoUnitario` (Obligatorio, precio de costo)
  - *(Al editar)* `lblStockActual` (sólo lectura)
- **Pie:** **`[Guardar Nuevo]`** / **`[Guardar Cambios]`** y **`[Cancelar]`**.

### 3. Comportamiento del Flujo
1. **Escenario Base:** Consulta insumos activos (`tipo = 'Insumo' AND estado = 'Activo'`) con `INNER JOIN` a `Categoria`, `Unidad_Medida` y `CASE` para estado de stock (`Normal` / `Crítico`).
2. **Búsqueda (`RF-085`):** Filtra por nombre o categoría sin `isset` (R4).
3. **Creación (`RF-080` a `RF-082`):** Precarga categorías, unidades y proveedores activos; valida campos obligatorios y unicidad de nombre entre insumos; inserta con `stock_fisico = 0`, `tipo = 'Insumo'`, `estado = 'Activo'`.
4. **Edición (`RF-086`, `RF-087`):** Precarga datos del insumo y maestros; valida nombre único contra otros insumos; actualiza `nombre`, `id_categoria`, `id_unidad`, `id_prov`, `stock_minimo`, `costo_unitario`.
5. **Baja Lógica (`RF-088`):** Muestra `<Panel de Confirmación>`; si Aceptar, verifica existencia activa y ejecuta `UPDATE estado = 'Inactivo'`.

---

## Trazabilidad de Requerimientos
| RF | Cubierto por |
| :--- | :--- |
| RF-080 Formulario con nombre/descripción/unidad/proveedor/stock mínimo/precio costo | `<Modal Formulario de Insumo>` |
| RF-081 Validar nombre del insumo no duplicado | `SELECT id_prod FROM Producto WHERE nombre = ? AND tipo = 'Insumo'` |
| RF-082 Guardar nuevo insumo con stock inicial en cero | `INSERT INTO Producto ... VALUES (..., 0, 'Insumo', 'Activo')` |
| RF-083 / RF-084 Listar con stock, unidad, estado y alerta visual | Grilla `<Gestión de Insumos>` + `CASE WHEN stock_fisico <= stock_minimo` |
| RF-085 Buscar insumo por nombre o categoría | `case (boton <Buscar>)` con `INNER JOIN Categoria` |
| RF-086 / RF-087 Editar nombre/stock mínimo/precio de costo y guardar | `case (boton <Editar Insumo>)` + `UPDATE Producto ...` |
| RF-088 Dar de baja (desactivar) sin eliminar | `case (boton <Dar de Baja Insumo>)` + `UPDATE estado = 'Inactivo'` |
