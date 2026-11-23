---
tipo: sistema
proyecto: ADS-2026 II
estado: consolidado
fecha: 2026-10-03
---
# Memoria del Negocio (CARTOLIMP)

Este documento centraliza el conocimiento y las decisiones arquitectónicas del negocio CARTOLIMP para el desarrollo de su software.

## 1. Contexto General
- **Empresa**: CARTOLIMP (Microempresa).
- **Giro**: Producción y venta de productos de limpieza; venta de insumos químicos al por mayor y menor.
- **Personal Real**: 2 trabajadores (Jefe y Socia).
- **Escalabilidad**: El sistema mapea roles específicos aunque en la práctica los asuman las mismas personas.

## 2. Definición de Actores del Sistema (Privilegios)
1. **Usuario**: Actor base (Autenticación y claves).
2. **Administrador**: Privilegios totales de sistema (Usuarios).
3. **Vendedor**: Encargado del Punto de Venta (Clientes, Ventas).
4. **Encargado de Almacén**: Dueño del inventario (Proveedores, Insumos, Productos, Producción, Mermas).
*(Nota: El rol Gerente fue absorbido por Almacén/Admin dado que las Compras se simplificaron a ingresos directos).*

## 3. Decisiones de Diseño y Arquitectura
- **APIs descartadas**: Esquema MVC puro. Ingreso manual de datos (DNI/RUC).
- **Venta Simplificada**: Cotizar, cobrar y despachar ocurren en un solo Caso de Uso (`Registrar venta`).
- **Reservas como Estado**: No hay un CUS complejo de reservas; es simplemente un estado de la venta ("Reservado") dentro de `Registrar venta`, que exige un cobro parcial (adelanto).
- **Adiós a la Burocracia de Compras**: Se elimina la "Orden de Compra". Los proveedores son solo un catálogo, y las llegadas de stock se registran directamente como `Registrar ingreso de mercancía` dentro de Inventario.
- **Control de Errores**: Todo error humano en inventario se corrige mediante `Gestionar movimientos de inventario` (Kardex), anulando registros para mantener la auditoría intacta.
- **Producción Incluida**: El sistema descontará insumos para sumar productos terminados (`Registrar producción`).
- **Identidad por DNI**: En `Gestionar usuarios` (y como criterio general), el **DNI (`documento`) es el identificador único e inmutable** de la persona. La validación de existencia/duplicado se hace por DNI, no por `username`. El DNI es editable solo al crear; al editar va en sólo lectura. El `username` es una credencial de acceso editable, no la identidad.

## 4. Casos de Uso del Sistema Definitivos (14 CUS)
- **Seguridad**: UC01. Autenticar usuario, UC02. Actualizar clave, UC03. Recuperar clave, UC04. Gestionar usuarios.
- **Ventas**: UC05. Gestionar clientes, UC06. Registrar venta, UC07. Atender reclamo y devolución.
- **Inventario**: UC08. Gestionar proveedores, UC09. Gestionar insumos químicos, UC10. Gestionar productos de limpieza, UC11. Registrar ingreso de mercancía, UC12. Registrar producción, UC13. Registrar merma o pérdida, UC14. Gestionar movimientos de inventario.
