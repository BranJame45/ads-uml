---
tipo: entregable
entregable: Flujo de Caso de Uso y Arquitectura de Vista
caso_de_uso: UC01. Autenticar usuario
estado: finalizado (Arquitectura Estricta)
fecha: 2026-11-23
---
# UC01: Autenticar Usuario (Login)

## Arquitectura de la Interfaz (Visión UX/UI)

Caso de uso transversal de acceso al sistema. Actor: **Usuario**. Implementa un flujo lineal de autenticación en una única interfaz de entrada (`<Iniciar Sesión>`) que, tras validar credenciales, estado y rol, redirige al `<Panel Principal>` correspondiente.

### Interfaz 1: `<Iniciar Sesión>`
- **Campos de entrada:**
  - `txtUsername` (Obligatorio)
  - `txtPassword` (Obligatorio, enmascarado)
- **Acciones:**
  - Botón primario **`[Ingresar]`** (`btnIngresar`).
  - Enlace secundario `¿Olvidaste tu contraseña?` (dispara `UC03. Recuperar clave`, fuera del alcance de este CUS).

### Interfaz 2: `<Panel Principal>` (Destino tras autenticación exitosa)
- Renderiza las opciones del sistema habilitadas según el rol obtenido de la BD (`Administrador`, `Vendedor` o `Encargado de Almacén`).

### Comportamiento del Flujo (Interacción Paso a Paso)
1. **Ingreso al sistema:** El Usuario accede a la pantalla `<Iniciar Sesión>`, ingresa su `username` y `password`, y presiona **`[Ingresar]`**.
2. **Validación de petición y campos:** El Backend verifica `isset(btnIngresar)` y valida que ni `txtUsername` ni `txtPassword` estén vacíos (`RF-002`). Si falta alguno → `<Panel de Error>`.
3. **Verificación de existencia (`RF-003`):** Consulta en `Usuario` por `username`. Si no existe → `<Panel de Error>` *"El usuario ingresado no existe"* (`RF-008`).
4. **Verificación de contraseña (`RF-004`):** Compara la contraseña ingresada contra la almacenada. Si no coincide → `<Panel de Error>` *"Contraseña incorrecta"* (`RF-008`).
5. **Verificación de estado (`RF-005`):** Verifica que `estado == 'Activo'`. Si está inactivo → `<Panel de Error>` *"El usuario se encuentra inactivo"* (`RF-008`).
6. **Carga de rol y redirección (`RF-006`, `RF-007`):** Consulta los roles asignados en `Usuario_Rol` + `Rol`, carga los datos en sesión y redirige al `<Panel Principal>` correspondiente al rol.

---

## Lógica Transaccional y de Seguridad (Backend)

1. **Uso de `isset()`:** `btnIngresar` lleva `isset(btnIngresar)` obligatorio (transición de interfaz + consulta a BD).
2. **Mensajes de error específicos (`RF-008`):** Cada validación fallida informa exactamente la causa (campos vacíos, usuario inexistente, contraseña incorrecta o usuario inactivo).
3. **Consultas parametrizadas:** La búsqueda por `username` y la consulta de roles usan parámetros `?` para prevenir inyección SQL.

---

## Trazabilidad de Requerimientos
| RF | Cubierto por |
| :--- | :--- |
| RF-001 Formulario con usuario y contraseña | Vista `<Iniciar Sesión>` (`txtUsername`, `txtPassword`, `btnIngresar`) |
| RF-002 Validar campos no vacíos | Rombo `¿Campos vacíos?` |
| RF-003 Verificar si usuario existe en BD | `SELECT ... FROM Usuario WHERE username = ?` + `¿Existe usuario?` |
| RF-004 Verificar contraseña correcta | Rombo `¿Contraseña correcta?` |
| RF-005 Verificar estado activo/inactivo | Rombo `¿Estado == 'Activo'?` |
| RF-006 Consultar rol asignado en BD | `SELECT ... FROM Usuario_Rol INNER JOIN Rol ...` |
| RF-007 Redirigir al panel principal según rol | `:Redirigir al usuario al <Panel Principal> correspondiente a su rol;` |
| RF-008 Mensaje de error específico según falla | Paneles `<Error>` diferenciados por cada validación fallida |
