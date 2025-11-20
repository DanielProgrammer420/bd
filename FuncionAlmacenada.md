**C.R.A.F.T. — Funciones Almacenadas en SQL Server**  

---

### **Teoría detallada**  
**Definición**:  
Las funciones almacenadas en SQL Server son módulos reutilizables que encapsulan lógica T-SQL y devuelven un valor (escalar o tabla). A diferencia de los *stored procedures*, **no permiten efectos secundarios** (ej: modificaciones en tablas permanentes, excepto variables de tabla en TVFs multi-instrucción) y pueden usarse en cláusulas `SELECT`, `WHERE`, o `JOIN`.  

**Tipos**:  
1. **Funciones Escalares (Scalar-valued Functions):**: Devuelven un único valor (ej: `INT`, `VARCHAR`).
   - Input: 0 o más parámetros.
   - Output: Un único valor de datos (int, varchar, decimal, etc.).
   - Uso: Cálculos matemáticos, manipulación de cadenas, conversiones.
   - Nota de rendimiento: Tienden a ser lentas si se usan en la cláusula WHERE o SELECT sobre millones de filas, ya que se ejecutan fila por fila (Row By Agonizing Row).
   
   **Ejemplo:**  
   ```sql  
   CREATE FUNCTION dbo.CalcularIVA (@monto DECIMAL(10,2))  
   RETURNS DECIMAL(10,2)  
   AS  
   BEGIN  
      RETURN @monto * 0.16;  
   END;  
   ```
   
2. **Funciones con Valor de Tabla en Línea (Inline Table-Valued Functions - iTVF):**: Devuelven una tabla mediante una única consulta `SELECT`. Actúan como vistas parametrizadas.

   - Input: 0 o más parámetros.
   - Output: Una tabla virtual (table).
   - Estructura: Contienen una única sentencia SELECT. No tienen cuerpo BEGIN...END.
   - Ventaja: Son "transparentes" para el motor. SQL Server expande la definición de la función dentro de la consulta principal, permitiendo una optimización excelente. Son, en esencia, "Vistas parametrizadas".
   
   **Ejemplo:**  
   ```sql  
   CREATE FUNCTION dbo.EmpleadosPorDepartamento (@deptID INT)  
   RETURNS TABLE  
   AS  
   RETURN SELECT * FROM Empleados WHERE DepartamentoID = @deptID;  
   ```  

3. **Multi-Statement Table-Valued Functions (mTVF)**: Devuelven una tabla construida mediante múltiples instrucciones (usa variables de tabla).

  - Input: 0 o más parámetros.
  - Output: Una variable tipo tabla que debes declarar explícitamente.
  - Estructura: Tienen un bloque BEGIN...END. Permiten lógica compleja (IF, WHILE, declaraciones de variables) para llenar la tabla de retorno.
  - Desventaja: Rendimiento inferior a las iTVF, ya que usan tempdb y el estimador de cardinalidad a veces falla al predecir cuántas filas devolverán.

   **Ejemplo:**  
   ```sql
     
   CREATE FUNCTION dbo.VentasConsolidadas (@anio INT)  
   RETURNS @resultado TABLE (Producto VARCHAR(50), Total DECIMAL(10,2))  
   AS  
   BEGIN  
      INSERT INTO @resultado  
      SELECT Producto, SUM(Monto) FROM Ventas  
      WHERE YEAR(Fecha) = @anio  
      GROUP BY Producto;  
      RETURN;  
   END;  
   ```  

**Restricciones clave**:  
- No admiten transacciones explícitas (`BEGIN TRANSACTION`).  
- No pueden llamar a stored procedures (excepto `sp_executesql` en ciertos contextos).  
- No permiten modificaciones en tablas permanentes (solo en variables de tabla).  
- No pueden usar funciones no deterministas (ej: `GETDATE()`) sin marcarlas explícitamente como `WITH SCHEMABINDING`.  

**Diferencias con Stored Procedures**:  
| Característica          | Función                          | Stored Procedure               |  
|-------------------------|----------------------------------|--------------------------------|  
| Valor de retorno        | Obligatorio (escalar o tabla)    | Opcional (parámetros OUTPUT)   |  
| Efectos secundarios     | No (excepto variables de tabla) | Sí (INSERT/UPDATE/DELETE)      |  
| Uso en consultas        | Sí (ej: `SELECT dbo.Func()`)     | No (requiere `EXEC`)           |  

---

### **Ejemplo paso a paso**  

1. **Función Escalar**: Función para calcular el precio con impuesto (IVA) sobre un monto.  
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

   **Desglose de cada componente**
    | **Componente**               | **Explicación**                                                                 |
    |------------------------------|---------------------------------------------------------------------------------|
    | `CREATE FUNCTION dbo.Nombre` | - **`dbo`**: Esquema predeterminado (evita conflictos de nombres).<br>- **`Nombre`**: Identificador único. |
    | `@parametro TIPO = valor`    | - **Parámetros**: Valores de entrada.<br>- **`= 0.16`**: Valor por defecto si no se especifica. |
    | `RETURNS TIPO`               | Define el tipo de dato que devolverá la función (ej: `DECIMAL`, `VARCHAR`, `INT`). |
    | `BEGIN...END`                | Bloque que encapsula la lógica T-SQL. Obligatorio en funciones escalares.     |
    | `DECLARE @variable TIPO`     | Variables internas para cálculos intermedios (no persisten fuera de la función). |
    | `IF...RETURN NULL`           | Validación básica para evitar resultados inválidos (buenas prácticas).        |
    | `RETURN valor`               | **¡Solo un valor!** Las funciones escalares devuelven siempre un único valor. |


3. **iTVF**:  
   ```sql  
   CREATE FUNCTION dbo.ClientesActivos (@pais VARCHAR(50))  
   RETURNS TABLE  
   AS  
   RETURN (  
      SELECT ClienteID, Nombre, Email  
      FROM Clientes  
      WHERE Pais = @pais AND Activo = 1  
   );  
   ```  
   **Uso**:  
   ```sql  
   SELECT * FROM dbo.ClientesActivos('México');  
   ```  

---

### **Errores comunes**  
1. **Violación de restricciones**:  
   ```sql  
   CREATE FUNCTION dbo.ErrorEjemplo()  
   RETURNS INT  
   AS  
   BEGIN  
      UPDATE Empleados SET Salario = Salario * 1.1; -- ❌ Error: No se permiten DML en funciones escalares.  
      RETURN 1;  
   END;  
   ```  

2. **Olvidar `BEGIN/END` en mTVF**:  
   ```sql  
   CREATE FUNCTION dbo.FuncIncorrecta()  
   RETURNS @tabla TABLE (ID INT)  
   AS  
   INSERT INTO @tabla VALUES (1); -- ❌ Falta BEGIN/END.  
   RETURN;  
   ```  

3. **Usar `GETDATE()` sin `SCHEMABINDING`**:  
   ```sql  
   CREATE FUNCTION dbo.FechaActual()  
   RETURNS DATETIME  
   AS  
   BEGIN  
      RETURN GETDATE(); -- ❌ Advertencia: función no determinista.  
   END;  
   ```  

---

### **Buenas prácticas**  
1. **Prefiere iTVF sobre mTVF**: El optimizador de SQL Server puede integrar iTVFs mejor en planes de ejecución.  
2. **Evita funciones escalares en `SELECT` masivos**: Causan rendimiento pobre (ejecución fila por fila). Usa `APPLY` con iTVFs en su lugar.  
3. **Califica siempre con esquema**: `dbo.NombreFuncion` evita ambigüedad y mejora rendimiento.  
4. **Documenta con comentarios**: Explica el propósito, parámetros y restricciones.  
5. **Prueba el rendimiento**: Usa `SET STATISTICS IO ON` para detectar cuellos de botella.  

---

### **Ejercicios**  
**Básicos**:  
1. Crea una función escalar que convierta grados Celsius a Fahrenheit.  
2. Escribe una iTVF que liste productos con stock menor a un valor dado.  
3. Modifica una función existente para que devuelva `NULL` si el parámetro es negativo.  

**Intermedios**:  
1. Diseña una mTVF que devuelva el top 5 de ventas por mes usando una variable de tabla.  
2. Crea una función que valide si un email tiene formato correcto (usa `CHARINDEX` y `LIKE`).  
3. Combina una iTVF con `CROSS APPLY` para calcular impuestos por producto.  

**Avanzados**:  
1. Implementa una función determinista que calcule días hábiles entre dos fechas (sin usar bucles).  
2. Crea una mTVF con manejo de errores (`TRY...CATCH`) para procesar datos corruptos.  
3. Optimiza una función escalar lenta reemplazándola con una iTVF y `APPLY`.  

---

### **Preguntas tipo examen + soluciones**  
1. **¿Por qué no se puede usar `INSERT` en una función escalar?**  
   *R: Las funciones escalares no permiten efectos secundarios que modifiquen el estado de la base de datos.*  

2. **¿Cuál es la ventaja de una iTVF frente a una vista?**  
   *R: Las iTVF aceptan parámetros, permitiendo lógica dinámica, mientras las vistas son estáticas.*  

3. **Corrige el error en este código**:  
   ```sql  
   CREATE FUNCTION dbo.Error()  
   RETURNS TABLE  
   AS  
   BEGIN  
      RETURN SELECT * FROM Pedidos;  
   END;  
   ```  
   *R: Las iTVF no usan `BEGIN/END`. La sintaxis correcta es:*  
   ```sql  
   CREATE FUNCTION dbo.Error()  
   RETURNS TABLE  
   AS  
   RETURN SELECT * FROM Pedidos;  
   ```  

---

### **Resumen final para exponer**  
**Slide 1 (Introducción)**:  
- Definición: "Módulos reutilizables que devuelven valores sin efectos secundarios."  
- Tipos: Escalares (un valor), iTVF (vista parametrizada), mTVF (lógica compleja con tabla).  

**Slide 2 (Casos de uso)**:  
- **Escalares**: Cálculos matemáticos (impuestos, conversiones).  
- **iTVF/mTVF**: Reportes parametrizados, capas de abstracción para aplicaciones.  

**Slide 3 (Mejores prácticas)**:  
- Evita funciones escalares en consultas masivas.  
- Usa `SCHEMABINDING` para funciones deterministas.  
- Siempre califica con el esquema (`dbo.`).  

**Slide 4 (Demo rápida)**:  
1. Crear función escalar para descuentos.  
2. Usar iTVF en un `JOIN` con la tabla `Pedidos`.  

**Slide 5 (Conclusión)**:  
- Ventajas: Modularidad, reutilización, integración en consultas.  
- Advertencia: Rendimiento crítico en funciones escalares.  
- ¡Preguntas!  

---  
**¡Listo para tu exposición!** Combina teoría rigurosa con ejemplos prácticos y destaca las diferencias clave frente a stored procedures. 😊
