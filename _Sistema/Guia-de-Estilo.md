---
tipo: sistema
proyecto: ADS-2026 II
estado: borrador
fecha: 2026-10-04
---
# Guía de Estilo y Reglas del Profesor

Esta guía contiene las reglas extraídas de las indicaciones (Tipo C) y los modelos (Tipo B) del profesor.

## 1. Nomenclatura y Estructura
- **Casos de Uso del Sistema**: `UCXX. [Nombre del Caso de Uso]` (ej. `UC01. Autenticar usuario`).
- **Estructura de Carpetas**: Cada caso de uso debe tener su propia carpeta `UCXX. Nombre` que contiene los diagramas relacionados.

> 📘 **Manuales detallados:**
> - Diagramas de **actividades** → `Reglas-Diagramas-Actividades.md` (R1–R12 + checklist, validado contra UC06/UC12/UC04).
> - Diagramas de **secuencia de análisis** (robustez BCE) → `Reglas-Diagramas-Secuencia.md` (S1–S11 + checklist, validado contra DS_Gestionar proveedores y UC10).
> - Diagramas de **objetos de análisis** (robustez BCE) → `Reglas-Diagramas-Objetos.md` (O1–O9 + checklist, validado contra UC10 y UC06; trazabilidad triple con actividades y secuencia).
> - **Diagrama de Entidades Consolidado** → `02-Entregables/D. Entidades.txt` (se actualiza obligatoriamente cada vez que se crea o modifica un DO, según regla O9).
>
> Esta sección es el resumen general de actividades.

## 2. Diagrama de Actividades por Caso de Uso
- **Carril del Actor**: La regla más estricta exige que en el carril (swimlane) del actor **NO** haya ninguna actividad, **SOLO** el punto inicial (`start`). A partir de ahí, el sistema se encarga de todo.
- **Restricciones de Código PlantUML**: 
  - Usar siempre `scale 0.7` (o menor) después del tag de inicio para evitar el crash de límite de tamaño (`PLANTUML_LIMIT_SIZE`).
  - **Prohibido el uso de bucles** (`repeat...while`). El profesor usa arquitecturas lineales. Para iteraciones de UI, usar bifurcaciones anidadas (`switch/case` y `if/else`) que terminen indefectiblemente en un comando `stop`. El usuario es quien "reinicia" el flujo en la UI.
  - Toda rama terminal, ya sea de éxito o error, debe cerrarse con la instrucción `stop`. No dejar ramas sueltas.
  - En los nodos de convergencia múltiple (después de un `endif` o `endswitch` donde se juntan varias flechas), la siguiente instrucción obligatoria debe ser una actividad (`:Acción;`), nunca otro rombo de condición directo.
- **Queries/Consultas SQL**: Deben incluirse explícitamente mediante `note right:` debajo de la actividad. Los comandos DML (`INSERT INTO`, `UPDATE`, `SELECT`) deben ir completos con los campos implicados.
- **Flujo Markdown Sincronizado**: Cada CUS debe venir acompañado de un archivo `Flujo-UCXX.md` que explique la "Arquitectura de la Interfaz". Las vistas en este archivo deben coincidir 1:1 con las declaradas en el diagrama PlantUML, sin utilizar terminologías obsoletas (como "Pasos").
- **Herramienta**: Todo diagrama se realiza utilizando código **PlantUML** estricto, respetando los colores y diseño base del profesor.

## 3. Reglas Estrictas de Arquitectura y Casos de Uso
- **Unicidad de Actores**: Un Caso de Uso (CUS) **NO** puede ser activado por dos actores distintos. Siempre debe haber un solo actor primario disparador.
- **Validación de Botones (Sin Sesión)**: No incluir validaciones de sesión (Auth) en los diagramas de cada caso de uso. Asumir que el entorno ya está validado de forma global.
- **¿Cuándo usar `isset(btnX)`?**: Solo se valida el botón con `isset()` cuando la acción implica una **transición entre interfaces** o el acceso a una **capa de lógica o datos**. NO se usa para acciones de búsqueda/display dentro de la misma vista. Criterio:

  | Tipo de Acción | ¿Usa `isset()`? |
  |---|---|
  | Búsqueda / filtro en la misma vista (`btnBuscar`) | ❌ No |
  | Navegación a otro formulario/panel | ✅ Sí |
  | Modificación de sesión temporal (carrito) | ✅ Sí |
  | Operación DML (INSERT / UPDATE / DELETE) | ✅ Sí |
  | Confirmación de alerta antes de continuar | ✅ Sí |
- **Relaciones Simples**: Mantener los CUS separados e independientes. Evitar el uso excesivo o complejo de relaciones `<<include>>` o `<<extend>>`.
- **Sin APIs Externas**: No se utilizarán APIs externas (ej. RENIEC/SUNAT) para no complejizar la arquitectura MVC estricta del proyecto. Todo registro será manual.
- **Enfoque CRUD**: Las operaciones de mantenimiento (crear, leer, actualizar, eliminar) sobre una entidad deben agruparse en un CUS tipo "Gestionar [Entidad]".
