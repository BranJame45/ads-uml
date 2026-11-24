---
tipo: sistema
proyecto: ADS-2026 II
estado: vigente
fecha: 2026-11-18
referencia: Manual del Diagrama de Objetos de Análisis / VOPC (PlantUML, robustez BCE)
deriva_de: Reglas-Diagramas-Secuencia.md (autoridad de métodos) + Reglas-Diagramas-Actividades.md (autoridad de datos)
ejemplos_validados: UC10. Gestionar productos de limpieza, UC06. Registrar venta
---
# Manual del Diagrama de Objetos de Análisis (VOPC) — CARTOLIMP

El "diagrama de objetos de análisis" (DO) es un **diagrama de clases de participación (VOPC)**: muestra las clases BCE que colaboran en el CU, con sus **atributos** y **métodos**, y las **relaciones numeradas** que trazan la interacción. Se construye **a partir del diagrama de secuencia** del mismo CU.

> 🔴 **REGLA DE PROCESO (obligatoria):** el DO se construye **revisando continuamente el Diagrama de Actividades (DA) y el Diagrama de Secuencia (DS)** del mismo CU. El DA aporta las tablas/columnas (atributos de entidad) y las vistas (fronteras); el DS aporta los métodos y su asignación. No se construye un DO "de memoria": se va contrastando cada clase, atributo, método y relación contra el DA y el DS. Si no coinciden, hay un error que corregir en alguno.

> 🔁 **PARIDAD DE FLUJO (los 3 diagramas deben cuadrar entre sí).** El mismo CU tiene DA + DS + DO con correspondencia 1:1 en ambos sentidos:
> - **DA ↔ DO:** cada tabla de los SQL = una entidad (columnas = atributos con tipo); cada vista/panel = una frontera (botones/campos = atributos); cada operación de BD = una relación control→entidad con su método; cada `case`/botón = una rama numerada.
> - **DO ↔ DS:** cada lifeline = una clase; cada mensaje que recibe un objeto = un método de esa clase; cada camino de mensaje = una relación numerada (mismo orden).
> - **DA ↔ DS:** cada `case` = un `group`; cada `isset`/validación/SELECT/INSERT/UPDATE/panel = su mensaje/`break`/`CI_Mensaje*` correspondiente, en el mismo orden.
> Si algo aparece en uno y no en los otros dos, **falta** o **sobra**: se corrige hasta lograr la paridad.

> **REGLA DE ORO (asignación de métodos):** cada método se coloca en la clase que **RECIBE ese mensaje** en el diagrama de **secuencia**, con el **nombre exacto**. El `Controlador` **NO** acumula los métodos de las entidades. Error típico a evitar: poner `consultarProveedores/insertarProveedor/eliminarProveedor` en el controlador cuando el DS muestra `obtenerProveedoresActivos/registrarProveedor/desactivarProveedor` recibidos por la **Entidad**.

> **TRAZABILIDAD TRIPLE:**
> - **Métodos ⇐ Secuencia:** cada mensaje del DS = un método de la clase que lo recibe (mismo nombre).
> - **Atributos de Entidad ⇐ Actividades:** las columnas de los `note right: SELECT/INSERT/UPDATE` + sus tipos SQL.
> - **Atributos de Frontera ⇐ Flujo/Actividades:** los botones y campos de cada vista.
> - **Relaciones numeradas ⇐ Secuencia:** el orden (1,2,3…; ramas 5a/6a, 5b/6b, 5c…) replica la secuencia de mensajes = los `case` del switch de actividades.

---

## O1 — Cabecera y skinparams (fijos)
```
@startuml
skinparam ClassBackgroundColor #FFFFE0
skinparam ClassBorderColor #8B0000
skinparam ArrowColor #8B0000
skinparam ClassStereotypeFontColor #8B0000
skinparam ClassHeaderBackgroundColor #F5E6E6
skinparam shadowing false
skinparam DefaultFontName Arial
```

## O2 — Título
`title DO_<Nombre del Caso de Uso> (UCXX)` — prefijo **`DO_`** + nombre del CU + código correcto.

## O3 — Clases y estereotipos (EN ESPAÑOL)
Se usa `class` (no `object`). El **nombre visible SÍ lleva prefijo** `CI_/CC_/CE_` (paridad exacta con las lifelines del DS). Solo el actor no lleva prefijo. Aliases: `a_` actor, `b_` frontera, `ctrl` controlador, `e_` entidad.
```
class "<Actor real>"                   as a_xxx       <<Agente>>
class "CI_Ventana<Entidad>"            as b_ventana   <<Frontera>>
class "CI_Formulario<Entidad>"         as b_form      <<Frontera>>
class "CI_MensajeConfirmacion"         as b_confirm   <<Frontera>>
class "CI_MensajeSistema"              as b_msg       <<Frontera>>
class "CC_<NombreCU>Controller"        as ctrl        <<Controlador>>
class "CE_<tabla>"                     as e_xxx       <<Entidad>>
class "CE_<tabla_puente>"              as e_rel       <<Entidad>>
```
Estereotipos: `<<Agente>>` (actor), `<<Frontera>>` (boundary, **sin `CI_PanelPrincipal`**), `<<Controlador>>` (control), `<<Entidad>>` (entity).

## O4 — Miembros de cada clase
- **`<<Agente>>` (actor):** sin miembros.
- **`<<Frontera>>`:** 
  - **Atributos `+`** = controles de UI (botones/campos/listas) que maneja esa vista, tomados del `Flujo-UCXX.md`/actividades. Ej.: `+ btnAgregar`, `+ txtRUC`, `+ chkCategorias[ ]`, `+ listadoRegistros`.
    - En `CI_MensajeConfirmacion`: `+ mensaje : String`, `+ btnAceptar`, `+ btnCancelar`.
    - En `CI_MensajeSistema`: `+ mensaje : String`, `+ descripcion : String`, `+ link : String`.
  - **Métodos `+`** = **TODOS los mensajes que la frontera recibe en el DS**, con nombre exacto:
    - `+ CI_<Vista>Show()` — el mensaje que le dirige el `ctrl` para mostrarla (conserva el prefijo `CI_`).
    - `+ click<Botón>()` — un método por cada click que el actor le dirige (p. ej. `clickBuscar`, `clickNuevoProducto`, `clickEditarProducto`, `clickDarBajaProducto`, `clickCancelar`, `clickAceptar`).
    - `+ ingresarDatos()` / `+ modificarDatos()` — en formularios de Nuevo / Editar.
  - Regla: si un mensaje aparece en el DS como `X -> b_<y> : metodo(...)`, el método va dentro de la frontera `b_<y>` con el nombre **idéntico**. Si no está declarado y se usa en una relación numerada, hay inconsistencia.
- **`<<Controlador>>`:** métodos `+` = **solo** los que el DS dirige al control: las validaciones (`validarBtn...()`, `validarBoton()`, `validarRol...()`, `validarCampos...()`, `validarCampoBusqueda()`) y métodos de coordinación directa sin `isset` según R4 (ej. `buscarProductos()`). **No** lleva métodos de acceso a datos (`obtener/verificar/registrar/actualizar/desactivar`).
- **`<<Entidad>>`:** 
  - Atributos `-` con **tipo SQL**, de las columnas de los `note right` del diagrama de actividades: `- ruc : VARCHAR(11)`, `- id_categoria : INT (FK)`, `- estado : VARCHAR(10)`, etc. Las tablas maestras (ej. `CE_categoria`, `CE_unidad_medida`) y tablas puente N:M también se modelan como `<<Entidad>>` con sus columnas y métodos.
  - Métodos `+` = los mensajes que el DS dirige a ESA entidad, con nombre exacto: `obtener...()`, `verificar...()`, `registrar...()`, `actualizar...()`, `desactivar...()`. La entity puente N:M lleva sus `registrar<Rel>()/actualizar<Rel>()/obtener<Rel>Asignadas()`.

## O5 — Asignación de métodos (el corazón del diagrama)
Recorrer el diagrama de secuencia y, por cada mensaje `X -> Y : metodo(...)`, **agregar `metodo()` a la clase Y** (la que lo recibe):
- `b_ -> ctrl : validarBtn...()` ⇒ método del **Controlador**.
- `ctrl -> ctrl : validarBoton()/validarRol()/validarCampos()` ⇒ método del **Controlador** (auto-llamadas).
- `ctrl -> e_x : obtener/verificar/registrar/actualizar/desactivar...()` ⇒ método de la **Entidad** `e_x`.
- `ctrl -> b_x : CI_<Vista>Show()` ⇒ método de la **Frontera** `b_x` (conserva el prefijo `CI_`).
Nunca renombrar (no `consultar` por `obtener`, no `insertar` por `registrar`, no `eliminar` por `desactivar`): se copia el nombre del DS.

## O6 — Relaciones NUMERADAS (trazan la secuencia)
Flechas dirigidas `-->` con etiqueta `N. <acción>`, numeradas en el **orden del diagrama de secuencia**. El flujo inicial va `1,2,3,4…`; cada rama de operación usa sufijo de letra para no colisionar: Buscar `5a,6a,7a…`, Agregar `5b,6b,7b…`, Editar `5c,6c,7c…`, Eliminar `5d,6d…`.

**Ramas `alt` del DS → sufijo `-cancel` en el DO (SIEMPRE en 1er lugar, priorizando el camino negativo).** Cuando el DS modela `alt / else` para el Cancelar (ver S6 del manual de secuencia), el DO coloca **primero** la rama de cancelación (`8b-cancel. [alt-Cancelar] clickCancelar(btnCancelar) → cierra modal`) y **segundo** la rama positiva (`8b. [else-Guardar] ingresarDatos(..., btnGuardar)`). El Cancelar es una interacción **actor → frontera** (no pasa por ctrl porque no requiere validación). En el arranque del CU (cuando inicia por acción de un botón), la relación `1.` va **directamente del Agente al Controlador** (sin `CI_PanelPrincipal`):
```
a_actor     --> ctrl        : 1. validarBtnX(btnX)
ctrl        --> e_entidad   : 2. obtener<Entidad>Activos()
ctrl        --> b_ventana   : 3. CI_Ventana<Entidad>Show(listado)
b_ventana   --> ctrl        : 4a. [BUSCAR] buscar<Entidad>(textoBusqueda)
ctrl        --> e_entidad   : 5a. obtener<Entidad>Filtrados(textoBusqueda)
ctrl        --> b_ventana   : 6a. CI_Ventana<Entidad>Show(listadoFiltrado)
b_ventana   --> ctrl        : 4b. [AGREGAR] Envía btnNuevo<Entidad>
ctrl        --> b_form      : 5b. CI_Formulario<Entidad>Show(...)
a_actor     --> b_form      : 6b-cancel. [alt-Cancelar] clickCancelar(btnCancelar) → cierra modal
b_form      --> ctrl        : 6b. [else-Guardar] ingresarDatos(..., btnGuardar)
...
ctrl        --> b_msg       : * CI_MensajeSistemaShow() en éxito o validación fallida
```
Reglas de robustez para las flechas: en el arranque `actor→control` (`1. validarBtn<CUS>`); luego `actor→frontera`, `frontera→control`, `control→entidad`, `control→frontera`. **Prohibido** `actor→entidad`, `frontera→entidad`. El mensaje global al `CI_MensajeSistema` se marca con `*`. En ramas de éxito tras DML, incluir también la re-consulta `obtener<Entidad>Activos()` y `CI_Ventana<Entidad>Show(listado)` en el mismo orden del DS.

## O7 — Tabla de trazabilidad (verificación)
| Elemento del DO | Fuente |
| :--- | :--- |
| Método del `<<Controlador>>` | mensaje que el DS dirige al control (`validar*` y coordinación sin `isset` como `buscar*`) |
| Método de una `<<Entidad>>` | mensaje que el DS dirige a esa entidad (`obtener/verificar/registrar/actualizar/desactivar`) |
| Método de una `<<Frontera>>` | `CI_<Vista>Show()`, `click*()`, `ingresarDatos()`/`modificarDatos()` que recibe en el DS |
| Atributo de una `<<Entidad>>` (con tipo SQL) | columna en los `note right` SQL del diagrama de actividades |
| Atributo de una `<<Frontera>>` | botón/campo de esa vista en el `Flujo-UCXX.md` |
| Clase `<<Entidad>>` | tabla de los SQL de actividades (incluida la puente N:M) |
| Clase `<<Frontera>>` | vista/panel (`Mostrar ventana/modal/panel`) de actividades |
| Relación numerada N | el mensaje Nº del DS = paso del `case` de actividades |

## O8 — Checklist antes de cerrar un DO
- [ ] Skinparams `Class*` + `ClassHeaderBackgroundColor #F5E6E6`; título `DO_<Nombre> (UCXX)` con código correcto.
- [ ] Actor real del CU; estereotipos en español `<<Agente>>/<<Frontera>>/<<Controlador>>/<<Entidad>>`; nombres CON prefijo `CI_/CC_/CE_` (paridad exacta con el DS; solo el actor va sin prefijo).
- [ ] **Cada método está en la clase que lo RECIBE en el diagrama de secuencia, con el nombre idéntico.** El controlador solo tiene `validar*` (y coordinación sin `isset` como `buscar*`); las entidades tienen sus `obtener/verificar/registrar/actualizar/desactivar`; las fronteras tienen `CI_*Show()` + `click*()` + `ingresarDatos()/modificarDatos()`.
- [ ] Atributos de entidad con tipo SQL, tomados de las columnas de los `note` de actividades.
- [ ] Atributos de frontera = botones/campos de cada vista.
- [ ] Relaciones dirigidas y **numeradas** según el orden del DS; robustez respetada (arranque directo `1. actor→control`, luego `actor→frontera`, `frontera→control`, `control→entidad`, `control→frontera`; prohibido `actor→entidad` y `frontera→entidad`).
- [ ] Ramas de cancelación modeladas con sufijo `-cancel` en 1er lugar (`actor→frontera`, priorizando el camino negativo) antes de la rama positiva `else`.
- [ ] Mismos objetos/clases que las lifelines del DS; todo trazable a actividades.
- [ ] **Diagrama de Entidades Consolidado (`02-Entregables/D. Entidades.txt`) actualizado** con las entidades `CE_*`, atributos SQL, relaciones FK y métodos nuevos o modificados de este CU.

---

## O9 — Actualización obligatoria del Diagrama de Entidades Consolidado (`D. Entidades.txt` / `Trabajo aplicativo.mdl`)
Cada vez que se crea o modifica un `DO. Objetos.txt` (o un `DA`/`DS` que introduzca o altere tablas, columnas, relaciones o métodos de entidad), es **obligatorio** actualizar [02-Entregables/D. Entidades.txt](file:///c:/Users/islac/OneDrive/Desktop/ADS-2026%20II/02-Entregables/D.%20Entidades.txt) manteniendo paridad con el modelo maestro en [Trabajo aplicativo.mdl](file:///c:/Users/islac/OneDrive/Desktop/ADS-2026%20II/02-Entregables/Trabajo%20aplicativo.mdl):
1. **Entidades nuevas:** agregar la clase `CE_<tabla> <<Entidad>>` en el paquete de módulo correspondiente (`Seguridad`, `Ventas`, `Inventario y Producción`).
2. **Atributos acumulados:** fusionar cualquier columna SQL nueva (con su tipo y marca `PK`/`FK`) sin eliminar atributos definidos por otros Casos de Uso.
3. **Métodos acumulados:** agregar los nuevos métodos `obtener/verificar/registrar/actualizar/desactivar/...()` en la `CE_*` correspondiente manteniendo los existentes de otros CUs.
4. **Relaciones FK:** trazar o actualizar las asociaciones/composiciones (`1 -- 0..*`, `1 *-- 1..*`) entre las entidades vinculadas por clave foránea.
