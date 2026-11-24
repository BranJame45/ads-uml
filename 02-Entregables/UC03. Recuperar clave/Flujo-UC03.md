---
tipo: entregable
entregable: Flujo de Caso de Uso y Arquitectura de Vista
caso_de_uso: UC03. Recuperar clave
estado: finalizado (Arquitectura Estricta)
fecha: 2026-11-24
---
# UC03: Recuperar Clave (por Pregunta Secreta)

## Arquitectura de la Interfaz (Visión UX/UI)

Caso de uso transversal de seguridad disparado desde el enlace `¿Olvidaste tu contraseña?` de `<Iniciar Sesión>`. Actor: **Usuario**. Cada usuario registrado cuenta con una `pregunta_secreta` y su respectiva `respuesta_secreta` en la tabla `Usuario`, lo que permite restablecer la contraseña sin APIs externas ni envío de correos.

### Interfaz 1: `<Formulario Buscar Usuario>`
- **Campos:** `txtUsername` (Obligatorio).
- **Acciones:** **`[Continuar]`** (`btnBuscarUsuario`) y **`[Cancelar]`** (`btnCancelar`).

### Interfaz 2: `<Formulario Pregunta Secreta>`
- **Visualización:** `lblPreguntaSecreta` (sólo lectura, cargada desde la BD para ese usuario).
- **Campos:** `txtRespuestaSecreta` (Obligatorio).
- **Acciones:** **`[Verificar Respuesta]`** (`btnVerificarRespuesta`) y **`[Cancelar]`** (`btnCancelar`).

### Interfaz 3: `<Formulario Nueva Clave>`
- **Campos:** `txtClaveNueva` (Obligatorio, enmascarado, mínimo 8 caracteres).
- **Acciones:** **`[Guardar Clave]`** (`btnGuardarClave`) y **`[Cancelar]`** (`btnCancelar`).

### Comportamiento del Flujo (Interacción Paso a Paso)
1. **Identificación del Usuario:** El Usuario hace clic en `"¿Olvidaste tu contraseña?"`, ingresa su `username` y presiona `[Continuar]`. El sistema valida `isset(btnBuscarUsuario)`, verifica que el usuario exista y esté `Activo`, y obtiene su `pregunta_secreta`.
2. **Validación de Pregunta Secreta:** En `<Formulario Pregunta Secreta>`, se muestra la pregunta asignada al usuario. El usuario ingresa su `respuesta_secreta` y presiona `[Verificar Respuesta]`. El sistema valida `isset(btnVerificarRespuesta)` y verifica en `Usuario` que la respuesta coincida.
3. **Nueva Contraseña:** En `<Formulario Nueva Clave>`, ingresa la nueva contraseña (`>= 8` caracteres) y presiona `[Guardar Clave]`. El sistema encripta la contraseña y ejecuta `UPDATE Usuario SET password = ? WHERE id_usu = ?`.

---

## Lógica Transaccional y de Seguridad (Backend)

1. **Uso de `isset()`:** `btnRecuperarClave`, `btnBuscarUsuario`, `btnVerificarRespuesta` y `btnGuardarClave` llevan `isset()` obligatorio.
2. **Validación de estado activo:** Solo usuarios con `estado = 'Activo'` pueden recuperar su contraseña.
3. **Casos Negativos Cubiertos:**
   - Username vacío, inexistente o usuario inactivo → Error.
   - Respuesta secreta vacía o incorrecta → Error.
   - Nueva contraseña vacía o menor a 8 caracteres → Error.

---

## Trazabilidad de Requerimientos
| RF | Cubierto por |
| :--- | :--- |
| RF-014 Formulario para ingresar username | `<Formulario Buscar Usuario>` (`txtUsername`) |
| RF-015 Verificar si username existe en BD | `SELECT id_usu, username, pregunta_secreta, estado FROM Usuario WHERE username = ?` |
| RF-016 / RF-017 Mostrar pregunta secreta del usuario | `lblPreguntaSecreta` en `<Formulario Pregunta Secreta>` |
| RF-018 Verificar identidad mediante respuesta secreta | `SELECT id_usu FROM Usuario WHERE id_usu = ? AND respuesta_secreta = ?` |
| RF-019 Mostrar formulario para nueva contraseña | `<Formulario Nueva Clave>` (`txtClaveNueva`) |
| RF-020 Guardar nueva contraseña encriptada en BD | `UPDATE Usuario SET password = ? WHERE id_usu = ?` |
