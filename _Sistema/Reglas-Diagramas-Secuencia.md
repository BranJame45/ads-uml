---
tipo: sistema
proyecto: ADS-2026 II
estado: vigente
fecha: 2026-11-18
referencia: Manual de construcción de Diagramas de Secuencia de Análisis (PlantUML, robustez BCE)
ejemplo_oficial: DS_Gestionar proveedores
ejemplos_validados: DS_Gestionar proveedores, UC10. Gestionar productos de limpieza, UC06. Registrar venta
---
# Manual de Construcción de Diagramas de Secuencia de Análisis — CARTOLIMP

Receta oficial para construir el diagrama de **secuencia de análisis** (DS) de cualquier CU, derivada del ejemplo oficial `DS_Gestionar proveedores`. Usa el modelo de **robustez BCE** (Boundary–Control–Entity). Complementa al manual de actividades: un mismo CU tiene su diagrama de actividades Y su diagrama de secuencia, y deben ser coherentes entre sí.

> 🔴 **REGLA DE PROCESO (obligatoria):** el DS **se construye revisando continuamente el Diagrama de Actividades (DA) del mismo CU**. El DA es la fuente de la verdad del comportamiento: cada `isset`, validación, SELECT/INSERT/UPDATE, panel y `case` del DA debe tener su correspondencia en el DS. No se construye un DS "de memoria" ni desde el `Flujo-UCXX.md` solo: se va contrastando mensaje por mensaje contra el DA. Si el DA y el DS no coinciden, uno de los dos está mal.

> 🔁 **PARIDAD DE FLUJO (los 3 diagramas deben cuadrar entre sí).** El mismo CU tiene DA + DS + DO, y deben tener correspondencia 1:1 en ambos sentidos:
> - **DA ↔ DS:** cada `case` del switch = un `group`; cada `isset(btn)` = `validarBoton()` + `break`; cada rombo de validación = validación + `break`; cada SELECT/INSERT/UPDATE = un mensaje a entidad; cada panel `<Error/Éxito/Aviso/Confirmación>` = `CI_MensajeSistema`/`CI_MensajeConfirmacion`. **El orden de los pasos coincide.**
> - **DO ↔ DS:** cada lifeline = una clase; cada mensaje que recibe un objeto = un método de esa clase; cada camino de mensaje = una relación numerada (mismo orden).
> - **DA ↔ DO:** cada tabla de los SQL = una entidad (columnas = atributos); cada vista/panel = una frontera (botones/campos = atributos); cada operación de BD = una relación control→entidad con su método; cada `case`/botón = una rama numerada.
> Si algo aparece en uno y no en los otros dos, **falta** o **sobra**: se corrige hasta lograr la paridad.

> **Regla de oro (robustez + excepción de arranque):** durante el flujo de vistas del CU, un **Actor solo habla con Boundary** (nunca con Entity). **Excepción única de arranque:** cuando el Caso de Uso empieza por la acción de un botón disparador, **NO se modela `CI_PanelPrincipal`** y el arranque va **directamente del Actor al Controlador** (`actor -> ctrl : validarBtn<CUS>(btn<CUS>: String): bool`).

---

## S1 — Cabecera y skinparams (fijos)
```
@startuml
!pragma teoz true
skinparam sequenceArrowColor #8B0000
skinparam sequenceLifeLineBorderColor #8B0000
skinparam sequenceParticipantBorderColor #8B0000
skinparam sequenceParticipantBackgroundColor #FFFFE0
skinparam sequenceActorBorderColor #8B0000
skinparam sequenceGroupBorderColor #8B0000
skinparam shadowing false
skinparam DefaultFontName Arial
```
- Misma paleta vino/crema que actividades (#8B0000 / #FFFFE0). **No** lleva `scale`.
- `!pragma teoz true` es obligatorio.

## S2 — Título
`title DS_<Nombre del Caso de Uso> (UCXX)` — prefijo **`DS_`**, nombre del CU, y el código del CU entre paréntesis. (En el ejemplo oficial decía `(UC06)` pero el CU real de proveedores es UC08; **usar el código correcto** de la matriz.)

## S3 — Declaración de participantes y NOMENCLATURA
Se declaran en este orden: **actor → boundaries → control → entities**.
```
actor    "Nombre"                      as alias
boundary "CI_<PascalCase>"             as b_xxx
control  "CC_<NombreCU>Controller"     as ctrl
entity   "CE_<snake_case_tabla>"       as e_xxx
```
- **Actor (A):** el actor primario del CU (según la Memoria-Negocio: Usuario/Administrador/Vendedor/Encargado de Almacén). El ejemplo usa "Gerente"; alinear al actor real del CU.
- **Boundary (B) → prefijo `CI_`** (Clase Interfaz), PascalCase. **NO se incluye `CI_PanelPrincipal`**. Boundaries típicos de un CRUD:
  - `CI_Ventana<Entidad>` (b_ventana) — la grilla/ventana de gestión.
  - `CI_Formulario<Entidad>` (b_form) — el modal de alta/edición.
  - `CI_MensajeConfirmacion` (b_confirm) — diálogo de confirmación (`btnAceptar`/`btnCancelar`).
  - `CI_MensajeSistema` (b_msg) — TODOS los mensajes de sistema (Error/Éxito/Aviso).
- **Control (C) → prefijo `CC_`**, un único controlador por CU: `CC_<NombreCU>Controller` (alias `ctrl`).
- **Entity (E) → prefijo `CE_`** + nombre de tabla en snake_case, una por tabla tocada: entidad principal, tablas **maestras** usadas para combos/FK (ej. `CE_categoria`, `CE_unidad_medida`) y tablas **puente N:M** (ej. `CE_proveedor_categoria`, alias `e_prov_cat`).
- **Aliases:** `b_` para boundaries, `ctrl` para el control, `e_` para entities.

## S4 — Firmas de mensajes y flechas
- Todo mensaje lleva **firma completa tipada**: `metodo(param: Tipo, param2: Tipo): TipoRetorno`.
- Flecha de llamada síncrona: `->`. Flecha de retorno/respuesta: `-->` (punteada).
- **Auto-llamada** del control para validaciones internas: `ctrl -> ctrl : validarBoton(btnX: String): bool`.
- Tipos de retorno canónicos: `bool` (validaciones), `int` (conteos/ids: un verificar…Duplicado devuelve int, un registrar… devuelve el id int), `List` (colecciones), `Object` (un registro), `void` (acciones de búsqueda/filtro sin `isset` que repintan la vista directamente). Tipos de parámetro comunes: `String`, `int`, `double`, `bool`, `List`, `Object`.

## S5 — Secuencia de ARRANQUE (directo Actor → Controlador, sin PanelPrincipal)
Cuando el caso de uso inicia por la acción de un botón disparador, la secuencia arranca directamente del Actor al Controlador:
```
gerente -> ctrl : validarBtnX(btnX: String): bool
activate ctrl

ctrl -> ctrl : validarBoton(btnX: String): bool
break si validarBoton() == false
  ctrl -> b_msg : CI_MensajeSistemaShow(msg: String, descripcion: String, link: String)
  b_msg --> gerente : [Error] Acceso no permitido
end

ctrl -> ctrl : validarRol<Actor>(rolUsuario: String): bool
break si validarRol<Actor>() == false
  ctrl -> b_msg : CI_MensajeSistemaShow(msg: String, descripcion: String, link: String)
  b_msg --> gerente : [Error] Privilegios insuficientes
end

ctrl -> e_<entidad> : obtener<Entidad>Activos(): List
activate e_<entidad>
e_<entidad> --> ctrl : listado<Entidad>
deactivate e_<entidad>

ctrl -> b_ventana : CI_Ventana<Entidad>Show(listado<Entidad>: List)
activate b_ventana
b_ventana --> gerente : Muestra Ventana<Entidad> con listado
```
- No se declara ni se activa `CI_PanelPrincipal`.
- `validarBoton()` es el **equivalente directo del `isset(btn)`** del diagrama de actividades.
- `validarRol<Actor>()` es el chequeo de privilegios.
- El `ctrl` se activa una sola vez aquí y se desactiva al final (`deactivate b_ventana` / `deactivate ctrl`).

## S6 — Una operación = un `group`
Cada botón/operación del CU es un `group <NOMBRE OPERACION>` (equivalente a cada `case` del switch de actividades). Patrón interno de cada group:
```
group AGREGAR <ENTIDAD>
  gerente -> b_ventana : clickAgregar<Entidad>(btnAgregar<Entidad>: String)
  b_ventana -> ctrl : validarBtnAgregar<Entidad>(btnAgregar<Entidad>: String): bool

  ctrl -> ctrl : validarBoton(btnAgregar<Entidad>: String): bool
  break si validarBoton() == false
    ctrl -> b_msg : CI_MensajeSistemaShow(msg: String, descripcion: String, link: String)
    b_msg --> gerente : [Error] Acceso no permitido
  end

  ' ... carga de datos auxiliares (ej. categorías) con sus breaks ...
  ctrl -> b_form : CI_Formulario<Entidad>Show(...)
  activate b_form
  b_form --> gerente : Muestra Formulario<Entidad> vacío

  gerente -> b_form : ingresarDatos(..., btnGuardar: String)
  b_form -> ctrl : validarBtnGuardar(btnGuardar: String): bool

  ctrl -> ctrl : validarBoton(btnGuardar: String): bool
  break si validarBoton() == false ... end

  ctrl -> ctrl : validarCampos...(...): int
  break si validarCampos...() == 0 ... end

  ctrl -> e_<entidad> : verificar<Clave>Duplicado(clave): int
  activate e_<entidad>
  e_<entidad> --> ctrl : resultado
  deactivate e_<entidad>
  break Si resultado != 0 ... end

  ctrl -> e_<entidad> : registrar<Entidad>(...): int
  activate e_<entidad>
  e_<entidad> --> ctrl : id_<entidad>
  deactivate e_<entidad>

  ' N:M si aplica:
  ctrl -> e_<puente> : registrar<Relacion>(id, lista): bool
  activate e_<puente>
  e_<puente> --> ctrl : confirmacion
  deactivate e_<puente>

  ctrl -> b_msg : CI_MensajeSistemaShow(msg: String, descripcion: String, link: String)
  b_msg --> gerente : [Éxito] <Entidad> registrado!

  ctrl -> e_<entidad> : obtener<Entidad>Activos(): List
  activate e_<entidad>
  e_<entidad> --> ctrl : listado<Entidad>
  deactivate e_<entidad>

  ctrl -> b_ventana : CI_Ventana<Entidad>Show(listado<Entidad>: List)
  b_ventana --> gerente : Muestra Ventana<Entidad>

  deactivate b_form
end
```
- Groups típicos de un CRUD: `BUSCAR`, `AGREGAR`, `EDITAR`, `DAR DE BAJA` / `ELIMINAR`. (En transaccionales, los groups mapean a los cases del CU: buscar, confirmar, etc.)
- **Patrón `BUSCAR` (sin `isset` por excepción R4 del DA):** no lleva `validarBtnBuscar` ni `validarBoton()`; la frontera invoca directo `buscar<Entidad>(textoBusqueda: String): void`, el controlador valida el campo (`validarCampoBusqueda(): bool` + `break`), consulta `obtener<Entidad>Filtrados(textoBusqueda): List` y repinta `CI_Ventana<Entidad>Show(listadoFiltrado: List)`.
- **Carga de maestros (combos/FK):** cuando el formulario tiene combos de tablas maestras (ej. categoría, unidad), tanto `AGREGAR` como `EDITAR` consultan `obtener<Maestro>Activos(): List` antes de `CI_Formulario<Entidad>Show(...)` y los pasan como parámetros.
- **Re-consulta tras DML exitoso:** después de `[Éxito]` (`CI_MensajeSistemaShow`), el controlador vuelve a consultar `obtener<Entidad>Activos(): List` antes de llamar a `CI_Ventana<Entidad>Show(listado: List)`.
- **Se modelan ambas ramas del Cancelar priorizando SIEMPRE el camino negativo (Cancelar va 1º)** usando `alt / else` tras abrir la vista (formulario o confirmación), exactamente igual que en el Diagrama de Actividades:
  ```
  ctrl -> b_form : CI_Formulario<X>Show()
  activate b_form
  b_form --> encargado : Muestra Formulario<X>
  alt el usuario presiona btnCancelar
    encargado -> b_form : clickCancelar(btnCancelar: String)
    b_form --> encargado : Cerrar modal y mantener vista actual
  else el usuario presiona btnGuardar
    encargado -> b_form : ingresarDatos(..., btnGuardar: String)
    ... flujo de guardar ...
  end
  deactivate b_form
  ```
  Mismo patrón para `CI_MensajeConfirmacion` (`alt` `btnCancelar` 1º / `else` `btnAceptar` 2º), colocando `deactivate b_confirm` dentro de cada rama inmediatamente después de la interacción con `b_confirm`. Así cada sub-rama de botón del DA tiene su correspondencia en el DS en el mismo orden (camino negativo primero).
- **Editar** carga datos del registro (`obtenerDatos<Entidad>(): Object`), luego los maestros activos si hay combos y, si hay N:M, `obtener<Relacion>Asignadas(): List` antes de mostrar el formulario; la unicidad se valida contra otros (`verificar<Clave>EnOtro<Entidad>(clave, id): int`).
- **Dar de Baja / Eliminar** abre `CI_MensajeConfirmacion` y, tras `clickAceptar(btnAceptar)` → `validarBtnAceptar(btnAceptar)` + `validarBoton(btnAceptar)` + `verificarExistenciaActiva(id)`, hace `desactivar<Entidad>(id): bool` (baja lógica).

## S7 — El patrón `break` (corte por validación)
`break si <condición>` encapsula la salida temprana cuando una validación falla; dentro va SIEMPRE el mensaje al actor vía `CI_MensajeSistema`. Es el **equivalente del `<Error> ... stop`** del diagrama de actividades. Hay un `break` por cada validación que pueda cortar el flujo (botón, rol, campos, duplicado, stock, etc.).

## S8 — Mensajes al usuario
- `CI_MensajeSistemaShow(msg: String, descripcion: String, link: String)` para **todo** `[Error]` / `[Éxito]` / `[Aviso]`; la respuesta al actor es `b_msg --> actor : [Error|Éxito|Aviso] <texto>`.
- `CI_MensajeConfirmacionShow(msg: String)` muestra el diálogo con `(btnAceptar, btnCancelar)`, priorizando el camino negativo: en la rama `alt` de cancelación: `actor -> b_confirm : clickCancelar(btnCancelar: String)`, `b_confirm --> actor : Cerrar confirmación y mantener vista actual` + `deactivate b_confirm`; en la rama `else` de aceptación: `actor -> b_confirm : clickAceptar(btnAceptar: String)`, `b_confirm -> ctrl : validarBtnAceptar(btnAceptar: String): bool` + `deactivate b_confirm` (y luego `validarBoton(btnAceptar)` + `break`).

## S9 — Activación / desactivación
- `activate ctrl` una vez (tras la primera llamada del arranque); `deactivate ctrl` al final.
- Cada entity: `activate` al llamarla, `--> ctrl : <retorno>`, `deactivate` inmediatamente después (activaciones cortas por cada acceso a BD).
- Boundaries: `activate` cuando se muestran; `b_form` se desactiva al cerrar el modal; `b_confirm` al responder (dentro de cada rama `alt/else`); `b_ventana` se desactiva al final.
- Cierre del diagrama: `deactivate b_ventana` / `deactivate ctrl`.

## S10 — Métodos de entidad (verbos canónicos)
`obtener...()` (List/Object), `verificar...Duplicado()/...EnOtro...()` (int), `registrar...()` (int=id), `actualizar...()` (bool), `desactivar...()` (bool). Relación N:M: `registrar<Relacion>`, `actualizar<Relacion>`, `obtener<Relacion>Asignadas` sobre la entity puente (patrón equivalente al DELETE+INSERT de actividades).

## S11 — Mapeo Actividades → Secuencia (para derivar un DS desde su diagrama de actividades)
| En Actividades | En Secuencia |
| :--- | :--- |
| `isset(btnX)` + `<Error> stop` | `ctrl -> ctrl : validarBoton(btnX): bool` + `break si validarBoton()==false` |
| Rombo de validación (¿campos?, ¿duplicado?, ¿stock?) + `<Error> stop` | auto-llamada/entity `validar.../verificar...` + `break si ...` |
| `SELECT` de listar/buscar | `ctrl -> e_x : obtener.../buscar...(): List` |
| `SELECT` de precarga (editar) | `ctrl -> e_x : obtenerDatos...(): Object` |
| `INSERT` | `ctrl -> e_x : registrar...(): int` |
| `UPDATE` de datos | `ctrl -> e_x : actualizar...(): bool` |
| `UPDATE estado='Inactivo'` (baja lógica) | `ctrl -> e_x : desactivar...(): bool` |
| N:M DELETE+INSERT | `actualizar<Relacion>()` sobre la entity puente |
| `case <Buscar>` (sin `isset` por R4) | `group BUSCAR` con `b_ventana -> ctrl : buscar...(): void` (sin `validarBoton`) |
| `SELECT` de maestros (combos/FK) | `ctrl -> e_maestro : obtener<Maestro>Activos(): List` antes de `CI_Formulario...Show` |
| `<Confirmación>` panel | `CI_MensajeConfirmacion` (Show + `alt` clickCancelar 1º / `else` clickAceptar/validarBtnAceptar 2º) |
| `Clic en <Cancelar>` en modal | rama `alt` (1º, camino negativo) con `actor -> b_form : clickCancelar(btnCancelar)` (sin pasar por `ctrl`) |
| `<Error>/<Éxito>/<Aviso>` panel | `CI_MensajeSistema` (CI_MensajeSistemaShow) |
| Ventana `<Gestión de X>` / modal `<Formulario de X>` | `CI_Ventana<X>` / `CI_Formulario<X>` |
| Cada `case` del switch | un `group <OPERACION>` |

---

## Checklist antes de cerrar un DS
- [ ] Cabecera + skinparams + `!pragma teoz true`; título `DS_<Nombre> (UCXX)` con código correcto.
- [ ] Actor correcto del CU (coherente con la Memoria-Negocio), no "Gerente" por defecto.
- [ ] Participantes con prefijos `CI_` / `CC_...Controller` / `CE_snake_case` y aliases `b_/ctrl/e_`.
- [ ] Robustez respetada: ningún `actor -> e_x` ni `boundary -> entity`. El único `actor -> ctrl` permitido es el arranque directo del CU (`validarBtn<CUS>`, sin `CI_PanelPrincipal`).
- [ ] Arranque directo: `actor -> ctrl : validarBtn<CUS>` → `validarBoton` (break) → `validarRol` (break) → `obtener listado` → mostrar `CI_Ventana<Entidad>`.
- [ ] Ramas de cancelación (`b_form` y `b_confirm`) modeladas con `alt` (Cancelar 1º, camino negativo) / `else` (Guardar/Aceptar 2º).
- [ ] Un `group` por operación; cada uno abre con validarBtnX → validarBoton + break.
- [ ] Toda validación con posible fallo tiene su `break si ...` y su mensaje por `CI_MensajeSistema`.
- [ ] Mensajes con firma tipada `metodo(p: Tipo): Retorno`; llamadas `->`, retornos `-->`.
- [ ] Activaciones: `ctrl` una vez; entities activate/deactivate por acceso; boundaries al mostrar/cerrar; cierre final balanceado.
- [ ] Coherente con el diagrama de actividades del mismo CU (mismos botones, validaciones y operaciones de BD).

---

## S12 — Regla Híbrida para Rational Rose (`DS PlantUML` → `Trabajo aplicativo.mdl`)
Cuando el Diagrama de Secuencia se traslada o genera dentro de [Trabajo aplicativo.mdl](file:///c:/Users/islac/OneDrive/Desktop/ADS-2026%20II/02-Entregables/Trabajo%20aplicativo.mdl) (Rational Rose) para cumplir con las restricciones estrictas de UML del profesor:
1. **Aislamiento de la línea de vida del Actor:**
   - Solo existe **un único mensaje** desde el Actor en todo el diagrama: `[00] seq=1 : <Actor> → CI_PanelPrincipal : CI_PanelPrincipalShow()`.
   - Ninguna otra flecha sale del Actor ni regresa jamás hacia el Actor.
2. **Inicio directo desde la Frontera hacia el Controlador (sin `click*`, `ingresarDatos`, `modificarDatos`, `clickCancelar`):**
   - Cada acción o botón nace directamente como mensaje `CI_<Vista> → CC_<CU>Controller`:
     - Con `isset`: `validarBtn<X>(btn<X>: String): bool`
     - Sin `isset` (excepción R4, búsquedas): `buscar<X>(textoBusqueda: String): void`
   - Las ramas `else btnCancelar` no se trazan como flechas en Rose.
3. **Sin `validarRol<Actor>()` en el arranque:** tras `validarBtn<CUS>` → `validarBoton` → `CI_MensajeSistemaShow`, pasa directo a la primera consulta o apertura de ventana principal.
4. **Cortes de validación (`break`) y mensajes al usuario:**
   - Cada `break` de PlantUML se traduce en una llamada `CC_<CU>Controller → CI_MensajeSistema : CI_MensajeSistemaShow(msg: String, descripcion: String, link: String)` sin flecha de retorno, acompañada de sus `Label`s visuales (`[si <condición>]` y `[Error|Aviso|Éxito] <mensaje>`).
5. **Únicas flechas de retorno (`ToClientFromSupplier`):**
   - Exclusivamente de **Entidad → Controlador** (`CE_<tabla> --> CC_<CU>Controller`): `listado*`, `datos*`, `resultado`, `id_*`, `confirmacion*`. Nunca hay flechas de retorno entre `Frontera` y `Controlador`.
6. **Cierre de cada operación DML:**
   - Cada bloque DML termina directamente en el `CI_MensajeSistemaShow` de `[Éxito]`, sin re-consultar el listado ni re-invocar `CI_Ventana...Show()` al cierre del bloque.
7. **Numeración de secuencias:**
   - `seq=1` agrupa el arranque (`1`, `1.1`, `1.1.1`...) y la búsqueda inicial en la vista principal (`1.1.1.2.1`...).
   - Las demás operaciones del CU nacen como secuencias raíz independientes (`seq=2`, `seq=3`, `seq=4`...) desde su frontera hacia el controlador.
