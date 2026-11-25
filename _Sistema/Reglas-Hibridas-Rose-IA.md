---
tipo: sistema
proyecto: ADS-2026 II
estado: vigente
fecha: 2026-11-24
referencia: Reglas Híbridas PlantUML → Rational Rose (.mdl) para Agentes / IA
---
# Reglas Híbridas para Edición de Rational Rose (`.mdl`) por Agentes / IA

Este documento define las reglas obligatorias que debe seguir cualquier agente o IA al trasladar los diagramas PlantUML (`.txt`) al modelo nativo de Rational Rose (`.mdl` en formato Petal).

> 🔴 **Regla de Seguridad de Archivos:**
> - **Nunca** modificar directamente un archivo `.mdl` maestro sin crear primero un respaldo `.bak` en la carpeta `scratch/`.
> - Si el usuario indica un archivo de trabajo específico (ej. `Trabajo aplicativo Ajuste.mdl`), trabajar **exclusivamente** sobre ese archivo y no tocar el archivo estable de referencia (ej. `Trabajo aplicativo (1).mdl`).
> - Verificar siempre que el balance de paréntesis `( )` agregados sea exacto (`open == close`) y preservar el encoding `latin-1` y los saltos de línea `CRLF`.

---

## 1. Diagrama de Actividades (`D. Actividades.txt` → `State_Machine` + `ActivityDiagram`)

### 1.1 Ubicación en Rational Rose
Todo Diagrama de Actividades vive **dentro de su Caso de Uso** en:
`Use Case View` → `Use-Case Model` → `Use Cases` → `Modulo <Seguridad | Ventas | Almacen>` → `<UCXX. Nombre del CU>` → `statemachine (object State_Machine "State/Activity Model...")` → `statediagrams (object ActivityDiagram "DA UCXX. <Nombre>")`.

### 1.2 Carriles (`Swimlane` / `Partition`)
- **`Partition` del Actor (carril izquierdo, `width ~ 750`):**
  - Vinculado al actor real del CU (`class "Use Case View::Use-Case Model::Actors::<Actor>"`, `quidu` del Actor).
  - **Único contenido:** el `State` de tipo `"StartState"` cuya etiqueta es la acción disparadora del actor (ej. `Dar clic en el botón "Gestionar Clientes" del Panel Principal`).
- **`Partition "SISTEMA"` (carril derecho, ancho amplio para las ramas paralelas):**
  - Contiene absolutamente todas las actividades, decisiones, estados finales (`EndState`) y notas SQL.

### 1.3 Mapeo 1:1 PlantUML → Petal (Rational Rose)
| Elemento en PlantUML (`D. Actividades.txt`) | Objeto en Rose (`State_Machine`) | Vista en Rose (`ActivityDiagram`) |
| :--- | :--- | :--- |
| `start` + `:Acción del actor;` | `(object State "<Acción>" type "StartState")` | `StateView "StartState"` dentro del `Swimlane` del Actor |
| `:Acción del sistema;` | `(object ActivityState "<Acción>")` | `ActivityStateView` dentro del `Swimlane "SISTEMA"` |
| `if (...)` / `switch (...)` | `(object Decision "<Condición>")` | `DecisionView` + `SegLabel` |
| Rama `then (SI/NO/TRUE/False)` o `case (boton <X>)` | `(object State_Transition ... label "" Event (object Event "<etiqueta>"))` *(NO usar `condition`, que agrega corchetes `[...]`; va como `Event`)* | `TransView` + `SegLabel` con `<etiqueta>` |
| `stop` | `(object State "$UNNAMED$..." type "EndState")` | `StateView "EndState"` (un nodo independiente por rama terminal) |
| `note right: SELECT/INSERT/UPDATE...` | *(solo visual en el diagrama)* | `NoteView` con el SQL completo + `AttachView` hacia su `ActivityStateView` |

### 1.4 Reglas de Integridad Petal para Actividades
1. **Rutas completas y `Event` en `State_Transition`:** `client` (origen) y `supplier` (destino) deben usar la ruta jerárquica exacta:
   `Use Case View::Use-Case Model::Use Cases::Modulo <X>::<UCXX. Nombre>::<State_Machine_Name>::<NombreExactoNodo>`
   *(y su `client_quidu` / `supplier_quidu` coincidiendo con los `quid` reales de los nodos en `states`)*.
   Para las ramas de salida de un `Decision` (`FALSE`/`TRUE`, `SI`/`NO`, `boton <X>`), asignar la etiqueta en **`Event (object Event "<etiqueta>")`** con `label ""` (nunca en `condition`, ya que `condition` lo renderiza como texto libre/guarda `[...]`).
2. **Nodos con el mismo texto:** En `D. Actividades.txt` hay múltiples `:Mostrar panel <Error> "Acceso no permitido";`. En `State_Machine`, Rose distingue nodos por su `quid`, pero como `State_Transition` referencia también por nombre, **cada instancia repetida debe tener un sufijo o espacio/salto mínimo diferenciador** (o usar sufijo limpio como `"Mostrar panel <Error> \"Acceso no permitido\" "` con `quid` único) para evitar ambigüedades al enlazar en Rose.
3. **Coordenadas, tamaño de cajas y espaciado anti-solapamiento:**
   - **Tamaño de `ActivityStateView` (evitar texto entrecortado):**
     - Calcular ancho amplio según longitud del texto: `width = min(1100, max(480, len(label) * 12 + 120))`.
     - En su `ItemLabel`, usar `max_width = width - 106` y `nlines = 2` con `autoResize TRUE` (valores nativos de Rose; usar `nlines = 3` o `4` desplaza el texto hacia arriba dentro de la cápsula).
   - **Separación entre columnas del `switch`:** mínimo **`2200–2300 px`** entre troncos principales (ej. `X = 1750, 3950, 6250, 8550, 10750`).
   - **Ramas de error / `<Cancelar>`:** a **`X - 620 px`** (nunca invaden la nota SQL de la columna anterior).
   - **Notas SQL (`NoteView`):** a **`X + 620 px`**, `width ~ 680 px`, evitando chocar con la columna siguiente.
   - **Paso vertical (`Y`):** mínimo **`260–280 px`** entre actividad/rombo para que las etiquetas `[SI]`, `[NO]`, `[False]`, `[TRUE]` queden legibles.
   - **Ancho del `Swimlane "SISTEMA"`:** extender hasta **`X_ultima_columna + 1100 px`** (y colocar el `Swimlane ""` de cierre al final) para que la última rama nunca quede cortada ni desbordada.

---

## 2. Diagrama de Secuencia (`DS. Secuencia.txt` → `Mechanism` + `InteractionDiagram`)

Se rige por la **Regla Híbrida S12** de [Reglas-Diagramas-Secuencia.md](file:///c:/Users/islac/OneDrive/Desktop/ADS-2026%20II/_Sistema/Reglas-Diagramas-Secuencia.md#L236-L255):
- Ubicación: `Logical View` → `Analysis Model` (clases) y `Realizacion Modulo <X>` → `UCRXX. <Nombre>` → `Mechanism` + `InteractionDiagram "DS_<Nombre>"`.
- **Orden estricto de columnas (MVC):** `Actor` → todos los `Boundaries` (`CI_*`, incluyendo `CI_MensajeSistema`) → `Controlador` (`CC_*Controller`) → todas las `Entidades` (`CE_*`).
- **Actor aislado:** único mensaje inicial `1: ...` (sin retornos ni segundas llamadas al actor).
- **Cancelar en MVC estricto (sin Boundary → Boundary):** al presionar `btnCancelar`, el formulario **no** llama directamente a la ventana anterior; pasa por el controlador (`CI_Formulario* → CC_*Controller : cancelarOperacion(btnCancelar: String): void` y `CC_*Controller → CI_Ventana* : CI_Ventana*Show()`).
- **Autollamadas en Rose (`SelfMessView`):** toda llamada del controlador hacia sí mismo (`validarBoton`, `validarRol*`, `validarCampos*`, `encriptarPassword`, etc.) debe generarse con `(object SelfMessView ...)` (con `terminus` a `+150 px` a la derecha y `Focus_Entry` con `Nested = TRUE`, `height = 90`), nunca como `InterMessView`.
- **Entidades con barra única por consulta:** cada llamada `CC_*Controller → CE_*` y su retorno punteado (`Return` + `ToClientFromSupplier`) comparten **el mismo `Focus_Of_Control`** en la entidad.
- **Barras continuas del controlador:** cada flujo/petición al controlador mantiene **1 `Focus_Of_Control` continuo** de inicio a fin (sin partirse en las autollamadas ni en las salidas de error hacia `CI_MensajeSistema`).
- **Sin cuadros `NoteView` automáticos:** los rectángulos agrupadores se agregan manualmente en Rose; dejar el lienzo limpio solo con `Label`s de condición (`[si ...]`) y resultado (`[Error|Éxito|Aviso]`).
- **Sintaxis Petal de mensajes (`Mechanism`):** en cada `object Link`, la lista de mensajes debe declararse obligatoriamente como `messages (list Messages ...)` (nunca `message_list`, ya que provoca `Petal syntax error near 'message_list'` al cargar en Rose).

---

## 3. Diagrama de Objetos (`DO. Objetos.txt` → `Association` + `ClassDiagram`) y Diseño (`Design Model`)

### 3.1 Archivo de trabajo y ubicación
- Trabajar **exclusivamente** sobre `Trabajo aplicativo DO.mdl` (no modificar `Trabajo aplicativo Ajuste.mdl` ni `Trabajo aplicativo (1).mdl`).
- Ubicación de clases compartidas: `Logical View::Analysis Model::{Boundary, Controllet, Entity}` con estereotipos `"boundary"`, `"control"`, `"entity"`.
- Ubicación de cada DO: `Logical View::Analysis Model::Realizacion Modulo <Seguridad | Ventas | Almacen>::UCRXX. <Nombre>` (`logical_models` para `Association` y `logical_presentations` para `ClassDiagram "DO_<Nombre>"`).

### 3.2 Reglas para `Entity` y Diagrama `Entities`
- El diagrama `Logical View::Analysis Model::Entity::Entities` debe incluir las **`18` entidades `CE_*`** y sus **`24` asociaciones** con cardinalidades (`1`, `0..n`, `1..n`).
- En los atributos de cada `CE_*`, indicar explícitamente **`INT (PK)`**, **`INT (FK)`** o **`INT (PK, FK)`** en el campo `type`.
- **No** usar `exportControl "Protected"` ni `"Implementation"` para simular PK/FK (en análisis UML de Rose eso solo cambia visibilidad).
- Acumular en cada `CE_*` las operaciones (`operations`) usadas por los CUS trazados desde `DO. Objetos.txt` y los `DA` aprobados.

### 3.3 Reglas para cada `DO_<Nombre>` (PlantUML → Rose Petal)
1. **Actores y vistas (`ClassView`)**:
   - Actor: `Use Case View::Use-Case Model::Actors::<Actor>` con `icon "Actor"`, `icon_style "Icon"`, `autoResize TRUE`.
   - `Boundary`: `icon "boundary"`, `width 230`, `height 154`.
   - `Controllet` (`CC_*Controller`): `icon "control"`, `height 154`.
   - `Entity` (`CE_*`): `icon "entity"`, `height 154`.
2. **Diferencia clave `DO. Objetos.txt` (PlantUML) vs `DO_*` (Rational Rose):**
   - En **PlantUML (`DO. Objetos.txt`)** se trazan **todas** las interacciones numeradas (`1.`, `2.`, `3a.`, `6b-cancel.`, `*`), incluyendo múltiples flechas entre el mismo par de clases y flechas `Actor → Frontera` por cada `click*` / `-cancel`.
   - En **Rational Rose (`Trabajo aplicativo DO.mdl`)** el DO es un VOPC estructural limpio con **una única asociación por par conectado** (sin flechas duplicadas ni flechas `Actor → Frontera` para `click*` / `-cancel`):
     - **Caso especial `UCR01. Autenticar usuario`**: el actor conecta directamente con **`CI_FormularioLogin`** (sin flecha `Usuario → CC_AutenticarUsuarioController`) y **todas** las asociaciones van sin etiqueta (`$UNNAMED$...`, sin `SegLabel`).
     - **Casos `UCR02` a `UCR14`**: solo el **primer paso** `Actor → CC_*Controller` lleva etiqueta (`1. validarBtn<Accion>(btn<Accion>)`) tanto en `(object Association "<etiqueta>" ...)` como en `AssociationViewNew` + `SegLabel`. Todas las demás asociaciones (`Controlador ↔ Frontera`, `Controlador ↔ Entidad`) van sin etiqueta (`$UNNAMED$...` sin `SegLabel`).
3. **Trazabilidad previa al pase**:
   - Verificar siempre que `DO. Objetos.txt` esté sincronizado con el `DA` aprobado (ej. en `UC11` se añadió `boton <Ver Detalle>` en `D. Actividades.txt`; incluirlo al generar `UCR11`).

### 3.4 Diseño (`Design Model::Entity`)
- Clases rectangulares UML (sin estereotipo `entity` ni prefijo `CE_`), con compartimentos completos de atributos SQL y métodos acumulados.
