

## 🔹 ¿Qué significa "Primera Forma Normal"?

Una tabla está en **Primera Forma Normal** si cumple estas **tres condiciones**:

1. **Cada columna contiene valores atómicos** (indivisibles).
2. **No hay grupos repetidos ni tablas anidadas dentro de una celda**.
3. **Cada fila es única** (identificada por una clave primaria).

> ✅ **Valor atómico**: un dato que no se puede dividir más en el contexto de la base de datos.  
> ❌ **Valor no atómico**: una lista, un conjunto, un arreglo, o varios valores en una misma celda.

---

## 🔸 Ejemplo práctico: **tabla NO normalizada**

Imagina que guardas información de **edificios y sus gastos**, pero de forma incorrecta:

| edificio_id | nombre          | gastos                     |
|-------------|------------------|----------------------------|
| 1           | EDIFICIO-1       | Limpieza, Seguridad, Luz   |
| 2           | EDIFICIO-2       | Agua, Limpieza             |

⚠️ **Problema**: la columna `gastos` contiene **múltiples valores en una sola celda** → **NO está en 1FN**.

---

## 🔸 Paso 1: Aplicar la **Primera Forma Normal**

Dividimos los valores no atómicos en **filas separadas**:

| edificio_id | nombre          | tipo_gasto    |
|-------------|------------------|----------------|
| 1           | EDIFICIO-1       | Limpieza       |
| 1           | EDIFICIO-1       | Seguridad      |
| 1           | EDIFICIO-1       | Luz            |
| 2           | EDIFICIO-2       | Agua           |
| 2           | EDIFICIO-2       | Limpieza       |

✅ Ahora:
- Cada celda tiene un **único valor** (atómico).
- No hay listas ni grupos repetidos.
- Cada fila representa una **asociación única**: un edificio + un tipo de gasto.

> 🔍 **Nota**: esta tabla aún **no tiene clave primaria única**. Para cumplir del todo con la 1FN, debemos definir una PK.  
> Posible solución: usar `(edificio_id, tipo_gasto)` como clave primaria compuesta.

---

## 🔸 Otro ejemplo: fechas o teléfonos múltiples

### ❌ Tabla no normalizada:
| persona_id | nombre     | telefonos               |
|------------|------------|--------------------------|
| 101        | Ana López  | 11-1234-5678, 11-8765-4321 |

### ✅ Tabla en 1FN:
| persona_id | nombre     | telefono        |
|------------|------------|------------------|
| 101        | Ana López  | 11-1234-5678     |
| 101        | Ana López  | 11-8765-4321     |

Ahora cada teléfono es un registro independiente → **estructura relacional correcta**.

---

## 🔹 ¿Por qué es importante la 1FN?

1. **Evita ambigüedades**: no sabes cuántos valores hay en una celda.
2. **Permite consultas precisas**: puedes filtrar por "Limpieza" sin tratar cadenas.
3. **Habilita relaciones**: puedes vincular `tipo_gasto` con una tabla de catálogo.
4. **Es requisito para 2FN, 3FN, etc.**: no puedes normalizar más si no estás en 1FN.

---

## 🔸 ¿Cómo saber si tu tabla está en 1FN?

Haz estas preguntas:
- ¿Hay alguna celda con **más de un valor** (separado por comas, punto y coma, etc.)?
- ¿Hay **columnas repetidas**, como `telefono1`, `telefono2`, `telefono3`?
- ¿Puedo dividir un valor y seguir teniendo sentido en el modelo?

→ Si la respuesta es **sí** a alguna, **NO estás en 1FN**.

---

## ✅ Conclusión

La **Primera Forma Normal** es simple pero esencial:  
> **"Una celda = un valor".**

Al cumplirla, pasas de una hoja de cálculo desordenada a una **estructura relacional sólida**, lista para relacionarse con otras tablas y soportar consultas complejas sin errores.

¿Quieres que te muestre cómo se aplica esto al modelo **"Consorcio 2025"**?
