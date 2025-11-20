### **Explicación Paso a Paso de una Función Escalar en SQL Server**  
**Ejemplo**: Función para calcular el **precio con impuesto** (IVA) sobre un monto.  

---

#### **1. Código de la función con comentarios detallados**  
```sql
CREATE FUNCTION dbo.CalcularPrecioConIVA (  -- 👉 Define el nombre y esquema (dbo)
    @monto DECIMAL(10,2),                 -- 👉 Parámetro de entrada: monto base
    @tasaIVA DECIMAL(5,2) = 0.16          -- 👉 Parámetro con valor por defecto (16%)
)
RETURNS DECIMAL(10,2)                     -- 👉 Tipo de dato del valor devuelto
AS
BEGIN                                     -- 👉 Inicio del bloque de lógica
    DECLARE @precioConIVA DECIMAL(10,2);  -- 👉 Variable interna para cálculos
    
    -- Validación básica: si el monto es negativo, devuelve NULL
    IF @monto < 0
        RETURN NULL;
    
    -- Cálculo del precio con IVA
    SET @precioConIVA = @monto * (1 + @tasaIVA);
    
    RETURN @precioConIVA;                 -- 👉 Devuelve el resultado final
END;
```

---

#### **2. Desglose de cada componente**  
| **Componente**               | **Explicación**                                                                 |
|------------------------------|---------------------------------------------------------------------------------|
| `CREATE FUNCTION dbo.Nombre` | - **`dbo`**: Esquema predeterminado (evita conflictos de nombres).<br>- **`Nombre`**: Identificador único. |
| `@parametro TIPO = valor`     | - **Parámetros**: Valores de entrada.<br>- **`= 0.16`**: Valor por defecto si no se especifica. |
| `RETURNS TIPO`               | Define el tipo de dato que devolverá la función (ej: `DECIMAL`, `VARCHAR`, `INT`). |
| `BEGIN...END`                | Bloque que encapsula la lógica T-SQL. Obligatorio en funciones escalares.     |
| `DECLARE @variable TIPO`     | Variables internas para cálculos intermedios (no persisten fuera de la función). |
| `IF...RETURN NULL`           | Validación básica para evitar resultados inválidos (buenas prácticas).        |
| `RETURN valor`               | **¡Solo un valor!** Las funciones escalares devuelven siempre un único valor. |

---

#### **3. Flujo de ejecución**  
1. **Llamada a la función**:  
   ```sql
   SELECT dbo.CalcularPrecioConIVA(100.00, DEFAULT); -- Usa el valor por defecto (0.16)
   ```
2. **Parámetros recibidos**:  
   - `@monto = 100.00`  
   - `@tasaIVA = 0.16` (por defecto).  
3. **Validación**:  
   - `@monto` no es negativo → continúa.  
4. **Cálculo**:  
   - `$100 * (1 + 0.16) = 116.00$`.  
5. **Resultado**:  
   - Devuelve `116.00` como `DECIMAL(10,2)`.  

---

#### **4. Errores comunes y cómo evitarlos**  
❌ **Error 1**: Usar funciones no deterministas sin `SCHEMABINDING`.  
```sql
-- ❌ Incorrecto: GETDATE() puede causar problemas de rendimiento y caché.
CREATE FUNCTION dbo.FechaActual()
RETURNS DATE
AS
BEGIN
    RETURN GETDATE(); -- ¡No es determinista!
END;
```
✅ **Solución**:  
```sql
-- ✅ Correcto: Marcar como no determinista explícitamente (aunque no es obligatorio en SQL Server).
CREATE FUNCTION dbo.FechaActual()
RETURNS DATE
WITH SCHEMABINDING -- Restringe cambios en objetos dependientes
AS
BEGIN
    RETURN CONVERT(DATE, GETDATE());
END;
```

❌ **Error 2**: Intentar modificar datos.  
```sql
-- ❌ ¡Nunca hacer esto! Las funciones no permiten efectos secundarios.
CREATE FUNCTION dbo.ErrorEjemplo()
RETURNS INT
AS
BEGIN
    UPDATE Productos SET Precio = Precio * 1.1; -- Error de sintaxis: no se permite DML.
    RETURN 1;
END;
```

---

#### **5. Buenas prácticas destacadas**  
1. **Siempre usa esquema (`dbo.`)**:  
   - Evita ambigüedad y mejora rendimiento (SQL Server no busca en múltiples esquemas).  
2. **Valida parámetros de entrada**:  
   - Ejemplo: `IF @monto <= 0 RETURN NULL` para evitar resultados inválidos.  
3. **Evita funciones escalares en consultas masivas**:  
   - **Mal**: `SELECT *, dbo.CalcularPrecioConIVA(Precio) FROM Productos` (ejecuta fila por fila).  
   - **Bien**: Usa `CROSS APPLY` con una iTVF o cálculos directos en el `SELECT`.  
4. **Documenta en el código**:  
   ```sql
   /* 
   Descripción: Calcula el precio con IVA.
   Autor: TuNombre
   Fecha: 2025-11-21
   Notas: Si @monto es negativo, devuelve NULL.
   */
   ```

---

#### **6. Ejemplo de uso en una consulta**  
```sql
-- Tabla de ejemplo
CREATE TABLE Productos (
    ID INT PRIMARY KEY,
    Nombre VARCHAR(50),
    PrecioBase DECIMAL(10,2)
);

INSERT INTO Productos VALUES 
(1, 'Laptop', 800.00),
(2, 'Mouse', 25.50);

-- Llamada a la función
SELECT 
    Nombre,
    PrecioBase,
    dbo.CalcularPrecioConIVA(PrecioBase, 0.12) AS PrecioConIVA_12%, -- Tasa personalizada
    dbo.CalcularPrecioConIVA(PrecioBase, DEFAULT) AS PrecioConIVA_16% -- Usa valor por defecto
FROM Productos;
```

**Resultado**:  
| Nombre  | PrecioBase | PrecioConIVA_12% | PrecioConIVA_16% |  
|---------|------------|-------------------|-------------------|  
| Laptop  | 800.00     | 896.00            | 928.00            |  
| Mouse   | 25.50      | 28.56             | 29.58             |  

---

#### **7. Resumen para tu exposición**  
**Puntos clave a destacar**:  
- **Propósito**: Reutilizar lógica matemática o de negocio sin repetir código.  
- **Restricciones**:  
  - No admiten operaciones DML (`INSERT`, `UPDATE`, `DELETE`).  
  - No pueden llamar a stored procedures.  
- **Rendimiento**:  
  - Rápida para cálculos simples en pocas filas.  
  - **Peligroso** en consultas con miles de filas (usa iTVF + `APPLY` en su lugar).  
- **Casos de uso real**:  
  - Cálculo de impuestos, conversiones de moneda, formateo de textos.  

**Frase para concluir**:  
*"Las funciones escalares son herramientas poderosas para encapsular lógica atómica, pero su uso inadecuado puede convertirlas en cuellos de botella. ¡Domina cuándo y cómo usarlas!"* 😊
