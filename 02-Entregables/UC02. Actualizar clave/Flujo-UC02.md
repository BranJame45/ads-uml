---
tipo: entregable
entregable: Flujo de Caso de Uso y Arquitectura de Vista
caso_de_uso: UC02. Actualizar clave
estado: finalizado (Arquitectura Estricta)
fecha: 2026-11-23
---
# UC02: Actualizar Clave

## Arquitectura de la Interfaz (Visión UX/UI)

Caso de uso transversal de seguridad. Actor: **Usuario** (autenticado en el sistema). Permite cambiar su propia contraseña desde una vista de formulario modal/independiente.

### Interfaz 1: `<Formulario Actualizar Clave>`
- **Campos de entrada:**
  - `txtClaveActual` (Obligatorio, enmascarado)
  - `txtClaveNueva` (Obligatorio, enmascarado, longitud mínima de 8 caracteres)
- **Pie del Formulario:**
  - **`[Guardar]`** (`btnGuardar`, primario)
  - **`[Cancelar]`** (`btnCancelar`, secundario)

### Comportamiento del Flujo (Interacción Paso a Paso)
1. **Apertura:** El Usuario presiona `"Actualizar Clave"` en su menú de perfil. El controlador valida `isset(btnActualizarClave)` y muestra `<Formulario Actualizar Clave>`.
2. **Cancelación:** Si presiona **`[Cancelar]`**, se cierra el formulario sin realizar cambios.
3. **Validación y Actualización:** Ingresa `txtClaveActual` y `txtClaveNueva` y presiona **`[Guardar]`**:
   - Valida `isset(btnGuardar)` y que los campos no estén vacíos.
   - Valida la longitud mínima de la nueva contraseña (`RF-011`).
   - Consulta la contraseña actual del usuario (`id_usu` de sesión) en `Usuario` y verifica que coincida (`RF-010`).
   - Encripta la nueva contraseña y ejecuta `UPDATE Usuario SET password = ? WHERE id_usu = ?` (`RF-012`).
   - Muestra `<Panel de Éxito>` *"Contraseña actualizada correctamente"* (`RF-013`).

---

## Lógica Transaccional y de Seguridad (Backend)

1. **Uso de `isset()`:** Tanto `btnActualizarClave` (transición al formulario) como `btnGuardar` (DML) llevan `isset()`. `btnCancelar` cierra el formulario sin ir al controlador.
2. **Encriptación:** La nueva contraseña se encripta antes de ejecutarse el `UPDATE`.
3. **Casos Negativos Cubiertos:**
   - Campos obligatorios vacíos → Error.
   - Nueva contraseña menor a la longitud mínima → Error.
   - Contraseña actual no coincide con la BD → Error.

---

## Trazabilidad de Requerimientos
| RF | Cubierto por |
| :--- | :--- |
| RF-009 Formulario con contraseña actual y nueva | `<Formulario Actualizar Clave>` (`txtClaveActual`, `txtClaveNueva`) |
| RF-010 Validar contraseña actual contra BD | `SELECT password FROM Usuario WHERE id_usu = ?` + `¿Contraseña actual correcta?` |
| RF-011 Validar longitud mínima de nueva clave | Rombo `¿Longitud mínima válida?` |
| RF-012 Guardar nueva contraseña encriptada en BD | `:Encriptar nueva contraseña;` + `UPDATE Usuario SET password = ? WHERE id_usu = ?` |
| RF-013 Mensaje de confirmación al actualizar | `<Panel de Éxito>` *"Contraseña actualizada correctamente"* |
