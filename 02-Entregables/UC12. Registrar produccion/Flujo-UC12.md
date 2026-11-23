---
tipo: entregable
entregable: Flujo de Caso de Uso y Arquitectura de Vista
caso_de_uso: UC12. Registrar produccion
estado: finalizado (Arquitectura Estricta)
fecha: 2026-10-06
---
# UC12: Registrar Producción (Documento Maestro)

## Arquitectura de la Interfaz (Visión UX/UI)

El flujo de este módulo está diseñado para ser altamente transaccional pero visualmente simple, garantizando que el encargado de almacén pueda manufacturar productos sin fricción. La lógica de negocio pesada y la validación de inventarios recaen exclusivamente en el backend (Controlador). Toda la operación se resuelve en una sola interfaz web.

### Interfaz Única: `<Registro de Producción>`
**Distribución Visual (Layout Lógico):**

Esta interfaz es un formulario web de página completa dividido en dos bloques verticales. El bloque inferior permanece oculto hasta que el usuario ejecute la acción de cálculo.

- **Bloque Superior (Definición del Producto):**
  - A la izquierda, un cuadro de texto `txtBuscarProducto` con placeholder *"Ingrese nombre o código del producto"* y un botón `[Buscar Producto]` a su derecha.
  - Debajo del buscador, una "Tarjeta de Producto" con Labels de solo lectura (`lblNombreProducto`, `lblCodigoProducto`, `lblStockActualTerminado`). Inicialmente vacía, se llena tras una búsqueda exitosa.
  - A la derecha de la tarjeta, un cuadro numérico `txtCantidadProducir` (deshabilitado hasta que se seleccione un producto válido) con un botón de acción secundario **`[Calcular Insumos]`** (Color Gris/Secundario) también deshabilitado inicialmente.

- **Bloque Inferior (Receta Calculada y Confirmación):**
  - Una Grilla (`DataGrid`) de solo lectura que aparece **únicamente** después de presionar `[Calcular Insumos]`. Esta grilla contiene 4 columnas:
    1. **Insumo Químico** (Nombre del insumo de la receta)
    2. **Cantidad Requerida** (El resultado de: cantidad_unitaria × cantidad_a_producir)
    3. **Stock Físico Actual** (Leído directamente de la BD al momento del cálculo)
    4. **Estado** (Indicador visual: Verde si stock actual ≥ cantidad requerida, Rojo si es insuficiente)
  - Debajo de la Grilla, dos botones alineados horizontalmente:
    - **`[Confirmar Producción]`** (Color Verde/Primario, grande)
    - **`[Cancelar]`** (Color Rojo, secundario)

**Comportamiento del Flujo (MVC Clásico):**
1. El encargado escribe "Lejía" en el buscador y presiona `[Buscar Producto]`. El sistema consulta la BD y devuelve los datos en la tarjeta superior. Se habilitan el campo de cantidad y el botón de cálculo.
2. Ingresa "50" en `txtCantidadProducir` y presiona `[Calcular Insumos]`.
3. El formulario viaja al Backend (vía POST). El Controlador lee la tabla de Recetas, multiplica cada proporción unitaria por 50, y recarga la página mostrando la Grilla inferior llena con los insumos y sus stocks.
4. El usuario revisa la grilla (puede ver visualmente si algún insumo está en rojo) y presiona `[Confirmar Producción]`.
5. El Backend re-verifica el stock en BD en tiempo real (para evitar inconsistencias por concurrencia). Si algún insumo no tiene stock suficiente, muestra `<Panel de Error>`. Si todo está bien, ejecuta el bloque DML completo (INSERT cabecera + INSERT detalle + UPDATE descuento insumos + UPDATE suma producto terminado) y muestra `<Panel de Éxito>`.

---

## Lógica Transaccional y de Seguridad (Backend)

1. **Uso de `isset()`:**
   - `[Buscar Producto]`: NO lleva `isset()`. Solo ejecuta un `SELECT` de display en la misma vista.
   - `[Calcular Insumos]`: SÍ lleva `isset()`. Ejecuta lógica matemática en el Controlador y transiciona el estado visual de la interfaz (habilita la grilla y el botón de confirmación).
   - `[Confirmar Producción]`: SÍ lleva `isset()`. Ejecuta operaciones DML críticas sobre el inventario.
2. **Casos Negativos Cubiertos (Validaciones de Secuencia + Datos):**
   - Campo de búsqueda vacío → Error: *"Debe ingresar un nombre o código de producto"*.
   - Producto no encontrado → Aviso: *"Producto no encontrado"*.
   - Clic en Calcular sin haber seleccionado producto → Error: *"Debe buscar y seleccionar un producto antes de calcular"*.
   - Cantidad ≤ 0 → Error: *"La cantidad a producir debe ser mayor a cero"*.
   - Producto sin receta configurada → Error: *"El producto seleccionado no tiene una receta asignada"*.
   - Clic en Confirmar sin haber calculado insumos → Error: *"Debe calcular los insumos requeridos antes de confirmar"*.
   - Stock insuficiente de uno o más insumos → Error: *"Stock insuficiente en uno o más insumos químicos..."*.
3. **Validación de Integridad:** La resta de insumos y la suma del producto terminado se ejecutan en un mismo bloque transaccional. Si un UPDATE falla, todo se revierte (Rollback implícito).
4. **Consistencia de Inventario:**
   - Materia Prima (Insumos): `stock_fisico = stock_fisico - cantidad_consumida`
   - Producto Terminado: `stock_fisico = stock_fisico + cantidad_producida`
5. **Limpieza de Sesión:** Al confirmar exitosamente o al cancelar, las variables temporales almacenadas en sesión (producto seleccionado, receta calculada) se destruyen para evitar datos residuales.
