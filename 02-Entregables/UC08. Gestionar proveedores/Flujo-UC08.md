---
tipo: entregable
entregable: Flujo de Caso de Uso y Arquitectura de Vista
caso_de_uso: UC08. Gestionar proveedores
estado: finalizado (Arquitectura Estricta)
fecha: 2026-10-07
---
# UC08: Gestionar Proveedores (CRUD con Borrado Lógico)

## Arquitectura de la Interfaz (Visión UX/UI)

Mismo patrón de catálogo que `UC04`: **Grilla Principal + Modal de Formulario**. El actor es el **Encargado de Almacén**, dueño del abastecimiento. La "eliminación" es un **borrado lógico** (`UPDATE estado = 'Inactivo'`), nunca un `DELETE` físico, y exige un **mensaje de confirmación** previo. La grilla lista **solo proveedores activos**.

### 1. Interfaz Principal: `<Gestión de Proveedores>`
**Distribución Visual (Layout Lógico):**

- **Bloque Superior (Filtros y Acciones):**
  - Un cuadro de texto `txtBuscador` con placeholder *"Buscar por razón social o RUC..."*.
  - Un botón secundario **`[Buscar]`** al costado.
  - Alineado a la derecha, un botón primario **`[+ Nuevo Proveedor]`**.

- **Bloque Central (DataGrid):**
  - Una tabla con **solo los proveedores activos** (`estado = 'Activo'`).
  - **Columnas:**
    1. Razón Social
    2. RUC
    3. Teléfono
    4. Correo
    5. Acciones (contenedor de mini botones)
  - **Botones por fila:**
    - Botón **`[Editar]`** (ícono de lápiz).
    - Botón **`[Eliminar]`** (ícono de papelera; dispara una **confirmación** y luego el **borrado lógico condicionado**, ver backend).

### 2. Interfaz Secundaria: `<Modal Formulario de Proveedor>`
**Distribución Visual (Layout Lógico):**

Aparece cuando se presiona `[Nuevo Proveedor]` o `[Editar]`.

- **Bloque de Datos (Inputs de Texto):**
  - `txtRazonSocial` (Obligatorio)
  - `txtRUC` (Obligatorio). **Es el identificador del proveedor.** Editable **solo al crear**; al editar se muestra en **sólo lectura** (deshabilitado), nunca se modifica.
  - `txtTelefono` (Obligatorio)
  - `txtCorreo` (Obligatorio)
  - `txtDireccion` (Obligatorio)

- **Pie del Modal (Botones):**
  - Botón **`[Guardar]`** (verde/primario, hace el POST).
  - Botón **`[Cancelar]`** (gris/secundario, cierra el modal sin hacer nada).

### 3. Comportamiento del Flujo (Interacción Paso a Paso)
1. **Escenario Base:** el Encargado entra a la vista. El Backend ejecuta un `SELECT` filtrado por `estado = 'Activo'` y renderiza la tabla solo con proveedores activos.
2. **Búsqueda:** escribe "Novaquímica" y presiona `[Buscar]`. La página recarga, el Backend filtra `estado = 'Activo' AND (razón social LIKE … OR RUC exacto)` y devuelve la grilla con las coincidencias activas.
3. **Flujo de Creación:** presiona `[+ Nuevo Proveedor]` (la petición pasa por el controlador con `isset`), que abre el formulario vacío con `txtRUC` **editable**. Llena los datos y presiona `[Guardar]`. Si **ya existe un proveedor con ese RUC** o faltan campos, devuelve `<Panel de Error>`. Si todo es correcto, inserta y muestra `<Panel de Éxito>`.
4. **Flujo de Edición:** presiona `[Editar]` en una fila. El Backend precarga los datos del proveedor y abre el modal con el `txtRUC` en **sólo lectura**. Modifica razón social, teléfono, correo o dirección (no el RUC) y presiona `[Guardar]`. Valida, actualiza y devuelve `<Panel de Éxito>`.
5. **Flujo de Eliminación (Lógica, con confirmación):** presiona `[Eliminar]`. El sistema muestra un `<Panel de Confirmación>` *"¿Está seguro de eliminar este proveedor?"*. Si elige **Cancelar**, vuelve a la vista sin cambios. Si elige **Aceptar**, el Backend verifica que el proveedor exista y esté activo y ejecuta `UPDATE estado = 'Inactivo'` (borrado lógico), mostrando `<Panel de Éxito>`. El proveedor desaparece de la grilla (que solo lista activos).

---

## Lógica Transaccional y de Seguridad (Backend)

1. **Uso de `isset()` (criterio: ¿hay una petición al controlador?):**
   - `isset()` es el guardián del controlador. En MVC estricto una interfaz no abre otra por sí sola; la navegación la decide el controlador. Un botón lo lleva cuando implica **(a) transición a otra interfaz**, (b) lectura de datos (`SELECT` de precarga), (c) DML, o (d) escritura de sesión. **Única excepción:** la búsqueda/filtro que solo repinta la MISMA vista.
   - `Buscar`: **NO** lleva `isset()` (solo recarga su propia grilla).
   - `Nuevo Proveedor` y `Editar Proveedor`: **SÍ** (abrir el formulario es transición de interfaz; Editar además hace `SELECT` de precarga).
   - `Guardar Nuevo`, `Guardar Cambios`, `Eliminar Proveedor`: **SÍ** (operaciones DML / transición).
   - `Aceptar` de la confirmación de eliminación: **SÍ** (es el disparador real del `UPDATE`; confirmación de alerta antes de continuar).

2. **Identidad por RUC:** el **RUC es el identificador del proveedor**: único e inmutable. La unicidad se valida al crear (`SELECT ... WHERE ruc = ?`, RF-073); al editar, el RUC viaja solo como referencia a través del `id_prov` y **nunca** se incluye en el `SET` del `UPDATE`.

3. **Casos Negativos Cubiertos:**
   - Buscar con campo vacío → Error.
   - Guardar proveedor con campos obligatorios en blanco → Error.
   - Registrar un RUC que ya existe en la BD → Error.
   - Editar/Eliminar un proveedor que ya no existe → Error.

4. **Borrado Lógico con Confirmación (RF-079):** la "eliminación" es un `UPDATE Proveedor SET estado = 'Inactivo'`, nunca un `DELETE`. El flujo exige: (1) `<Panel de Confirmación>` previo; (2) al **Aceptar**, verificar que el proveedor exista y esté activo; (3) ejecutar el `UPDATE`. La grilla lista solo `estado = 'Activo'`, por lo que el proveedor inhabilitado deja de mostrarse. El campo `estado` se fija en `'Activo'` al crear (INSERT).

---

## Trazabilidad de Requerimientos
| RF | Cubierto por |
| :--- | :--- |
| RF-072 Formulario con razón social/RUC/teléfono/correo/dirección | Modal `<Formulario de Proveedor>` |
| RF-073 Validar RUC no duplicado | `¿RUC ya registrado?` + `SELECT ... WHERE ruc = ?` |
| RF-074 Guardar nuevo proveedor | `INSERT INTO Proveedor ...` |
| RF-075 Listar los proveedores con nombre/RUC/teléfono | Grilla + `SELECT ... WHERE estado = 'Activo'` (los inhabilitados se consideran eliminados) |
| RF-076 Buscar por razón social o RUC | case `<Buscar>` |
| RF-077 Editar datos del proveedor | case `<Editar Proveedor>` |
| RF-078 Guardar cambios | `UPDATE Proveedor ...` |
| RF-079 Eliminar proveedor | case `<Eliminar Proveedor>`: confirmación → `UPDATE estado='Inactivo'` (borrado lógico) |
