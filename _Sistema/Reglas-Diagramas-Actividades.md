---
tipo: sistema
proyecto: ADS-2026 II
estado: vigente
fecha: 2026-10-06
referencia: Manual de construcción de Diagramas de Actividades (PlantUML)
ejemplos_validados: UC06. Registrar venta, UC12. Registrar producción, UC04. Gestionar usuarios
---
# Manual de Construcción de Diagramas de Actividades — CARTOLIMP

Este documento es la **receta oficial** para construir el `D. Actividades.txt` (PlantUML) de cualquier CUS. Cada regla está validada contra los diagramas aprobados **UC06**, **UC12** y **UC04**. Ante cualquier duda, imitar esos tres archivos. Este manual complementa a `Guia-de-Estilo.md` (reglas generales) con el **cómo** se arma cada actividad.

> Principio rector: el **carril del actor solo contiene `start` + el clic disparador**. Desde que entra a `|Sistema|`, el sistema ejecuta absolutamente todo. Arquitectura **lineal** (sin bucles): toda rama muere en `stop`; el usuario "reinicia" el flujo desde la UI.

---

## R1 — Cabecera PlantUML (fija)
```
@startuml
scale 0.7
skinparam ActivityBackgroundColor #FFFFE0
skinparam ActivityBorderColor #8B0000
skinparam ArrowColor #8B0000
skinparam PartitionBorderColor #000000
skinparam shadowing false
skinparam DefaultFontName Arial

title Diagrama de Actividades - UCXX: Nombre
```
- `scale 0.7` por defecto. Si el diagrama es largo (muchas ramas), **bajar el scale** para no romper `PLANTUML_LIMIT_SIZE` (UC06, muy extenso, usa `0.4`).

## R2 — Esqueleto de arranque (idéntico en los 3 ejemplos)
```
|Actor|
start
:Dar clic en el botón "Nombre del CUS" del Panel Principal;

|Sistema|
:Validar btnNombre;
if (isset(btnNombre)) then (False)
  :Mostrar panel <Error> "Acceso no permitido";
  stop
else (TRUE)
  :Mostrar ventana <Vista Principal> con ... (buscador vacío / grilla / área oculta);
  switch (Clic en ....)
  ...
  endswitch
endif

@enduml
```
- Si la vista principal necesita datos para pintarse (una grilla), se puede `Consultar en la BD` **antes** de `Mostrar ventana` (así lo hace UC04 con la lista de usuarios). Si no, se muestra la ventana directo (UC06/UC12).

## R3 — Un `case` por botón
- `switch (Clic en ....)` y un `case (boton <Nombre del Botón>)` por cada botón de la vista.
- **Switches anidados:** cuando un botón abre **otra vista** con sus propios botones, se abre un nuevo `switch` dentro de ese case (UC06: `<Iniciar Nueva Venta>` abre el catálogo con su propio switch de `<Buscar Ítem>`, `<Agregar al Carrito>`, etc.).

## R4 — `isset()` : CRITERIO MAESTRO → ¿hay una petición al controlador?
`isset()` es el **guardián del controlador**: la verificación que debe pasar toda acción que **viaja al controlador**. En MVC estricto **una interfaz no abre otra por sí sola**; la navegación entre interfaces la decide el controlador (vía link/POST). Por eso un botón lleva `isset()` cuando su acción implica cualquiera de:
- **(a) Transición a otra interfaz** (el controlador renderiza otra vista/formulario/modal-interfaz). ← razón principal y más olvidada.
- **(b)** Lectura de datos → un `SELECT` para precargar un registro.
- **(c)** DML (`INSERT` / `UPDATE` / `DELETE`).
- **(d)** Escritura de sesión del servidor (ej. el arreglo del carrito).

**Excepción única:** la **búsqueda/filtro que solo repinta la MISMA vista NO lleva `isset()`** aunque haga `SELECT`. No transiciona ni abre otra interfaz: solo recarga su propia grilla.

**El porqué:** si una acción que cambia de interfaz, lee, muta o escribe sesión queda sin `isset()`, se expone ese paso del controlador sin validar que la petición venga del botón legítimo. **Abrir un formulario es cambiar de interfaz → pasa por el controlador → lleva `isset()`**, aunque el formulario salga vacío. La idea de "modal vacío puro JS sin `isset`" es un patrón SPA que **contradice** el principio "una vista no habla con otra" y NO se usa en este proyecto.

**Caso de prueba (UC04):** los dos botones que abren el formulario llevan `isset()` porque ambos son transición de interfaz:
| Botón | ¿Qué hace? | ¿isset? |
|---|---|---|
| `<Buscar>` | `SELECT` que repinta la MISMA grilla | ❌ No (excepción única) |
| `<Nuevo Usuario>` | Transiciona al formulario (interfaz nueva) | ✅ Sí |
| `<Editar Usuario>` | Transiciona al formulario + `SELECT` de precarga | ✅ Sí |
| `<Guardar Nuevo>` / `<Guardar Cambios>` / `<Eliminar>` | DML | ✅ Sí |

Esto espeja a UC06, donde **todos** los botones que abren otra interfaz llevan `isset()` (`<Completar Reserva Existente>`, `<Iniciar Nueva Venta>`). Ninguna transición de interfaz queda sin guardián.

**Forma del guardián** (siempre la primera línea del case protegido, envolviendo TODO el resto):
```
case (boton <X>)
  :Validar btnX;
  if (isset(btnX)) then (False)
    :Mostrar panel <Error> "Acceso no permitido";
    stop
  else (TRUE)
    ... todo el case ...
  endif
```

## R5 — Orden interno del case (barato → caro)
1. `isset()` (si aplica, R4).
2. `:Sanitizar ...;` el input.
3. **Validaciones en cascada**, de la más barata a la más cara:
   `¿campo vacío?` → `¿hay selección en sesión/pantalla?` → `¿cantidad > 0?` → `¿existe en BD?` → `¿duplicado?` → `¿tiene receta/reservas?` → `¿stock suficiente?`
   Cada fallo cierra con: `:Mostrar panel <Error|Aviso> "mensaje concreto";` + `stop`.
4. **Rama feliz:** las operaciones (`SELECT`/`INSERT`/`UPDATE`) con su `note right:` SQL.
5. Cierre: `:Mostrar panel <Éxito> "...";` + `stop`.

## R6 — SQL en `note right:` (siempre, completo, parametrizado)
- La actividad describe (`:Registrar en la BD la cabecera de la venta;`) y **debajo** va el `note right:` con el DML real y campos completos.
- Siempre con `?` parametrizado (anti-inyección).
- Patrones confirmados: `GROUP_CONCAT` + `LEFT JOIN` para roles (UC04); `INNER JOIN` para receta (UC12); correlativo con `SELECT MAX(num)+1` (UC06).

## R7 — Verbos y vocabulario canónico
Validar, Verificar, Consultar en la BD, Registrar en la BD, Actualizar en la BD, Descontar/Sumar/Bloquear/Liberar en la BD, Generar, Cargar, Mostrar, Sanitizar, Capturar, Calcular, Recalcular, Añadir, Eliminar, Limpiar, Redireccionar, Asignar, Encriptar.

## R8 — Rombos y etiquetas
- Condición de negocio: `if (¿Existe producto?) then (SI) ... else (NO)`.
- Guardián isset: `if (isset(btnX)) then (False) ... else (TRUE)`.
- Elección de botón dentro de un formulario abierto: `if (Clic en ..) then (boton <Cancelar>) ... else (boton <Confirmar>)`.

## R9 — Paneles tipados
- `<Error>` — bloqueo / validación fallida / acceso no permitido / falla de operación.
- `<Aviso>` — registro **no encontrado** en una búsqueda puntual (cliente/producto/reserva inexistente).
- `<Confirmación>` — se le pregunta algo al usuario antes de continuar (ej. "¿emitir boleta a Público General?").
- `<Éxito>` — DML completado correctamente.

## R10 — Cierre total y limpieza de sesión
- **TODA** rama terminal (éxito o error) muere en `stop`. Ninguna queda suelta.
- `<Cancelar>` / redirecciones también cierran en `stop`.
- Tras un DML exitoso **o** al cancelar, si hubo escritura de sesión: `:Limpiar variables temporales de la sesión;` **antes** del `stop`.

## R11 — Convergencias y prohibición de bucles
- **Prohibido** `repeat`/`while`. Iteraciones de UI = `switch/case` + `if/else` que terminan en `stop`.
- Tras un `endif`/`endswitch` donde se juntan varias flechas, la siguiente instrucción **debe ser una actividad** `:Acción;`, nunca otro rombo directo.

## R12 — Consistencia de inventario (negocio CARTOLIMP)
- Reserva: `+ stock_separado`.
- Cobro de reserva: `- stock_separado` y `- stock_fisico`.
- Venta directa: `- stock_fisico`.
- Producción: `- stock_fisico` (insumos) y `+ stock_fisico` (producto terminado).
- Relación N:M (ej. roles): patrón **`DELETE` todo + re-`INSERT`**.

---

## Checklist antes de dar por cerrado un diagrama
- [ ] Carril del actor solo tiene `start` + el clic.
- [ ] `isset()` aplicado según el **criterio maestro** (R4), no por costumbre.
- [ ] Cada case: sanitiza → valida en cascada → opera → `<Éxito>`.
- [ ] Todo `SELECT`/`INSERT`/`UPDATE` tiene su `note right:` parametrizado.
- [ ] Toda rama cierra en `stop`; se limpia sesión donde corresponde.
- [ ] Sin `repeat`/`while`; convergencias seguidas de actividad.
- [ ] Balance estructural: nº de `if` == nº de `endif`, `switch` == `endswitch`.
- [ ] `Flujo-UCXX.md` describe vistas 1:1 con las del diagrama.
