---
tipo: entregable
entregable: Flujo de Caso de Uso y Arquitectura de Vista
caso_de_uso: UC05. Gestionar clientes
estado: finalizado (Arquitectura Estricta)
fecha: 2026-11-23
---
# UC05: Gestionar Clientes

## Arquitectura de la Interfaz (Visión UX/UI)

Patrón **Grilla Principal + Modal de Formulario + Modal de Historial**. Actor: **Vendedor**. La identidad del cliente es su número de documento (`DNI` 8 dígitos o `RUC` 11 dígitos), único e inmutable tras el registro.

### 1. Interfaz Principal: `<Gestión de Clientes>`
- **Bloque Superior:** `txtBuscador` (*"Buscar por documento o nombre..."*), botón **`[Buscar]`** (`btnBuscar`) y botón primario **`[+ Nuevo Cliente]`** (`btnNuevoCliente`).
- **Bloque Central (DataGrid):** Lista clientes activos (`estado = 'Activo'`) con columnas: Documento, Nombre / Razón Social, Teléfono, Correo, Acciones.
- **Botones por fila:** **`[Editar]`** (`btnEditarCliente`), **`[Eliminar]`** (`btnEliminarCliente`) y **`[Historial]`** (`btnHistorialCliente`).

### 2. Interfaz Secundaria: `<Modal Formulario de Cliente>`
- **Inputs:**
  - `cboTipoDocumento` (`DNI` / `RUC`, obligatorio, sólo lectura al editar)
  - `txtDocumento` (Obligatorio, 8 dígitos DNI / 11 dígitos RUC, sólo lectura al editar)
  - `txtNombre` (Obligatorio: Nombre o Razón Social)
  - `txtTelefono`
  - `txtCorreo`
  - `txtDireccion`
- **Pie:** **`[Guardar]`** (`btnGuardarNuevo` / `btnGuardarCambios`) y **`[Cancelar]`** (`btnCancelar`).

### 3. Interfaz Terciaria: `<Modal Historial de Compras>`
- Muestra las ventas asociadas al cliente seleccionado (`fecha`, `num_comprobante`, `tipo_comprobante`, `total`, `estado`) y el botón **`[Cerrar]`** (`btnCerrar`).

### 4. Comportamiento del Flujo
1. **Escenario Base:** Lista todos los clientes activos (`SELECT ... WHERE estado = 'Activo'`).
2. **Búsqueda (`RF-036`):** Filtra por documento o nombre sin `isset` (recarga la misma grilla).
3. **Creación (`RF-030` a `RF-034`):** Valida campos obligatorios, formato/longitud (`8` para DNI, `11` para RUC) y unicidad por `documento`; inserta con `estado = 'Activo'`.
4. **Edición (`RF-037`, `RF-038`):** Precarga datos (documento en sólo lectura) y actualiza datos de contacto (`nombre`, `telefono`, `correo`, `direccion`).
5. **Eliminación (`RF-039`):** Solicita confirmación; verifica existencia activa y que `COUNT(*)` de `Venta` sea `0`; si no tiene ventas, realiza baja lógica (`UPDATE estado = 'Inactivo'`).
6. **Historial (`RF-040`):** Consulta las ventas del cliente en `Venta` y las muestra en `<Modal Historial de Compras>`.

---

## Trazabilidad de Requerimientos
| RF | Cubierto por |
| :--- | :--- |
| RF-030 Formulario con tipo doc / nro doc / nombre / teléfono / correo / dirección | `<Modal Formulario de Cliente>` |
| RF-031 Validar tipo y longitud (8 DNI / 11 RUC) | Rombo `¿Formato y longitud válidos?` |
| RF-032 / RF-033 Verificar documento no duplicado y alertar | `SELECT id_cliente FROM Cliente WHERE documento = ?` |
| RF-034 Guardar nuevo cliente en BD | `INSERT INTO Cliente ...` |
| RF-035 Listar clientes con nombre / documento / teléfono | Grilla `<Gestión de Clientes>` |
| RF-036 Buscar cliente por documento o nombre | `case (boton <Buscar>)` |
| RF-037 / RF-038 Editar datos de contacto y guardar | `case (boton <Editar Cliente>)` + `UPDATE Cliente ...` |
| RF-039 Eliminar cliente si no tiene ventas asociadas | `case (boton <Eliminar Cliente>)` + `SELECT COUNT(*) FROM Venta` + `UPDATE estado='Inactivo'` |
| RF-040 Mostrar historial de compras del cliente | `case (boton <Historial>)` + `<Modal Historial de Compras>` |
