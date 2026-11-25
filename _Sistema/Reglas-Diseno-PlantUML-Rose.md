---
tipo: sistema
proyecto: ADS-2026 II
estado: vigente
fecha: 2026-11-25
referencia: Manual de Diseño (PlantUML + Rational Rose Design Model)
base_referencia: TrabajoOficialZegarra.mdl (Design Model) + Reglas de Análisis CARTOLIMP
ejemplo_validado: UC10. Gestionar productos de limpieza
---
# Manual de Diagramas de Diseño (DS y DO de Diseño) — CARTOLIMP

Este manual define las reglas oficiales para construir los **Diagramas de Secuencia de Diseño (`DS. Diseno.txt`)** y **Diagramas de Objetos/Clases de Diseño (`DO. Diseno.txt`)**, manteniendo **trazabilidad 1:1** con el `D. Actividades.txt` y `DS. Secuencia.txt` (Análisis) aprobados, e incorporando la arquitectura real de programación MVC (`shared`, `index`, `get`, `control`, `form*`, `Modelos` y `conexion`).

---

## 1. Arquitectura de Paquetes y Clases de Diseño

### 1.1 Paquete `shared` (Clases transversales compartidas)
1. **`pantalla`** (clase base de vistas):
   - Métodos:
     - `+ cabeceraShow(titulo: String): void`
     - `+ menuShow(): void`
     - `+ pieShow(): void`
   - **Herencia de vistas (`form*`):** Todas las pantallas funcionales (`formGestionar*`, `formAgregar*`, `formEditar*`, `formPanelPrincipal`) heredan de `pantalla` (`--|> pantalla`) e invocan `cabeceraShow(...)`, `menuShow()` y `pieShow()` al mostrarse.
   - **Herencia de diálogos (`ViewMensajeSistema`, `mensajeConfirmacion`):** Heredan de `pantalla`, pero al mostrarse invocan **únicamente `cabeceraShow(...)` y `pieShow()`** (sin `menuShow()`).
2. **`ViewMensajeSistema`** (`--|> pantalla`):
   - Atributos: `lblTitulo : JLabel`, `lblDescripcion : JLabel`, `btnAceptar : JButton`.
   - Método: `+ viewMensajeSistemaShow(tipo: String, descripcion: String, link: String): void`.
   - **Regla de parámetros concretos:** En el DS de Diseño se pasan los **valores literales exactos**, por ejemplo: `viewMensajeSistemaShow("Error", "Acceso no permitido", "listado")` o `viewMensajeSistemaShow("Éxito", "Producto registrado correctamente", "listado")`.
3. **`mensajeConfirmacion`** (`--|> pantalla`):
   - Atributos: `lblPregunta : JLabel`, `btnSi : JButton`, `btnNo : JButton`.
   - Método: `+ mensajeConfirmacionShow(descripcion: String): void`.
4. **`libreriaPDF`** (cuando el CU genera PDF): `instanciar(): libreriaPDF`, `renderizarTablaHTML(lista: array, columnas: array, titulo: String): String`, `generarYDescargarPDF(html: String): void`.

### 1.2 Paquete del Módulo (Control, Enrutamiento y Vistas concretas)
En Diseño, el controlador único de Análisis (`CC_*Controller`) y las fronteras genéricas se desdoblan en componentes de código:
1. **`index<CU>`** (ej. `indexGestionarProducto`):
   - Punto de entrada inicial del CU (recibe `Activar` del Actor) y delega al controlador `mostrarListado()` (o vista inicial).
2. **`get<Entidad>`** (ej. `getProducto`):
   - Receptor de todos los `POST` enviados desde las vistas (`formGestionar*`, `formAgregar*`, `formEditar*`, `mensajeConfirmacion`).
   - Responsabilidades:
     - Valida `isset(btn...)` (`validarBoton`). Si falla → llama directo a `ViewMensajeSistema.viewMensajeSistemaShow("Error", "Acceso no permitido", ...)`.
     - Valida campos vacíos / formato básico (`validarCampoBusqueda`, `validarCampos`). Si falla → llama directo a `ViewMensajeSistema.viewMensajeSistemaShow("Error", "...", ...)`.
     - Si pasa las validaciones de entrada → delega la operación al controlador (`controlGestionar*`).
3. **`control<CU>`** (ej. `controlGestionarProducto`):
   - Orquesta la lógica de negocio y el acceso a los Modelos (`Modelos::*`).
   - Evalúa reglas de negocio en BD (duplicados, existencia activa, stock) y llama a `ViewMensajeSistema.viewMensajeSistemaShow(...)` ante error de negocio o éxito.
   - Invoca `form*Show(...)` o `mensajeConfirmacionShow(...)` para renderizar la vista correspondiente.
   - Gestiona `cancelarOperacion(btnCancelar: String): void` redirigiendo a `formGestionar*Show(...)` (**MVC estricto: nunca hay comunicación directa Vista → Vista**).
4. **Vistas concretas (`form*`):**
   - Se separa la vista de agregar y la de editar en clases distintas cuando corresponda (`formGestionarProducto`, `formAgregarProducto`, `formEditarProducto`).
   - Sus atributos usan **tipos de control UI reales**: `input[text]`, `input[number]`, `textarea`, `select`, `checkbox[]`, `button`, `table`, `label`.

### 1.3 Paquete `Modelos` (Persistencia y `conexion`)
1. **`conexion`** (clase base de acceso a BD):
   - Métodos: `conectarBD(): void`, `desconectarBD(): void`, `beginTransaction(): void`, `commit(): void`, `rollBack(): void`.
2. **Modelos de tabla (`producto`, `categoria`, `unidad_medida`, `usuario`, etc. — sin prefijo `CE_`):**
   - **Todos los modelos heredan de `conexion`** (`--|> conexion`).
   - En el DS de Diseño, cada llamada desde `control*` a un modelo (ej. `obtenerProductosActivos()`) implica que el modelo ejecuta:
     1. `modelo -> conexion : conectarBD()`
     2. *(ejecuta la consulta/DML SQL)*
     3. `modelo -> conexion : desconectarBD()`
     4. `modelo --> control* : <retorno>` *(flecha punteada de retorno)*
3. **Transacciones (`beginTransaction`, `commit`, `rollBack`) vs Paridad:**
   - Aunque `conexion` declara `beginTransaction()`, `commit()` y `rollBack()`, **NO se agregan como ramas nuevas (`break`/`alt`) en el DS de Diseño** para mantener **100% la paridad y trazabilidad con `D. Actividades.txt` y `DS. Secuencia.txt` aprobados** (donde no existen esas ramas). La atomicidad transaccional queda encapsulada dentro de los métodos del modelo/controlador, mostrándose en el DS únicamente `conectarBD()` y `desconectarBD()`.

---

## 2. Reglas de Trazabilidad y Estilo (DS y DO de Diseño)

1. **Orden estricto de columnas MVC (heredado de DS Análisis):**
   - **Izquierda:** `Actor` → todas las **Interfaces / Vistas** (`form*`, `mensajeConfirmacion`, `ViewMensajeSistema`, `pantalla`).
   - **Medio:** **Controladores y Enrutadores** (`index*`, `get*`, `control*` — `index*` es controlador/enrutador, **no** interfaz).
   - **Derecha:** **Modelos / Entidades** (`producto`, `categoria`, etc.) y `conexion`.
2. **Trazabilidad 1:1 con Análisis (`DA` y `DS` Análisis):**
   - Mismas ramas (`group BUSCAR`, `AGREGAR`, `EDITAR`, `DAR DE BAJA`), mismas validaciones y mismos mensajes que `D. Actividades.txt` y `DS. Secuencia.txt`.
   - **Sin `validarRol*` en el arranque** (si no está en el `DA` aprobado, no se agrega en Diseño).
2. **Actor aislado:**
   - El Actor solo tiene **1 interacción inicial** (`actor -> index* : Activar`) y **nada retorna al Actor** en todo el diagrama.
3. **Prioridad de rama negativa y Cancelar:**
   - En `alt / else`, **el camino negativo (`btnCancelar` / `btnNo`) siempre va 1º**, y en `get*` / `control*` toda validación fallida usa `break si ...` llamando a `ViewMensajeSistema.viewMensajeSistemaShow("Error", ..., ...)`.
   - En Cancelar, se respeta MVC estricto: `form* -> get* : POST(btnCancelar)` → `get* -> control* : cancelarOperacion()` → `control* -> formGestionar* : formGestionar*Show(...)`.
4. **Firmas completas con parámetros y retorno:**
   - En el DS de Diseño, los métodos muestran tipos y valores concretos (`viewMensajeSistemaShow("Error", "...", "...")`, `cabeceraShow("Nuevo Producto")`).
   - En el DO de Diseño (`DO. Diseno.txt`), cada clase declara sus **atributos tipados** (`input[text]`, `JLabel`, `int`, `String`, etc.), sus **métodos base con parámetros tipados y retorno**, y sus relaciones de **herencia (`--|>`)** y **asociación/dependencia**.

---

## 3. Checklist de Diseño (antes de cerrar un CU)
- [ ] Trazabilidad 1:1 con `D. Actividades.txt` y `DS. Secuencia.txt` (mismas ramas, validaciones y SQL).
- [ ] `index*`, `get*`, `control*`, vistas `form*` separadas y modelos sin prefijo `CE_`.
- [ ] Herencia de `pantalla` (`cabeceraShow`, `menuShow`, `pieShow` en `form*`; solo `cabeceraShow` y `pieShow` en `ViewMensajeSistema` y `mensajeConfirmacion`).
- [ ] Herencia de `conexion` en todos los modelos (`conectarBD()` y `desconectarBD()` en cada acceso a BD del DS).
- [ ] `ViewMensajeSistema.viewMensajeSistemaShow("Tipo", "Descripción", "link")` con valores literales concretos.
- [ ] Actor aislado (solo `Activar` al inicio) y Cancelar en 1er lugar respetando MVC.
