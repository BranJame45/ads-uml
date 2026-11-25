---
tipo: sistema
proyecto: ADS-2026 II
estado: en progreso (Diseño - PlantUML y Design Model)
fecha: 2026-11-25
---
# Estado del Proyecto

**Dónde quedamos (listo para continuar en otro ordenador):**
1. **Análisis (`DA` + `DO` + `DS`):**
   - **PlantUML (`02-Entregables/`):** `UC01`–`UC14` completos (`Flujo`, `DA`, `DS`, `DO`) y `D. Entidades.txt`.
   - **Rational Rose `DA` (`Trabajo aplicativo Ajuste.mdl`):** `UC01`–`UC14` **aprobados** (congelado).
   - **Rational Rose `DO` (`Trabajo aplicativo DO.mdl`):** `UCR01`–`UCR14` + `Entities` **aprobados**.
   - **Rational Rose `DS` (`Trabajo aplicativo DS.mdl`):** **`UC04` y `UC10` aprobados** (con orden MVC estricto, `SelfMessView`, barra única en entidades, `cancelarOperacion` vía controlador y parámetros tipados).
2. **Diseño (`Design Model` — Fase actual):**
   - **Manual oficial creado:** `_Sistema/Reglas-Diseno-PlantUML-Rose.md` (basado en `TrabajoOficialZegarra.mdl` + reglas CARTOLIMP: `shared` con `pantalla` / `ViewMensajeSistema` / `mensajeConfirmacion`, `index*` / `get*` / `control*` / `form*`, `POST`, `Modelos` heredando de `conexion` con `conectarBD()` / `desconectarBD()`, parámetros literales concretos y trazabilidad 1:1 con `DA` y `DS` Análisis).
   - **Primer entregable de Diseño completado (PlantUML):**
     - `UC10. Gestionar productos de limpieza/DS. Diseno.txt`
     - `UC10. Gestionar productos de limpieza/DO. Diseno.txt`
3. **Próximo paso (en otro ordenador):**
   - Revisar `UC10/DS. Diseno.txt` y `UC10/DO. Diseno.txt` en PlantUML, y continuar generando los diagramas de Diseño (`DS. Diseno.txt` y `DO. Diseno.txt`) para los demás Casos de Uso (ej. `UC04`) y su traslado a `Design Model` en Rose.
