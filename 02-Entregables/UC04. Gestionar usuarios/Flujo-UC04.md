---
tipo: entregable
entregable: Flujo de Caso de Uso y Arquitectura de Vista
caso_de_uso: UC04. Gestionar usuarios
estado: finalizado (Arquitectura Estricta)
fecha: 2026-10-06
---
# UC04: Gestionar Usuarios (CRUD Base)

## Arquitectura de la Interfaz (Visión UX/UI)

Este caso de uso es la base para todos los catálogos del sistema (CRUDs). Está estructurado en un patrón clásico de **Grilla Principal + Modal de Formulario**, lo que permite crear y editar sin perder el contexto visual de la lista de registros.

### 1. Interfaz Principal: `<Gestión de Usuarios>`
**Distribución Visual (Layout Lógico):**

- **Bloque Superior (Filtros y Acciones):**
  - Un cuadro de texto `txtBuscador` con placeholder *"Buscar por DNI, nombre o rol..."*.
  - Un botón gris/secundario **`[Buscar]`** al costado.
  - Alineado a la derecha, un botón grande y primario **`[+ Nuevo Usuario]`**.

- **Bloque Central (DataGrid):**
  - Una tabla que muestra **todos los usuarios** del sistema (activos **e** inactivos), para poder reactivar a los deshabilitados.
  - **Columnas:** 
    1. DNI (Documento)
    2. Nombre Completo
    3. Nombre de Usuario (Username)
    4. Roles (Una cadena separada por comas, ej: *Vendedor, Encargado*)
    5. **Estado** (etiqueta visual: *Activo* en verde / *Inactivo* en gris)
    6. Acciones (Contenedor de mini botones)
  - **Botones por fila:**
    - Botón **`[Editar]`** (Ícono de lápiz).
    - Botón **`[Cambiar Estado]`** (toggle de borrado lógico): en una fila **Activa** actúa como *Deshabilitar* (ícono/texto rojo); en una fila **Inactiva** actúa como *Habilitar* (ícono/texto verde). Nunca elimina físicamente.

### 2. Interfaz Secundaria: `<Modal Formulario de Usuario>`
**Distribución Visual (Layout Lógico):**

Aparece flotando sobre la grilla principal cuando se presiona `[Nuevo Usuario]` o `[Editar]`.

- **Bloque de Datos Personales (Inputs de Texto):**
  - `txtDNI` (Obligatorio). **Es el identificador del usuario.** Editable **solo al crear**; al editar se muestra en **sólo lectura** (deshabilitado), nunca se modifica.
  - `txtNombreCompleto` (Obligatorio)
  - `txtUsername` (Obligatorio). Es una **credencial de acceso**, no la identidad; sí es editable al modificar.
  - `txtPassword` (Obligatorio al crear. Al editar, lleva el placeholder *"Dejar en blanco para mantener actual"*).

- **Bloque de Permisos (Checkboxes de Roles):**
  - Un recuadro o sección titulada "Asignación de Roles".
  - Una lista vertical u horizontal de Checkboxes (múltiple selección permitida):
    - `[ ] Administrador`
    - `[ ] Vendedor`
    - `[ ] Encargado de Almacén`
  - *(Nota UX: El backend exige que al menos un checkbox esté marcado antes de guardar).*

- **Pie del Modal (Botones):**
  - Botón **`[Guardar]`** (Verde/Primario, hace el POST de los datos).
  - Botón **`[Cancelar]`** (Rojo o Gris/Secundario, cierra el modal sin hacer nada).

### 3. Comportamiento del Flujo (Interacción Paso a Paso)
Esta narrativa define la secuencia exacta para que cualquier IA o desarrollador comprenda el ciclo de vida de la vista:

1. **Escenario Base:** El administrador entra a la vista. El Backend ejecuta un `SELECT` global (sin filtrar por estado) y renderiza la tabla con **todos** los usuarios, mostrando la columna Estado.
2. **Búsqueda:** El administrador escribe "Juan" y presiona `[Buscar]`. La página se recarga, el Backend filtra con `LIKE` (también sin filtrar por estado) y devuelve la grilla solo con las coincidencias, activas o inactivas.
3. **Flujo de Creación:**
   - Presiona `[+ Nuevo Usuario]`. La petición pasa por el controlador (guardián `isset`), que abre el formulario `<Formulario de Usuario>` vacío con el `txtDNI` **editable**.
   - Llena los datos, marca los checkboxes "Vendedor" y "Encargado de Almacén", y presiona `[Guardar]`.
   - Se envía un POST al Backend. Si **ya existe un usuario con ese DNI** o faltan campos, el Backend devuelve la vista con un `<Panel de Error>`. Si todo es correcto, inserta los datos, cierra el modal visualmente y muestra `<Panel de Éxito>`.
4. **Flujo de Edición:**
   - En la grilla, presiona el botón `[Editar]` de la fila de "María".
   - El Backend busca los datos de María (incluyendo sus roles) y carga la página abriendo el Modal con sus datos ya escritos en los *TextBox* y sus checkboxes de rol ya marcados. El `txtDNI` aparece en **sólo lectura**.
   - María modifica su nombre (no su DNI) y presiona `[Guardar]`. El Backend valida, actualiza y devuelve el `<Panel de Éxito>`.
5. **Flujo de Cambio de Estado (Borrado Lógico Bidireccional):**
   - Presiona el botón `[Cambiar Estado]` en la fila.
   - Se envía un POST al Backend, que lee el estado actual del usuario y lo **alterna**: si estaba 'Activo' lo pasa a 'Inactivo' (deshabilitar); si estaba 'Inactivo' lo pasa a 'Activo' (habilitar). La grilla se recarga y la fila **permanece visible** con su nuevo estado, permitiendo revertir la acción en cualquier momento.

---

## Lógica Transaccional y de Seguridad (Backend)

1. **Uso de `isset()` (criterio: ¿hay una petición al controlador?):**
   - **Regla:** `isset()` es el guardián del controlador. En MVC estricto **una interfaz no abre otra por sí sola**: la navegación la decide el controlador. Un botón lleva `isset()` cuando su acción implica **(a) transición a otra interfaz**, (b) lectura de datos (`SELECT` de precarga), (c) DML, o (d) escritura de sesión. **Única excepción:** la búsqueda/filtro que solo repinta la MISMA vista.
   - `Buscar`: **NO** lleva `isset()`. Consulta la BD pero solo recarga su propia grilla; no transiciona ni abre otra interfaz. Es la excepción explícita.
   - Abrir `Nuevo Usuario`: **SÍ** lleva `isset()`. Abrir el formulario es **transicionar a otra interfaz**, y esa transición pasa por el controlador — aunque el formulario salga vacío. Modelarlo como "modal JS sin `isset`" sería un patrón SPA que contradice el principio MVC "una vista no habla con otra".
   - Abrir `Editar Usuario`: **SÍ** lleva `isset()`. Es transición a formulario **más** un `SELECT` de precarga del usuario y sus roles. Doble motivo para el guardián.
   - Ambos espejan a `UC06`, donde **todos** los botones que abren otra interfaz (`[Completar Reserva Existente]`, `[Iniciar Nueva Venta]`) llevan `isset()`.
   - Las operaciones DML (`Guardar Nuevo`, `Guardar Cambios`, `Cambiar Estado`) **SÍ** llevan validación `isset()` estricta en el Backend antes de procesar el POST.
2. **Casos Negativos Cubiertos:**
   - Buscar con campo vacío → Error.
   - Intentar guardar usuario con campos en blanco → Error.
   - Guardar usuario sin seleccionar ningún checkbox de rol → Error.
   - Registrar un **DNI** que ya existe en la BD → Error (el DNI es único e identifica al usuario).
3. **Gestión de Relación Muchos a Muchos (Roles):**
   - El sistema soporta múltiples roles por usuario. Para las ediciones, el Backend aplica el patrón estándar: **Borrar todo y Re-insertar**. Al editar, primero se ejecuta `DELETE FROM Usuario_Rol WHERE id_usu = ?` y luego se insertan los nuevos checkboxes seleccionados. Esto evita lógica compleja de comparaciones "cuál rol se quitó y cuál se agregó".
4. **Identidad por DNI:** El **DNI (`documento`) es el identificador del usuario**: único e inmutable. La unicidad se valida al crear (`SELECT ... WHERE documento = ?`); al editar, el DNI viaja sólo como referencia (`WHERE id_usu = ?`) y **nunca** se incluye en el `SET` del `UPDATE`. El `username` es una credencial de acceso editable, no la identidad, por lo que no se valida como clave de unicidad del registro.
5. **Seguridad de Contraseñas:** 
   - Al crear, se hashea obligatoriamente. Al editar, el controlador solo ejecuta el `UPDATE password` si detecta que el campo no vino vacío desde el formulario.
6. **Borrado Lógico Bidireccional:** Nunca se ejecuta un `DELETE FROM Usuario`. El botón `[Cambiar Estado]` lee el `estado` actual y lo **alterna** con un `UPDATE` ('Activo' ↔ 'Inactivo'). La grilla lista **todos** los usuarios (sin `WHERE estado`), por lo que un usuario deshabilitado sigue siendo visible y se puede reactivar. Esto cumple RF-025 (listar todos con su estado) y RF-028 (cambiar estado en ambos sentidos).
