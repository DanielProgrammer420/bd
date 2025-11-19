La **Tercera Forma Normal (3FN)** es el siguiente paso en la **normalización de bases de datos relacionales**, después de la Primera y la Segunda Forma Normal. Su objetivo es **eliminar las dependencias transitivas** y garantizar que **todos los atributos no clave dependan únicamente de la clave primaria**, y no de otros atributos no clave.

---

## 🔹 ¿Qué es una dependencia transitiva?

Una **dependencia transitiva** ocurre cuando:

> **A → B** y **B → C**,  
> entonces **A → C** de forma **indirecta** (transitiva).

En una tabla, esto significa que un atributo no clave (**C**) depende de otro atributo no clave (**B**), y no directamente de la clave primaria (**A**).

La **3FN prohíbe este tipo de dependencia**.

---

## 🔸 Regla formal de la Tercera Forma Normal

Una tabla está en **Tercera Forma Normal** si:

1. Ya está en **Segunda Forma Normal (2FN)**, **y**
2. **Ningún atributo no clave depende de otro atributo no clave** (es decir, no hay dependencias transitivas).

> ✅ **Todos los atributos no clave deben depender **solo y directamente** de la clave primaria.**

---

## 🔸 Ejemplo práctico: tabla NO normalizada (violación de 3FN)

Imagina esta tabla `edificio`:

| edificio_id | nombre          | zona_id | nombre_zona | superficie_km2 |
|-------------|------------------|----------|--------------|----------------|
| 1           | EDIFICIO-1       | 3        | Centro       | 150            |
| 2           | EDIFICIO-2       | 3        | Centro       | 150            |
| 3           | EDIFICIO-3       | 5        | Norte        | 200            |

### Análisis:
- **Clave primaria**: `edificio_id` (simple).
- **Atributos no clave**: `nombre`, `zona_id`, `nombre_zona`, `superficie_km2`.

### ¿Hay dependencia transitiva?
- `edificio_id` → `zona_id` → ✅ (dependencia directa, correcta).
- `zona_id` → `nombre_zona` y `superficie_km2` → ✅ (esto es lógico: la zona define su nombre y superficie).
- Pero…  
  `edificio_id` → `nombre_zona` → ❌ **dependencia transitiva**.  
  `edificio_id` → `superficie_km2` → ❌ **dependencia transitiva**.

👉 Esto **viola la 3FN**, porque `nombre_zona` y `superficie_km2` no dependen directamente del edificio, sino de la **zona**.

### Problemas que causa:
- **Redundancia**: si hay 50 edificios en la zona "Centro", se repite "Centro" y "150" 50 veces.
- **Anomalías**:
  - **Actualización**: si cambia la superficie de la zona "Centro", hay que actualizar decenas de filas.
  - **Inserción**: no puedes registrar una nueva zona si no hay edificios asignados.
  - **Eliminación**: si eliminas el último edificio de una zona, pierdes la información de la zona.

---

## 🔸 Paso 1: Aplicar la Tercera Forma Normal

Dividimos la tabla en dos:

### Tabla 1: `edificio`
| edificio_id | nombre          | zona_id |
|-------------|------------------|----------|
| 1           | EDIFICIO-1       | 3        |
| 2           | EDIFICIO-2       | 3        |
| 3           | EDIFICIO-3       | 5        |

- **PK**: `edificio_id`
- Todos los atributos no clave (`nombre`, `zona_id`) dependen **directamente** de la PK → ✅ 3FN.

---

### Tabla 2: `zona`
| zona_id | nombre_zona | superficie_km2 |
|---------|--------------|----------------|
| 3       | Centro       | 150            |
| 5       | Norte        | 200            |

- **PK**: `zona_id`
- Los atributos dependen de la PK → ✅ 3FN.

---

### Relación:
- `edificio.zona_id` → FK a `zona.zona_id`.

---

## 🔹 ¿Por qué esto cumple la 3FN?

- En `edificio`, ya no hay información sobre la zona, solo su **identificador**.
- Toda la información sobre la zona está en su propia tabla.
- **No hay dependencias transitivas**: cada atributo no clave depende **solo de su propia clave primaria**.

---

## 🔸 Caso especial: ¿y si la clave primaria es compuesta?

La regla sigue siendo la misma: **ningún atributo no clave puede depender de otro atributo no clave**, incluso si la PK es compuesta.

Ejemplo:
```sql
-- MAL (violación de 3FN)
matrícula(
    alumno_id,
    curso_id,
    nombre_alumno,  -- ❌ depende de alumno_id, no de (alumno_id, curso_id)
    nombre_curso    -- ❌ depende de curso_id, no de la PK compuesta
)
```

✅ **Solución**: separar en `alumno`, `curso` y `matrícula`.

---

## 🔸 Resumen visual

| Forma Normal | Requisito |
|-------------|----------|
| **1FN** | Valores atómicos, sin grupos repetidos. |
| **2FN** | 1FN + **no hay dependencias parciales** (atributos no clave dependen de **toda** la PK compuesta). |
| **3FN** | 2FN + **no hay dependencias transitivas** (atributos no clave no dependen de otros atributos no clave). |

---

## ✅ Conclusión

La **Tercera Forma Normal** elimina la **redundancia lógica** causada por atributos que describen **otras entidades** (como la zona, el tipo de gasto, la categoría, etc.).

> **Clave para detectarla**:  
> Si ves un atributo que **describe algo que ya tiene su propia identidad** (como `nombre_zona`, `nombre_tipo_gasto`, `capital_provincia`), **probablemente deba ir en su propia tabla**.

Este principio es esencial en el **modelo "Consorcio 2025"**, donde ya se aplicó correctamente:  
- `edificio` → solo tiene `zona_id`  
- `zona` → tiene `nombre` y otros atributos propios.

¿Te gustaría ver cómo se aplicaría la 3FN a otra parte del modelo de consorcio?
