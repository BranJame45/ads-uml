---
tipo: entregable
entregable: Flujo de Caso de Uso y Arquitectura de Vista
caso_de_uso: UC10. Gestionar productos de limpieza
estado: finalizado (Arquitectura Estricta)
fecha: 2026-10-13
---
# UC10: Gestionar Productos de Limpieza (CRUD con Alerta de Stock)

## Arquitectura de la Interfaz (Visión UX/UI)

Mismo patrón de catálogo que `UC04`/`UC08`: **Grilla Principal + Modal de Formulario**. Actor: **Encargado de Almacén**. Dos particularidades de fondo:
- **Dos estados distintos:** el *estado del registro* (Activo/Inactivo, para la baja lógica) y el *estado de stock* (Normal/Crítico, calculado `stock_fisico <= stock_minimo`). La grilla muestra el **estado de stock** y resalta en alerta los productos críticos.
- **El stock no se gestiona aquí:** el producto nace con stock 0 y su stock físico solo lo mueven otros CUS (producción, ingreso de mercancía, merma, ventas). En este CRUD el stock físico es **sólo lectura**.

Los productos de limpieza son registros de la tabla `Producto` con `tipo = 'Terminado'` (misma tabla que los insumos de UC09/UC12, discriminada por `tipo`). **Categoría y Unidad son maestros**: tablas aparte (`Categoria`, `Unidad_Medida`) referenciadas por FK desde `Producto`. Los combos `cboCategoria` y `cboUnidad` se llenan con los registros activos de esos maestros al abrir el formulario.

### 1. Interfaz Principal: `<Gestión de Productos>`
- **Bloque Superior (Filtros y Acciones):**
  - `txtBuscador` con placeholder *"Buscar por nombre o categoría..."* y botón **`[Buscar]`**.
  - A la derecha, botón primario **`[+ Nuevo Producto]`**.
- **Bloque Central (DataGrid):** solo productos **activos**.
  - **Columnas:**
    1. Nombre
    2. Precio de Venta
    3. Stock Actual
    4. Estado de Stock (*Normal* / *Crítico*; la fila **Crítica** se resalta en alerta cuando `stock_fisico <= stock_minimo`)
    5. Acciones
  - **Botones por fila:**
    - Botón **`[Editar]`** (ícono de lápiz).
    - Botón **`[Dar de Baja]`** (ícono de papelera; dispara una **confirmación** y luego la **baja lógica**).

### 2. Interfaz Secundaria: `<Modal Formulario de Producto>`
Aparece al presionar `[Nuevo Producto]` o `[Editar]`.
- **Inputs:**
  - `txtNombre` (Obligatorio)
  - `txtDescripcion`
  - `cboCategoria` (Obligatorio)
  - `cboUnidad` (Obligatorio)
  - `txtPrecioVenta` (Obligatorio)
  - `txtStockMinimo` (Obligatorio)
  - *(Al editar)* `lblStockActual` — **sólo lectura**, informativo; no se modifica en este CUS.
- **Pie del Modal:** `[Guardar]` (primario) y `[Cancelar]` (secundario).

### 3. Comportamiento del Flujo (Interacción Paso a Paso)
1. **Escenario Base:** el Encargado entra. El Backend ejecuta un `SELECT` de productos `tipo='Terminado'` y `estado='Activo'`, calcula el estado de stock (`CASE`) y renderiza la grilla resaltando los críticos.
2. **Búsqueda:** escribe "Lejía" y presiona `[Buscar]`. El Backend filtra por nombre o categoría (solo activos) y recarga la grilla.
3. **Creación:** presiona `[+ Nuevo Producto]` (pasa por el controlador con `isset`). El Backend consulta **categorías activas** (`Categoria`) y **unidades activas** (`Unidad_Medida`) para precargar los combos, y abre el modal vacío. Llena el formulario y `[Guardar]`. Si el **nombre ya existe** o faltan campos → `<Panel de Error>`. Si es correcto, inserta con **stock inicial 0**, `estado='Activo'` y los FK de categoría/unidad elegidos, y muestra `<Panel de Éxito>`.
4. **Edición:** presiona `[Editar]`. El Backend precarga los datos del producto **y** los maestros (categorías, unidades) para los combos; el `stock actual` se muestra en sólo lectura. Modifica los campos (incluidas categoría/unidad si corresponde) y `[Guardar]`. Valida que el **nombre no choque con otro producto** y actualiza.
5. **Baja Lógica (con confirmación):** presiona `[Dar de Baja]`. Aparece `<Panel de Confirmación>` *"¿Está seguro de dar de baja este producto?"*. Si **Cancelar**, vuelve sin cambios. Si **Aceptar**, verifica que exista y esté activo y ejecuta `UPDATE estado='Inactivo'`. El producto desaparece de la grilla.

---

## Lógica Transaccional y de Seguridad (Backend)

1. **Uso de `isset()` (criterio: ¿hay una petición al controlador?):**
   - `Buscar`: **NO** (solo recarga su propia grilla).
   - `Nuevo Producto` y `Editar Producto`: **SÍ** (abrir el formulario es transición de interfaz; Editar además hace `SELECT` de precarga).
   - `Guardar Nuevo`, `Guardar Cambios`, `Dar de Baja Producto`: **SÍ** (DML / transición).
   - `Aceptar` de la confirmación de baja: **SÍ** (disparador real del `UPDATE`).

2. **Unicidad por nombre (editable):** el **nombre** es la clave de negocio y debe ser único entre productos `Terminado`. Al crear se valida `WHERE nombre = ? AND tipo = 'Terminado'`; al editar, como el nombre **sí es modificable**, se valida contra **otros** registros: `WHERE nombre = ? AND tipo = 'Terminado' AND id_prod != ?`.

3. **Stock gestionado fuera de este CUS:** el producto nace con `stock_fisico = 0` (RF-091). La edición **nunca** incluye `stock_fisico` en el `SET`; el stock solo cambia por producción, ingreso de mercancía, merma y ventas.

3.1. **Categoría y Unidad como maestros:** `categoria` y `unidad` se modelan como tablas aparte (`Categoria`, `Unidad_Medida`) referenciadas por FK (`id_categoria`, `id_unidad`) desde `Producto`. Al abrir el formulario (Nuevo o Editar), el Backend consulta ambos maestros filtrando por `estado='Activo'` para llenar los combos. La búsqueda por categoría (RF-094) usa `JOIN` contra `Categoria` filtrando por `c.nombre LIKE`. **Nota de scope:** la administración de esos maestros (CRUD de categorías y unidades) no está cubierta en los RF de UC10 — es un gap de la matriz a resolver si el negocio requiere crear/modificar categorías en vivo.

4. **Alerta de stock (RF-093):** el estado de stock se calcula en la consulta con `CASE WHEN stock_fisico <= stock_minimo THEN 'Crítico' ELSE 'Normal' END`; la vista resalta las filas críticas.

5. **Casos Negativos Cubiertos:**
   - Buscar con campo vacío → Error.
   - Guardar producto con campos obligatorios en blanco → Error.
   - Registrar un nombre que ya existe → Error.
   - Editar el nombre a uno que ya pertenece a otro producto → Error.
   - Editar/Dar de baja un producto que ya no existe → Error.

6. **Baja Lógica con Confirmación (RF-097):** "dar de baja" es un `UPDATE Producto SET estado = 'Inactivo'`, nunca un `DELETE`. Exige `<Panel de Confirmación>` previo y, al Aceptar, verificar existencia activa antes del `UPDATE`. La grilla lista solo `estado = 'Activo'`.

---

## Trazabilidad de Requerimientos
| RF | Cubierto por |
| :--- | :--- |
| RF-089 Formulario nombre/descripción/categoría/unidad/precio/stock mínimo | Modal `<Formulario de Producto>` |
| RF-090 Validar nombre no duplicado | `¿Nombre ya registrado?` + `SELECT ... WHERE nombre = ?` |
| RF-091 Guardar con stock inicial en cero | `INSERT ... stock_fisico = 0` |
| RF-092 Listar con nombre/precio/stock y estado (normal/crítico) | Grilla + `SELECT` con `CASE` de estado de stock |
| RF-093 Alerta visual cuando stock ≤ stock mínimo | `CASE ... 'Crítico'` + resaltado en grilla |
| RF-094 Buscar por nombre o categoría | case `<Buscar>` |
| RF-095 Editar nombre/precio/stock mínimo | case `<Editar Producto>` + `UPDATE` de esos campos |
| RF-096 Guardar cambios | `UPDATE Producto ...` |
| RF-097 Dar de baja (desactivar) sin eliminar | case `<Dar de Baja Producto>`: confirmación → `UPDATE estado='Inactivo'` |
