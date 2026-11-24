---
tipo: sistema
proyecto: ADS-2026 II
estado: completado
fecha: 2026-11-24
---
# Estado del Proyecto

**Dónde quedamos:**
- **14 / 14 Casos de Uso (`UC01` – `UC14`) completados** al 100% con su flujo textual (`Flujo-UCXX.md`), Diagrama de Actividades (`D. Actividades.txt`), Diagrama de Secuencia (`DS. Secuencia.txt`) y Diagrama de Objetos (`DO. Objetos.txt`).
- **Diagrama de Entidades Consolidado (`02-Entregables/D. Entidades.txt`)** completo y sincronizado con las 18 entidades `CE_*`, atributos SQL, relaciones FK y métodos acumulados de los 14 CUS.
- **Manuales de reglas (`_Sistema/`)** depurados y alineados con los últimos estándares (`Actor → Controlador` directo sin `CI_PanelPrincipal`, ramas `Cancelar` en 1er lugar priorizando camino negativo, y `BUSCAR` sin `isset`).

**Qué cambió (Sesión actual):**
- Limpieza de rezagos en `Reglas-Diagramas-Secuencia.md` (S8, S9, S11, Checklist) y `Reglas-Diagramas-Objetos.md` (Checklist).
- Ajustes de trazabilidad en `UC10` (`btnGuardarNuevo`/`btnGuardarCambios`) y `UC06` (`CI_VistaComprobante`, `btnModificarCantidad`, validación DNI/RUC y `INNER JOIN Categoria`).
- Creación completa (`Flujo + DA + DS + DO`) de `UC01`, `UC02`, `UC03` (recuperación por pregunta y respuesta secreta), `UC05`, `UC07`, `UC09`, `UC11` y `UC14`.
- Creación de `DS` y `DO` para `UC04`, `UC08`, `UC12` y `UC13`.
- Auditoría de trazabilidad y SQL sobre los 14 CUS (validación de `estado = 'Activo'` en `UC02`, `UC03`, `UC06`, `UC12`, `UC13`, y actividad disparadora en el carril del actor en todos los `DA`).
