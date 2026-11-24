---
tipo: entregable
entregable: Flujo de Caso de Uso y Arquitectura de Vista
caso_de_uso: UC06. Registrar venta
estado: finalizado (Arquitectura Estricta)
fecha: 2026-10-04
---
# UC06: Registrar Venta (Documento Maestro)

## Arquitectura de la Interfaz (Visión UX/UI)

El flujo de este módulo prioriza la rapidez operativa en mostrador (Point of Sale - POS). El sistema divide la operación en interfaces independientes y claras, blindadas mediante validaciones backend restrictivas (`isset`) en cada botón de transición.

### Interfaz 1: `<Selección de Cliente>`
**Distribución Visual:**
- **Sección Superior (Buscador):**
  - Un cuadro de texto `txtBuscarCliente` y un botón `[Buscar Cliente]`.
  - Debajo, una "Tarjeta de Cliente" (`lblNombre`, `lblDocumento`) que inicialmente indica que no hay cliente seleccionado.
- **Sección Inferior (Bifurcación de Negocio):**
  - Dos botones grandes que dirigen el destino del flujo:
    1. **`[Completar Reserva Existente]`**
    2. **`[Iniciar Nueva Venta]`**

**Comportamiento:**
- Si se da clic a **`[Completar Reserva Existente]`** sin haber buscado un cliente, el sistema bloquea la acción. Si hay cliente, el sistema busca reservas en la BD y, de existir, abre directamente la interfaz `<Cobro de Reserva>`.
- Si se da clic a **`[Iniciar Nueva Venta]`** sin cliente, el sistema advierte que se usará "Público General". Al aceptar, el flujo abre la interfaz `<Catálogo de Productos y Carrito>`.

---

### Interfaz 2A: `<Cobro de Reserva>` (Rama A)
Esta interfaz es exclusiva para saldar deudas de ventas que quedaron en estado 'Reservada'.
**Distribución Visual:**
- **Panel Izquierdo:** Un resumen de sólo lectura que muestra los ítems de la reserva y el saldo pendiente.
- **Panel Derecho:**
  - `cboMetodoPago` (Efectivo, Yape, etc.).
  - `txtMontoIngresado`.
  - Botón **`[Confirmar Pago]`**.
- **Regla Estricta:** El botón de confirmación arrojará error si el monto ingresado es menor al Saldo Pendiente. Las reservas se liquidan en su totalidad, no en partes.

---

### Interfaz 2B: `<Catálogo de Productos y Carrito>` (Rama B)
Esta interfaz es exclusiva para nuevas ventas y se compone de dos paneles interactivos.
**Distribución Visual:**
- **Panel Izquierdo (Catálogo):**
  - Buscador propio `txtBuscarItem` y botón `[Buscar Ítem]`.
  - Grilla (`DataGrid`) con los resultados. Al seleccionar uno, se activa la opción para colocar la cantidad y presionar `[Agregar al Carrito]`.
- **Panel Derecho (El Carrito):**
  - En la parte superior, un `ComboBox` llamado `cboTipoComprobante` (Boleta / Factura).
  - Grilla temporal (basada en el arreglo de sesión) con los ítems agregados y sus respectivos botones de `[Modificar Cantidad]` y `[Eliminar Ítem del Carrito]`.
  - En la parte inferior, los totales sumados (`Subtotal`, `IGV`, `Total General`) y el botón **`[Procesar Venta]`**.

**Validación de Tipo de Comprobante:**
- Al presionar `[Procesar Venta]`, si el `cboTipoComprobante` es **"Factura"** y el cliente es "Público General" o no tiene **RUC**, el sistema arroja error. Si es **"Boleta"** con cliente registrado, valida que tenga **DNI** (si es "Público General", permite Boleta por defecto).

---

### Interfaz 3: `<Cobro Final>`
El cajero aterriza aquí cuando la venta del carrito (Interfaz 2B) es válida.
**Distribución Visual:**
- **Transacción:** 
  - `cboMetodoPago` y `txtMontoIngresado`.
  - Botón Final: **`[Confirmar Transacción]`**.

**Comportamiento Backend (Reserva Automática vs Venta):**
- Si el `txtMontoIngresado` es **menor** al Total General, el backend crea un registro tipo "Ticket Interno" con `estado = 'Reservada'`, calculando el saldo restante. Se incrementa el `stock_separado` en la BD.
- Si el `txtMontoIngresado` cubre o excede el total, se emite la Factura/Boleta con `estado = 'Completada'`, descontando el `stock_fisico`. El método de pago simplemente se registra en la base de datos sin alterar el flujo.

---

### Interfaz 4: `<Vista de Comprobante>`
Se muestra automáticamente tras completar una venta directa o liquidar una reserva (`estado = 'Completada'`).
**Distribución Visual:**
- **Vista Previa:** Datos de la Boleta/Factura generada (correlativo, cliente, detalle de ítems y totales).
- **Acciones:** `[Imprimir]`, `[Descargar PDF]` y `[Finalizar]` (retorna a `<Selección de Cliente>`).

---

## Lógica Transaccional y de Seguridad (Backend)

1. **Uso de `isset()`:** Sólo los botones que implican transiciones de vista, escritura de sesión temporal o sentencias DML (Insert/Update) están protegidos con `isset()`. Los botones de simple búsqueda (`[Buscar Cliente]`, `[Buscar Ítem]`) ejecutan `SELECT` sin validación estricta de controlador.
2. **Ciclo de Vida de la Sesión Temporal:** El arreglo del carrito nace al entrar a la Interfaz 2B y se destruye al ejecutar el DML en la Interfaz 3 o al darle al botón Cancelar.
3. **Consistencia de Inventario:** 
   - Reservas: `+ stock_separado`
   - Cobro de Reservas: `- stock_separado`, `- stock_fisico`
   - Ventas Directas: `- stock_fisico`
