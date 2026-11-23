---
tipo: entregable
entregable: Flujo de Caso de Uso y Arquitectura de Vista
caso_de_uso: UC13. Registrar merma o pérdida
estado: finalizado (Arquitectura Estricta)
fecha: 2026-10-13
---
# UC13: Registrar Merma o Pérdida (Transaccional)

## Arquitectura de la Interfaz (Visión UX/UI)

Flujo **transaccional** (no CRUD), con el mismo molde que `UC12. Registrar producción`: una sola interfaz donde el **Encargado de Almacén** busca el ítem afectado, declara la cantidad perdida y el motivo, y confirma. La lógica de validación de stock y el descuento recaen en el backend. La merma afecta el inventario **solo al confirmar**; cancelar no toca el stock.

### Interfaz Única: `<Registro de Merma>`
**Distribución Visual (Layout Lógico):**

- **Bloque Superior (Identificación del Ítem):**
  - `cboTipoItem` — selector **Insumo / Producto** (determina sobre qué `tipo` de la tabla `Producto` se busca).
  - `txtBuscarItem` con placeholder *"Nombre o código del ítem"* y botón `[Buscar Ítem]`.
  - "Tarjeta de Ítem" de sólo lectura (`lblNombre`, `lblCodigo`, `lblStockActual`), vacía hasta una búsqueda exitosa.
- **Bloque Inferior (Datos de la Merma):** deshabilitado hasta seleccionar un ítem.
  - `txtCantidadPerdida` (numérico).
  - `cboMotivo` — **Derrame / Rotura / Vencimiento / Evaporación**.
  - Botones **`[Confirmar Merma]`** (verde/primario) y **`[Cancelar]`** (rojo/secundario).

**Comportamiento del Flujo (MVC Clásico):**
1. El Encargado elige el tipo (p. ej. *Insumo*), escribe "Formol" y presiona `[Buscar Ítem]`. El sistema consulta la BD y llena la tarjeta con el ítem y su stock actual; habilita cantidad y motivo.
2. Ingresa la cantidad perdida, elige el motivo y presiona `[Confirmar Merma]`.
3. El Backend re-verifica el stock en tiempo real: si la cantidad supera el stock disponible, muestra `<Panel de Error>`. Si es válida, calcula el costo estimado, inserta la merma y descuenta el stock, mostrando `<Panel de Éxito>`.
4. Si en cualquier momento antes de confirmar presiona `[Cancelar]`, se limpia la sesión y se vuelve al Panel Principal **sin afectar el stock**.

---

## Lógica Transaccional y de Seguridad (Backend)

1. **Uso de `isset()`:**
   - `[Buscar Ítem]`: **NO** lleva `isset()`. Es una búsqueda de display que llena la tarjeta dentro de la **misma** vista (no transiciona).
   - `[Confirmar Merma]`: **SÍ** lleva `isset()`. Ejecuta las operaciones DML sobre el inventario.
   - `[Cancelar]`: limpia la sesión y redirige (mismo patrón que `UC12`); no realiza DML.

2. **Ítem polimórfico (insumo/producto):** el `cboTipoItem` mapea a `tipo = 'Insumo'` o `tipo = 'Terminado'` sobre la tabla `Producto`. La búsqueda incluye ese filtro para no mezclar catálogos.

3. **Validaciones en cascada (RF-117/118):**
   - Tipo no seleccionado o búsqueda vacía → Error.
   - Ítem no encontrado → Aviso.
   - Confirmar sin ítem en sesión → Error.
   - Cantidad ≤ 0 o motivo sin seleccionar → Error.
   - **Cantidad de merma > stock disponible → Error** (re-verificado en BD al confirmar, para evitar inconsistencias por concurrencia).

4. **Registro y descuento atómico (RF-119/120):** al confirmar se ejecuta, en un mismo bloque transaccional: `INSERT INTO Merma (id_prod, cantidad, motivo, costo_estimado, fecha, id_usuario)` y `UPDATE Producto SET stock_fisico = stock_fisico - ?`. El **costo estimado** = cantidad perdida × costo unitario del ítem.

5. **Cancelación segura (RF-121):** como el stock solo se modifica en `[Confirmar Merma]`, presionar `[Cancelar]` (o abandonar antes de confirmar) nunca altera el inventario; solo se limpian las variables de sesión.

---

## Trazabilidad de Requerimientos
| RF | Cubierto por |
| :--- | :--- |
| RF-116 Formulario tipo/ítem/cantidad/motivo | Interfaz `<Registro de Merma>` (cboTipoItem, buscador, txtCantidadPerdida, cboMotivo) |
| RF-117 Verificar cantidad ≤ stock disponible | `¿Cantidad perdida > stock disponible?` + `SELECT stock_fisico` |
| RF-118 Mensaje de error si excede stock | `<Error>` "La cantidad de merma no puede superar el stock existente" |
| RF-119 Descontar del stock al confirmar | `UPDATE Producto SET stock_fisico = stock_fisico - ?` |
| RF-120 Guardar merma con fecha/usuario/motivo/costo | `INSERT INTO Merma (..., fecha, id_usuario)` + cálculo de costo estimado |
| RF-121 Cancelar sin afectar stock | case `<Cancelar>` (limpia sesión, no hay DML de stock) |

> **Nota de modelo de datos:** el *costo estimado* asume un campo de **costo unitario** por ítem en la tabla `Producto` (costo de compra para insumos, costo de producción para terminados). Si el esquema final valúa la merma con otro criterio (p. ej. precio de venta), basta cambiar la columna usada en el cálculo.
