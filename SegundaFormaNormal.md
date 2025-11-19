Para entenderla bien, primero debes dominar la **Primera Forma Normal (1FN)**. La 2FN **solo se aplica cuando la clave primaria está compuesta por dos o más atributos**.

---

## 🔹 ¿Qué es una dependencia parcial?

Una **dependencia parcial** ocurre cuando un atributo **no clave** depende **solo de una parte** de la clave primaria compuesta, y no de toda ella.

> 📌 **Regla de la 2FN**:  
> **Toda columna que no forme parte de la clave primaria debe depender de la clave primaria completa, no de una parte de ella.**

---

## 🔸 Ejemplo práctico: tabla NO normalizada (violación de 2FN)

Imagina esta tabla `pedido`:

| pedido_id | producto_id | nombre_producto | cantidad | precio_unitario |
|-----------|-------------|------------------|----------|------------------|
| 100       | P01         | Lápiz            | 5        | 20.00            |
| 100       | P02         | Cuaderno         | 2        | 150.00           |
| 101       | P01         | Lápiz            | 3        | 20.00            |

### Análisis:
- **Clave primaria compuesta**: `(pedido_id, producto_id)`  
  → Necesaria porque un pedido puede tener varios productos.
- **Atributos no clave**: `nombre_producto`, `cantidad`, `precio_unitario`.

### ¿Hay dependencia parcial?
- `cantidad` depende de **todo el pedido**: de *ese pedido* y *ese producto* → ✅ **dependencia total**.
- Pero `nombre_producto` y `precio_unitario` **dependen solo de `producto_id`**, **no de `pedido_id`**.

👉 Esto es una **violación de la 2FN**.

---

## 🔸 Paso 1: Aplicar la Segunda Forma Normal

Dividimos la tabla en dos:

### Tabla 1: `detalle_pedido`  
(Contiene solo lo que depende de **toda la clave**)

| pedido_id | producto_id | cantidad |
|-----------|-------------|----------|
| 100       | P01         | 5        |
| 100       | P02         | 2        |
| 101       | P01         | 3        |

- **PK**: `(pedido_id, producto_id)`
- Todos los atributos no clave (`cantidad`) dependen de **toda la clave** → ✅ 2FN cumplida.

---

### Tabla 2: `producto`

| producto_id | nombre_producto | precio_unitario |
|-------------|------------------|------------------|
| P01         | Lápiz            | 20.00            |
| P02         | Cuaderno         | 150.00           |

- **PK**: `producto_id`
- Los atributos dependen de la clave completa → ✅ 2FN (y también 3FN).

---

### ✅ Resultado:
- **No hay dependencias parciales**.
- **Evitamos redundancia**: si el precio del lápiz cambia, solo se actualiza en **un lugar**.
- **Evitamos anomalías**:
  - **Actualización**: no hay que cambiar el precio en 100 filas, solo en una.
  - **Inserción**: puedo cargar un nuevo producto aunque no esté en ningún pedido.
  - **Eliminación**: si se elimina el último pedido de un producto, no pierdo la info del producto (si lo guardo en otra tabla).

---

## 🔹 ¿Cuándo **NO** se aplica la 2FN?

Si la **clave primaria es simple** (un solo atributo), **la tabla ya está en 2FN** automáticamente.

Ejemplo:

| persona_id | nombre | edad |
|------------|--------|------|
| 1          | Ana    | 30   |

- PK = `persona_id` (simple).
- `nombre` y `edad` dependen de toda la clave (que es solo `persona_id`).
- **No hay dependencia parcial posible** → ✅ ya está en 2FN.

---

## 🔸 Resumen visual

| Forma Normal | Requisito |
|-------------|----------|
| **1FN** | Valores atómicos, sin grupos repetidos. |
| **2FN** | **1FN +** ningún atributo no clave depende parcialmente de una clave primaria compuesta. |

---

## ✅ Conclusión

La **Segunda Forma Normal** elimina la **redundancia lógica** que surge cuando información de una entidad (como `producto`) se repite dentro de otra entidad compuesta (como `pedido`).

> **Clave para detectarla**:  
> Si tienes una PK compuesta y notas que algunos datos se repiten en varias filas (ej. nombre y precio del producto), probablemente estés violando la 2FN.

¿Te gustaría ver cómo se aplica esto al modelo **"Consorcio 2025"**?
