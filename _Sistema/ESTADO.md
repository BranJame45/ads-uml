---
tipo: sistema
proyecto: ADS-2026 II
estado: en progreso (Pase a Rational Rose - DO)
fecha: 2026-11-24
---
# Estado del Proyecto

**Dónde quedamos (para continuar en otro ordenador):**
1. **Entregables PlantUML (`02-Entregables/`):**
   - `UC01`–`UC14` completos (`Flujo-UCXX.md`, `D. Actividades.txt`, `DO. Objetos.txt`, `DS. Secuencia.txt`) y `D. Entidades.txt`.
   - En `UC11/D. Actividades.txt` se agregó la rama `boton <Ver Detalle>` (pendiente reflejarla en `UC11/DO. Objetos.txt` y `UC11/DS. Secuencia.txt` si se requiere consistencia documental; sí incluirla al pasar `UCR11` a Rose).
2. **Rational Rose — Diagramas de Actividades (`Trabajo aplicativo Ajuste.mdl`):**
   - **`UC01`–`UC14` aprobados** al 100% (archivo congelado como aprobado; no modificar).
3. **Rational Rose — Diagramas de Objetos de Análisis (`Trabajo aplicativo DO.mdl`):**
   - **Completados y verificados:**
     - `Logical View::Analysis Model::Entity::Entities` (`18` entidades `CE_*` con `INT (PK)` / `INT (FK)` / `INT (PK, FK)` y `24` asociaciones con cardinalidades).
     - `UCR01. Autenticar usuario` (`DO_Autenticar usuario`, directo `Usuario → CI_FormularioLogin`, solo 1er paso etiquetado).
     - `UCR02`–`UCR07` (`DO_Actualizar clave`, `DO_Recuperar clave`, `DO_Gestionar usuarios`, `DO_Gestionar clientes`, `DO_Registrar venta`, `DO_Atender reclamo y devolución`), iniciando `Actor → Controlador` en el 1er paso y el resto sin etiqueta.
   - **Pendiente (próximo paso en otro ordenador):**
     - Generar **`UCR08` a `UCR14`** (`Realizacion Modulo Almacen`: `UC08`–`UC14`) en `Trabajo aplicativo DO.mdl` siguiendo [Reglas-Hibridas-Rose-IA.md](file:///c:/Users/islac/OneDrive/Desktop/ADS-2026%20II/_Sistema/Reglas-Hibridas-Rose-IA.md) (sección 3).
     - Limpiar el `DO_Gestionar proveedores` antiguo que quedó ubicado dentro de `UCR10` al rehacer `UCR10`.
     - Luego pasar a Diagramas de Secuencia (`DS`) y `Design Model`.
