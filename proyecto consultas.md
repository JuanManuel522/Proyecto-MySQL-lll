1.Como analista, quiero listar todos los productos con su empresa asociada y el precio más bajo por ciudad.

```sql
SELECT 
  co.city_id,
  p.id AS product_id,
  p.name AS product_name,
  co.id AS company_id,
  co.name AS company_name,
  cp.price
FROM companyproducts cp
JOIN companies co ON cp.company_id = co.id
JOIN products p ON cp.product_id = p.id
JOIN (
  SELECT 
    co2.city_id, 
    cp2.product_id, 
    MIN(cp2.price) AS min_price
  FROM companyproducts cp2
  JOIN companies co2 ON cp2.company_id = co2.id
  GROUP BY co2.city_id, cp2.product_id
) AS min_prices ON 
  co.city_id = min_prices.city_id AND 
  cp.product_id = min_prices.product_id AND 
  cp.price = min_prices.min_price
ORDER BY co.city_id, p.name;
```

```sql
+---------+------------+-----------------------+-------------+------------------------------+--------+
| city_id | product_id | product_name          | company_id  | company_name                 | price  |
+---------+------------+-----------------------+-------------+------------------------------+--------+
| 001     |          5 | Audífonos alámbricos  | 900111005-5 | Comidas Express S.A.S        |  15000 |
| 001     |          8 | Blusa manga larga     | 900111008-8 | Vanguardia Electrónica S.A.S | 410000 |
| 001     |          1 | Camiseta básica       | 900111001-1 | Bocados del Valle S.A.S      |  12000 |
| 001     |         10 | Cojín decorativo      | 900111010-0 | Granos del Norte Ltda        |  23000 |
| 001     |          2 | Jeans Slim Fit        | 900111002-2 | TecnoGlobal Ltda             | 450000 |
| 001     |          6 | Juego de sábanas      | 900111006-6 | ElectroMundo S.A.            | 320000 |
| 001     |          4 | Lámpara LED           | 900111004-4 | Moda Urbana S.A.S            |  85000 |
| 001     |          9 | Pantaloneta deportiva | 900111009-9 | Ropa Total S.A.S             |  70000 |
| 001     |          7 | Toalla de baño        | 900111007-7 | Sabor Llanero Ltda           |  18000 |
| 001     |          3 | Zapatillas deportivas | 900111003-3 | Alimentos El Sol S.A.        |  38000 |
+---------+------------+-----------------------+-------------+------------------------------+--------+
```

2.Como administrador, deseo obtener el top 5 de clientes que más productos han calificado en los últimos 6 meses.

```sql
SELECT 
  c.id AS customer_id,
  c.name AS customer_name,
  COUNT(DISTINCT qp.product_id) AS total_products_rated
FROM quality_products qp
JOIN customers c ON qp.customer_id = c.id
WHERE qp.daterating >= DATE_SUB(CURDATE(), INTERVAL 6 MONTH)
GROUP BY c.id, c.name
ORDER BY total_products_rated DESC
LIMIT 5;
```

```sql
+-------------+----------------+----------------------+
| customer_id | customer_name  | total_products_rated |
+-------------+----------------+----------------------+
|           1 | Juan Pérez     |                    1 |
|           2 | María Gómez    |                    1 |
|           3 | Carlos Ruiz    |                    1 |
|           4 | Ana Torres     |                    1 |
|           5 | Luis Rodríguez |                    1 |
+-------------+----------------+----------------------+
```

3.Como gerente de ventas, quiero ver la distribución de productos por categoría y unidad de medida.

```sql
SELECT 
  cat.description AS category,
  uom.description AS unit_measure,
  COUNT(DISTINCT p.id) AS total_products
FROM products p
JOIN categories cat ON p.category_id = cat.id
JOIN companyproducts cp ON p.id = cp.product_id
JOIN unitofmeasure uom ON cp.unitmeasure_id = uom.id
GROUP BY cat.description, uom.description
ORDER BY cat.description, uom.description;
```

```sql
+-----------+--------------+----------------+
| category  | unit_measure | total_products |
+-----------+--------------+----------------+
| Alimentos | Caja         |              1 |
| Alimentos | Unidad       |              4 |
| comida    | Caja         |              1 |
| comida    | Unidad       |              4 |
+-----------+--------------+----------------+
```

4.Como cliente, quiero saber qué productos tienen calificaciones superiores al promedio general.

```sql
SELECT 
  p.id,
  p.name,
  AVG(qp.rating) AS promedio_producto
FROM 
  quality_products qp
JOIN 
  products p ON qp.product_id = p.id
GROUP BY 
  p.id, p.name
HAVING 
  AVG(qp.rating) > (
    SELECT AVG(rating) FROM quality_products
  );
SELECT p.id, p.name, AVG(r.rating) AS promedio_producto
FROM products p
JOIN companyproducts cp ON p.id = cp.product_id
JOIN rates r ON cp.company_id = r.company_id
GROUP BY p.id, p.name
HAVING AVG(r.rating) > (
    SELECT AVG(rating) FROM rates
);
```

```sql
+----+-----------------------+-------------------+
| id | name                  | promedio_producto |
+----+-----------------------+-------------------+
|  1 | Camiseta básica       |               4.5 |
|  3 | Zapatillas deportivas |               4.9 |
|  4 | Lámpara LED           |               4.2 |
|  6 | Juego de sábanas      |               4.7 |
|  8 | Blusa manga larga     |               4.1 |
| 10 | Cojín decorativo      |               4.8 |
+----+-----------------------+-------------------+
```

5.Como auditor, quiero conocer todas las empresas que no han recibido ninguna calificación.

```sql
SELECT id, name
FROM companies c
LEFT JOIN rates r ON c.id = r.company_id
WHERE r.company_id IS NULL

UNION

SELECT NULL AS id, 'Todas las empresas tienen calificación' AS name
WHERE NOT EXISTS (
  SELECT 1
  FROM companies c
  LEFT JOIN rates r ON c.id = r.company_id
  WHERE r.company_id IS NULL
);
```

```
+------+----------------------------------------+
| id   | name                                   |
+------+----------------------------------------+
| NULL | Todas las empresas tienen calificación |
+------+----------------------------------------+
```

6. Como operador, deseo obtener los productos que han sido añadidos como favoritos por más de 10 clientes distintos.

```sql
SELECT
    p.id,
    p.name,
    COUNT(DISTINCT f.customer_id) AS total_clientes
FROM
    products p
JOIN
    details_favorites df ON p.id = df.product_id
JOIN
    favorites f ON df.favorite_id = f.id
    GROUP BY
    p.id, p.name
HAVING
    COUNT(DISTINCT f.customer_id) > 10;
```

7.Como gerente regional, quiero obtener todas las empresas activas por ciudad y categoría.

```sql
SELECT 
    c.city_id AS ciudad,
    cat.description AS categoria,
    COUNT(*) AS total_empresas
FROM 
    companies c
JOIN 
    categories cat ON c.category_id = cat.id
GROUP BY 
    c.city_id, cat.description
ORDER BY 
    c.city_id, cat.description;
```

```sql
+--------+-------------+----------------+
| ciudad | categoria   | total_empresas |
+--------+-------------+----------------+
| 001    | Alimentos   |              2 |
| 001    | comida      |              3 |
| 001    | Electronica |              3 |
| 001    | Ropa        |              2 |
+--------+-------------+----------------+
```

8.Como especialista en marketing, deseo obtener los 10 productos más calificados en cada ciudad.

```sql
WITH ranked_products AS (
    SELECT 
        c.city_id,
        qp.product_id,
        p.name AS product_name,
        AVG(qp.rating) AS avg_rating,
        DENSE_RANK() OVER (
            PARTITION BY c.city_id 
            ORDER BY AVG(qp.rating) DESC
        ) AS ranking
    FROM 
        quality_products qp
    JOIN 
        companies c ON qp.company_id = c.id
    JOIN 
        products p ON qp.product_id = p.id
    GROUP BY 
        c.city_id, qp.product_id
)
SELECT 
    city_id,
    product_id,
    product_name,
    ROUND(avg_rating, 2) AS avg_rating,
    ranking
FROM 
    ranked_products
WHERE 
    ranking <= 10
ORDER BY 
    city_id, ranking;
```

```sql
+---------+------------+-----------------------+------------+---------+
| city_id | product_id | product_name          | avg_rating | ranking |
+---------+------------+-----------------------+------------+---------+
| 001     |          3 | Zapatillas deportivas |        4.9 |       1 |
| 001     |         10 | Cojín decorativo      |        4.8 |       2 |
| 001     |          6 | Juego de sábanas      |        4.7 |       3 |
| 001     |          1 | Camiseta básica       |        4.5 |       4 |
| 001     |          4 | Lámpara LED           |        4.2 |       5 |
| 001     |          8 | Blusa manga larga     |        4.1 |       6 |
| 001     |          2 | Jeans Slim Fit        |        3.8 |       7 |
| 001     |          7 | Toalla de baño        |        3.5 |       8 |
| 001     |          9 | Pantaloneta deportiva |          3 |       9 |
| 001     |          5 | Audífonos alámbricos  |        2.9 |      10 |
+---------+------------+-----------------------+------------+---------+
```

9.Como técnico, quiero identificar productos sin unidad de medida asignada.

```sql
SELECT 
    p.id,
    p.name,
    p.detail,
    p.category_id,
    'Este producto no tiene unidad de medida asignada.' AS mensaje
FROM 
    products p
LEFT JOIN 
    companyproducts cp ON p.id = cp.product_id
WHERE 
    cp.unitmeasure_id IS NULL

UNION ALL

SELECT 
    NULL AS id,
    NULL AS name,
    NULL AS detail,
    NULL AS category_id,
    'Todos los productos tienen unidad de medida asignada.' AS mensaje
FROM 
    dual
WHERE 
    NOT EXISTS (
        SELECT 1 
        FROM companyproducts 
        WHERE unitmeasure_id IS NULL
    );
```

```sql
+------+------+--------+-------------+-------------------------------------------------------+
| id   | name | detail | category_id | mensaje                                               |
+------+------+--------+-------------+-------------------------------------------------------+
| NULL | NULL | NULL   |        NULL | Todos los productos tienen unidad de medida asignada. |
+------+------+--------+-------------+-------------------------------------------------------+
```

10. Como gestor de beneficios, deseo ver los planes de membresía sin beneficios registrados.

```sql
SELECT 
    m.id,
    m.name,
    m.description
FROM 
    memberships m
LEFT JOIN 
    membershipbenefits mb ON m.id = mb.membership_id
WHERE 
    mb.membership_id IS NULL;
```

```sql
+----+-------------+--------------------------------------------------------------------------------+
| id | name        | description                                                                    |
+----+-------------+--------------------------------------------------------------------------------+
|  4 | VIP         | Atención personalizada, ofertas especiales y prioridad máxima.                 |
|  5 | Corporativa | Diseñada para empresas con múltiples usuarios y facturación personalizada.     |
|  6 | Estudiante  | Acceso especial con descuento para estudiantes verificados.                    |
|  7 | Trial       | Acceso gratuito por tiempo limitado para probar el servicio.                   |
|  8 | Anual       | Membresía con beneficios extendidos durante 12 meses.                          |
|  9 | Mensual     | Membresía de pago mensual con posibilidad de cancelación en cualquier momento. |
| 10 | Invitado    | Acceso temporal y restringido sin necesidad de registro completo.              |
+----+-------------+--------------------------------------------------------------------------------+
```

11.Como supervisor, quiero obtener los productos de una categoría específica con su promedio de calificación.

```sql
SELECT
    p.id,
    p.name,
    p.category_id,
    AVG(qp.rating) AS promedio_calif
FROM
    products p
JOIN
    quality_products qp ON p.id = qp.product_id
WHERE
    p.category_id = 1 
GROUP BY
    p.id,
    p.name,
    p.category_id
ORDER BY
    promedio_calif DESC;
```

```sql
+----+-----------------------+-------------+----------------+
| id | name                  | category_id | promedio_calif |
+----+-----------------------+-------------+----------------+
|  3 | Zapatillas deportivas |           1 |            4.9 |
|  1 | Camiseta básica       |           1 |            4.5 |
|  8 | Blusa manga larga     |           1 |            4.1 |
|  2 | Jeans Slim Fit        |           1 |            3.8 |
|  9 | Pantaloneta deportiva |           1 |              3 |
+----+-----------------------+-------------+----------------+
```

12.Como asesor, deseo obtener los clientes que han comprado productos de más de una empresa.

```sql
SELECT 
    c.id AS customer_id,
    c.name AS customer_name,
    COUNT(DISTINCT f.company_id) AS empresas_compradas
FROM
    customers c
JOIN
    favorites f ON c.id = f.customer_id
GROUP BY
    c.id,
    c.name
HAVING
    COUNT(DISTINCT f.company_id) > 1;
```

```sql
+-------------+--------------------+--------------------+
| customer_id | customer_name      | empresas_compradas |
+-------------+--------------------+--------------------+
|           5 | Luis Rodríguez     |                  2 |
|           6 | Laura Mejía        |                  2 |
|          12 | Paula Vargas       |                  2 |
|          13 | Ricardo Sánchez    |                  2 |
|          14 | Valentina Restrepo |                  2 |
|          15 | Diego Castro       |                  2 |
+-------------+--------------------+--------------------+
```

13. Como director, quiero identificar las ciudades con más clientes activos.

```sql
SELECT
    city_id,
    COUNT(*) AS total_clientes
FROM
    customers
GROUP BY
    city_id
ORDER BY
    total_clientes DESC;
```

```sql
+---------+----------------+
| city_id | total_clientes |
+---------+----------------+
| 001     |             16 |
| 290     |              5 |
| 873     |              5 |
+---------+----------------+
```

14. Como analista de calidad, deseo obtener el ranking de productos por empresa basado en la media de `quality_products`.

```sql
SELECT
    qp.company_id,
    c.name AS company_name,
    qp.product_id,
    p.name AS product_name,
    AVG(qp.rating) AS average_rating,
    RANK() OVER (PARTITION BY qp.company_id ORDER BY AVG(qp.rating) DESC) AS product_rank
FROM
    quality_products qp
JOIN
    products p ON qp.product_id = p.id
JOIN
    companies c ON qp.company_id = c.id
GROUP BY
    qp.company_id,
    c.name,
    qp.product_id,
    p.name
ORDER BY
    qp.company_id,
    product_rank;
```

```sql
+-------------+------------------------------+------------+-----------------------+----------------+--------------+
| company_id  | company_name                 | product_id | product_name          | average_rating | product_rank |
+-------------+------------------------------+------------+-----------------------+----------------+--------------+
| 900111001-1 | Bocados del Valle S.A.S      |          1 | Camiseta básica       |            4.5 |            1 |
| 900111002-2 | TecnoGlobal Ltda             |          2 | Jeans Slim Fit        |            3.8 |            1 |
| 900111003-3 | Alimentos El Sol S.A.        |          3 | Zapatillas deportivas |            4.9 |            1 |
| 900111004-4 | Moda Urbana S.A.S            |          4 | Lámpara LED           |            4.2 |            1 |
| 900111005-5 | Comidas Express S.A.S        |          5 | Audífonos alámbricos  |            2.9 |            1 |
| 900111006-6 | ElectroMundo S.A.            |          6 | Juego de sábanas      |            4.7 |            1 |
| 900111007-7 | Sabor Llanero Ltda           |          7 | Toalla de baño        |            3.5 |            1 |
| 900111008-8 | Vanguardia Electrónica S.A.S |          8 | Blusa manga larga     |            4.1 |            1 |
| 900111009-9 | Ropa Total S.A.S             |          9 | Pantaloneta deportiva |              3 |            1 |
| 900111010-0 | Granos del Norte Ltda        |         10 | Cojín decorativo      |            4.8 |            1 |
+-------------+------------------------------+------------+-----------------------+----------------+--------------+
```

15. Como administrador, quiero listar empresas que ofrecen más de cinco productos distintos.

    ```sql
    SELECT
        c.id AS company_id,
        c.name AS company_name,
        COUNT(DISTINCT cp.product_id) AS total_products
    FROM
        companies c
    JOIN
        companyproducts cp ON c.id = cp.company_id
    GROUP BY
        c.id,
        c.name
    HAVING
        COUNT(DISTINCT cp.product_id) > 5
    ORDER BY
        total_products DESC;
    
    ```

    ```sql
    +-------------+-------------------------+----------------+
    | company_id  | company_name            | total_products |
    +-------------+-------------------------+----------------+
    | 900111001-1 | Bocados del Valle S.A.S |              6 |
    | 900111002-2 | TecnoGlobal Ltda        |              6 |
    +-------------+-------------------------+----------------+
    ```

    16. Como cliente, deseo visualizar los productos favoritos que aún no han sido calificados.

```sql
SELECT DISTINCT
    p.id AS product_id,
    p.name AS product_name,
    NULL AS mensaje
FROM
    favorites f
JOIN
    details_favorites df ON f.id = df.favorite_id
JOIN
    products p ON df.product_id = p.id
LEFT JOIN
    quality_products qp ON p.id = qp.product_id
WHERE
    qp.product_id IS NULL

UNION ALL

SELECT
    NULL AS product_id,
    NULL AS product_name,
    'Todos los productos tienen calificación' AS mensaje
FROM
    dual
WHERE NOT EXISTS (
    SELECT 1
    FROM
        favorites f2
    JOIN
        details_favorites df2 ON f2.id = df2.favorite_id
    JOIN
        products p2 ON df2.product_id = p2.id
    LEFT JOIN
        quality_products qp2 ON p2.id = qp2.product_id
    WHERE
        qp2.product_id IS NULL
);
```

```sql
+------------+--------------+-----------------------------------------+
| product_id | product_name | mensaje                                 |
+------------+--------------+-----------------------------------------+
|       NULL | NULL         | Todos los productos tienen calificación |
+------------+--------------+-----------------------------------------+
```

17.Como desarrollador, deseo consultar los beneficios asignados a cada audiencia junto con su descripción.

```sql
SELECT
    a.description AS audience_description,
    ab.benefit_id,
    b.description AS benefit_description
FROM
    audiencebenefits ab
JOIN
    audiences a ON ab.audience_id = a.id
JOIN
    benefits b ON ab.benefit_id = b.id;
```

```sql
+----------------------+------------+------------------------+
| audience_description | benefit_id | benefit_description    |
+----------------------+------------+------------------------+
| Clientes             |          1 | Envío Gratis           |
| Empleados            |          1 | Envío Gratis           |
| Clientes             |          2 | Descuentos Exclusivos  |
| Proveedores          |          2 | Descuentos Exclusivos  |
| Clientes             |          3 | Atención Prioritaria   |
| Empleados            |          3 | Atención Prioritaria   |
| Proveedores          |          4 | Contenido Premium      |
| Proveedores          |          5 | Ofertas Personalizadas |
| Empleados            |          5 | Ofertas Personalizadas |
+----------------------+------------+------------------------+
```

18. Como operador logístico, quiero saber en qué ciudades hay empresas sin productos asociados.

```sql
SELECT
    ci.name AS city_name
FROM
    citiesormunicipalities ci
JOIN
    companies c ON c.city_id = ci.code
LEFT JOIN
    companyproducts cp ON cp.company_id = c.id
WHERE
    cp.product_id IS NULL
GROUP BY
    ci.name

UNION ALL

SELECT
    'Todas las empresas tienen productos asociados' AS city_name
WHERE NOT EXISTS (
    SELECT 1
    FROM
        companies c2
    LEFT JOIN
        companyproducts cp2 ON cp2.company_id = c2.id
    WHERE
        cp2.product_id IS NULL
)
LIMIT 1;
```

```sql
+-----------------------------------------------+
| city_name                                     |
+-----------------------------------------------+
| Todas las empresas tienen productos asociados |
+-----------------------------------------------+
```

19. Como técnico, deseo obtener todas las empresas con productos duplicados por nombre

    ```sql
    SELECT
        c.id AS company_id,
        c.name AS company_name,
        p.name AS duplicated_product_name,
        COUNT(*) AS duplicate_count
    FROM
        companyproducts cp
    JOIN
        companies c ON cp.company_id = c.id
    JOIN
        products p ON cp.product_id = p.id
    GROUP BY
        c.id,
        p.name
    HAVING
        COUNT(*) > 1
    
    UNION ALL
    
    SELECT
        NULL AS company_id,
        NULL AS company_name,
        'No hay productos duplicados por empresa.' AS duplicated_product_name,
        NULL AS duplicate_count
    WHERE NOT EXISTS (
        SELECT 1
        FROM companyproducts cp2
        JOIN companies c2 ON cp2.company_id = c2.id
        JOIN products p2 ON cp2.product_id = p2.id
        GROUP BY c2.id, p2.name
        HAVING COUNT(*) > 1
    );
    ```

    

```sql
+------------+--------------+------------------------------------------+-----------------+
| company_id | company_name | duplicated_product_name                  | duplicate_count |
+------------+--------------+------------------------------------------+-----------------+
| NULL       | NULL         | No hay productos duplicados por empresa. |            NULL |
+------------+--------------+------------------------------------------+-----------------+
```

20.Como analista, quiero una vista resumen de clientes, productos favoritos y promedio de calificación recibido

```sql
CREATE OR REPLACE VIEW resumen_clientes_favoritos AS
SELECT
    c.id AS client_id,
    c.name AS client_name,
    COUNT(DISTINCT df.product_id) AS total_favoritos,
    ROUND(AVG(qp.rating), 2) AS promedio_calificacion
FROM
    customers c
LEFT JOIN
    favorites f ON f.customer_id = c.id
LEFT JOIN
    details_favorites df ON df.favorite_id = f.id
LEFT JOIN
    quality_products qp ON qp.product_id = df.product_id
GROUP BY
    c.id,
    c.name;
```

```sql
SELECT * FROM resumen_clientes_favoritos;
+-----------+--------------------+-----------------+-----------------------+
| client_id | client_name        | total_favoritos | promedio_calificacion |
+-----------+--------------------+-----------------+-----------------------+
|         1 | Juan Pérez         |               1 |                   4.5 |
|         2 | María Gómez        |               1 |                   3.8 |
|         3 | Carlos Ruiz        |               1 |                   4.9 |
|         4 | Ana Torres         |               1 |                   4.2 |
|         5 | Luis Rodríguez     |               2 |                   3.7 |
|         6 | Laura Mejía        |               2 |                   4.6 |
|         7 | Felipe Rojas       |               1 |                   3.5 |
|         8 | Andrea Castaño     |               1 |                   4.1 |
|         9 | Manuel Rivera      |               1 |                     3 |
|        10 | Camila Ortiz       |               1 |                   4.8 |
|        11 | Sebastián Moreno   |               1 |                   4.5 |
|        12 | Paula Vargas       |               2 |                  4.15 |
|        13 | Ricardo Sánchez    |               2 |                   4.7 |
|        14 | Valentina Restrepo |               2 |                  4.35 |
|        15 | Diego Castro       |               2 |                   3.7 |
|        16 | Cliente 16         |               0 |                  NULL |
|        17 | Cliente 17         |               0 |                  NULL |
|        18 | Cliente 18         |               0 |                  NULL |
|        19 | Cliente 19         |               0 |                  NULL |
|        20 | Cliente 20         |               0 |                  NULL |
|        21 | Cliente 21         |               0 |                  NULL |
|        22 | Cliente 22         |               0 |                  NULL |
|        23 | Cliente 23         |               0 |                  NULL |
|        24 | Cliente 24         |               0 |                  NULL |
|        25 | Cliente 25         |               0 |                  NULL |
|        26 | Cliente 26         |               0 |                  NULL |
+-----------+--------------------+-----------------+-----------------------+
```



## **2. Subconsultas**

1.Como gerente, quiero ver los productos cuyo precio esté por encima del promedio de su categoría.

```sql
SELECT
    p.id AS product_id,
    p.name AS product_name,
    p.category_id,
    p.price,
    cat.description AS category_name,
    cat_avg.avg_price AS average_category_price
FROM
    products p
JOIN
    categories cat ON p.category_id = cat.id
JOIN (
    SELECT
        category_id,
        AVG(price) AS avg_price
    FROM
        products
    GROUP BY
        category_id
) AS cat_avg ON p.category_id = cat_avg.category_id
WHERE
    p.price > cat_avg.avg_price
ORDER BY
    p.category_id, p.price DESC;
```

```sql
+------------+-----------------------+-------------+--------+---------------+------------------------+
| product_id | product_name          | category_id | price  | category_name | average_category_price |
+------------+-----------------------+-------------+--------+---------------+------------------------+
|          3 | Zapatillas deportivas |           1 | 120000 | Alimentos     |                  67200 |
|          2 | Jeans Slim Fit        |           1 |  89000 | Alimentos     |                  67200 |
|          6 | Juego de sábanas      |           4 |  68000 | comida        |                  40200 |
|          4 | Lámpara LED           |           4 |  45000 | comida        |                  40200 |
+------------+-----------------------+-------------+--------+---------------+------------------------+
```

2.Como administrador, deseo listar las empresas que tienen más productos que la media de empresas.

```sql
SELECT
    c.id AS company_id,
    c.name AS company_name,
    COUNT(cp.product_id) AS total_productos
FROM
    companies c
JOIN
    companyproducts cp ON cp.company_id = c.id
GROUP BY
    c.id, c.name
HAVING
    COUNT(cp.product_id) > (
        SELECT AVG(product_count) FROM (
            SELECT
                company_id,
                COUNT(product_id) AS product_count
            FROM
                companyproducts
            GROUP BY
                company_id
        ) AS promedio_empresas
    )
ORDER BY
    total_productos DESC;
```

```sql
+-------------+-------------------------+-----------------+
| company_id  | company_name            | total_productos |
+-------------+-------------------------+-----------------+
| 900111001-1 | Bocados del Valle S.A.S |               6 |
| 900111002-2 | TecnoGlobal Ltda        |               6 |
+-------------+-------------------------+-----------------+
```

3.Como cliente, quiero ver mis productos favoritos que han sido calificados por otros clientes.

```sql
SELECT DISTINCT
    p.id AS product_id,
    p.name AS product_name,
    qp.rating AS calificacion,
    qp.customer_id AS calificado_por
FROM
    favorites f
JOIN
    details_favorites df ON f.id = df.favorite_id
JOIN
    products p ON df.product_id = p.id
JOIN
    quality_products qp ON p.id = qp.product_id
WHERE
    f.customer_id = 6
    AND qp.customer_id <> f.customer_id
ORDER BY
    p.id, qp.rating DESC;
```

```sql
+------------+-----------------+--------------+----------------+
| product_id | product_name    | calificacion | calificado_por |
+------------+-----------------+--------------+----------------+
|          1 | Camiseta básica |          4.5 |              1 |
+------------+-----------------+--------------+----------------+
```

4.Como supervisor, deseo obtener los productos con el mayor número de veces añadidos como favoritos.

```sql
SELECT
    p.id AS product_id,
    p.name AS product_name,
    COUNT(df.id) AS veces_favorito
FROM
    products p
JOIN
    details_favorites df ON p.id = df.product_id
GROUP BY
    p.id, p.name
ORDER BY
    veces_favorito DESC;
```

```sql
+------------+-----------------------+----------------+
| product_id | product_name          | veces_favorito |
+------------+-----------------------+----------------+
|          1 | Camiseta básica       |              9 |
|          5 | Audífonos alámbricos  |              2 |
|          2 | Jeans Slim Fit        |              2 |
|          4 | Lámpara LED           |              2 |
|          3 | Zapatillas deportivas |              2 |
|          8 | Blusa manga larga     |              1 |
|         10 | Cojín decorativo      |              1 |
|          6 | Juego de sábanas      |              1 |
|          9 | Pantaloneta deportiva |              1 |
|          7 | Toalla de baño        |              1 |
+------------+-----------------------+----------------+
```

5.Como técnico, quiero listar los clientes cuyo correo no aparece en la tabla `rates` ni en `quality_products`.

```sql
SELECT
    c.id AS customer_id,
    c.name AS customer_name,
    c.email
FROM
    customers c
WHERE
    c.id NOT IN (SELECT DISTINCT customer_id FROM rates)
    AND c.id NOT IN (SELECT DISTINCT customer_id FROM quality_products);
```

```sql
+-------------+--------------------+--------------------------------+
| customer_id | customer_name      | email                          |
+-------------+--------------------+--------------------------------+
|          11 | Sebastián Moreno   | sebastian.moreno@example.com   |
|          12 | Paula Vargas       | paula.vargas@example.com       |
|          13 | Ricardo Sánchez    | ricardo.sanchez@example.com    |
|          14 | Valentina Restrepo | valentina.restrepo@example.com |
|          15 | Diego Castro       | diego.castro@example.com       |
|          16 | Cliente 16         | cliente16@example.com          |
|          17 | Cliente 17         | cliente17@example.com          |
|          18 | Cliente 18         | cliente18@example.com          |
|          19 | Cliente 19         | cliente19@example.com          |
|          20 | Cliente 20         | cliente20@example.com          |
|          21 | Cliente 21         | cliente21@example.com          |
|          22 | Cliente 22         | cliente22@example.com          |
|          23 | Cliente 23         | cliente23@example.com          |
|          24 | Cliente 24         | cliente24@example.com          |
|          25 | Cliente 25         | cliente25@example.com          |
|          26 | Cliente 26         | cliente26@example.com          |
+-------------+--------------------+--------------------------------+
```

6.Como gestor de calidad, quiero obtener los productos con una calificación inferior al mínimo de su categoría.

```sql
-- Productos con calificación inferior al mínimo de su categoría
WITH promedio_categoria AS (
    SELECT 
        p.category_id,
        MIN(AVG(qp.rating)) OVER (PARTITION BY p.category_id) AS min_categoria
    FROM quality_products qp
    JOIN products p ON qp.product_id = p.id
    GROUP BY p.category_id, qp.product_id
),
promedio_producto AS (
    SELECT 
        qp.product_id,
        p.name,
        p.category_id,
        AVG(qp.rating) AS promedio_producto
    FROM quality_products qp
    JOIN products p ON qp.product_id = p.id
    GROUP BY qp.product_id, p.name, p.category_id
)
SELECT 
    pp.product_id,
    pp.name AS nombre_producto,
    pp.category_id,
    pp.promedio_producto,
    pc.min_categoria
FROM promedio_producto pp
JOIN promedio_categoria pc ON pp.category_id = pc.category_id
WHERE pp.promedio_producto < pc.min_categoria;
```

7.Como desarrollador, deseo listar las ciudades que no tienen clientes registrados.

```sql
SELECT DISTINCT c.city_id
FROM companies c
WHERE c.city_id NOT IN (
    SELECT DISTINCT city_id FROM customers
);
```

8.Como administrador, quiero ver los productos que no han sido evaluados en ninguna encuesta.

```sql
SELECT p.id, p.name, p.detail, p.price
FROM products p
LEFT JOIN quality_products qp ON p.id = qp.product_id
WHERE qp.product_id IS NULL;
```

9.Como administrador, quiero ver los productos que no han sido evaluados en ninguna encuesta.

```sql
SELECT b.id, b.description, b.detail
FROM benefits b
LEFT JOIN audiencebenefits ab ON b.id = ab.benefit_id
WHERE ab.audience_id IS NULL;
```

10.Como cliente, deseo obtener mis productos favoritos que no están disponibles actualmente en ninguna empresa.

```sql
SELECT DISTINCT p.id, p.name, p.detail
FROM favorites f
JOIN details_favorites df ON f.id = df.favorite_id
JOIN products p ON p.id = df.product_id
WHERE f.customer_id = 4
  AND p.id NOT IN (
      SELECT DISTINCT product_id FROM companyproducts
  );
```

11.Como director, deseo consultar los productos vendidos en empresas cuya ciudad tenga menos de tres empresas registradas.

```sql
SELECT DISTINCT cp.product_id, p.name AS product_name, c.city_id
FROM companyproducts cp
JOIN companies c ON cp.company_id = c.id
JOIN products p ON cp.product_id = p.id
WHERE c.city_id IN (
    SELECT city_id
    FROM companies
    GROUP BY city_id
    HAVING COUNT(*) < 11
);
```

12.Como analista, quiero ver los productos con calidad superior al promedio de todos los productos.

```sql
SELECT p.id AS product_id, p.name AS product_name, AVG(qp.rating) AS average_quality
FROM products p
JOIN quality_products qp ON p.id = qp.product_id
GROUP BY p.id, p.name
HAVING AVG(qp.rating) > (
    SELECT AVG(rating) FROM quality_products
);
```

13.Como gestor, quiero ver empresas que sólo venden productos de una única categoría.

```sql
SELECT c.id, c.name
FROM companies c
JOIN companyproducts cp ON c.id = cp.company_id
JOIN products p ON cp.product_id = p.id
GROUP BY c.id, c.name
HAVING COUNT(DISTINCT p.category_id) = 1;
```

14.Como gerente comercial, quiero consultar los productos con el mayor precio entre todas las empresas.

```sql
SELECT p.id, p.name, MAX(cp.price) AS max_price
FROM products p
JOIN companyproducts cp ON p.id = cp.product_id
GROUP BY p.id, p.name
ORDER BY max_price DESC
LIMIT 10;
```

15.Como cliente, quiero saber si algún producto de mis favoritos ha sido calificado por otro cliente con más de 4 estrellas.

```sql
SELECT DISTINCT p.id AS product_id, p.name AS product_name, r.customer_id AS reviewer_id, r.rating
FROM favorites f
JOIN details_favorites df ON f.id = df.favorite_id
JOIN products p ON df.product_id = p.id
JOIN rates r ON r.customer_id <> f.customer_id
    AND r.company_id IN (
        SELECT company_id 
        FROM companyproducts cp2 
        WHERE cp2.product_id = p.id
    )
WHERE f.customer_id = 15
  AND r.rating > 4;
```

16.Como operador, quiero saber qué productos no tienen imagen asignada pero sí han sido calificados.

```sql
SELECT DISTINCT p.id, p.name
FROM products p
JOIN companyproducts cp ON p.id = cp.product_id
JOIN rates r ON cp.company_id = r.company_id
WHERE p.image IS NULL
  AND r.rating IS NOT NULL;
```

17.Como auditor, quiero ver los planes de membresía sin periodo vigente.

```sql
SELECT m.id, m.name
FROM memberships m
LEFT JOIN membershipperiods mp ON m.id = mp.membership_id
WHERE mp.period_id IS NULL;
```

18.Como especialista, quiero identificar los beneficios compartidos por más de una audiencia.

```sql
SELECT 
  ab.benefit_id,
  b.description,
  COUNT(ab.audience_id) AS audience_count
FROM audiencebenefits ab
JOIN benefits b ON ab.benefit_id = b.id
GROUP BY ab.benefit_id, b.description
HAVING COUNT(ab.audience_id) > 1;
```

19.Como técnico, quiero encontrar empresas cuyos productos no tengan unidad de medida definida.

```sql
SELECT DISTINCT
  c.id AS company_id,
  c.name AS company_name
FROM companies c
JOIN companyproducts cp ON c.id = cp.company_id
WHERE cp.unitmeasure_id IS NULL;
```

20.Como gestor de campañas, deseo obtener los clientes con membresía activa y sin productos favoritos.

```sql
SELECT c.id, c.name
FROM customers c
LEFT JOIN favorites f ON f.customer_id = c.id
LEFT JOIN details_favorites df ON df.favorite_id = f.id
WHERE df.product_id IS NULL;
```

##  **3. Funciones Agregadas**



### **1. Obtener el promedio de calificación por producto**

```sql
SELECT 
  p.id AS product_id,
  p.name AS product_name,
  ROUND(AVG(qp.rating), 2) AS average_rating
FROM products p
JOIN quality_products qp ON p.id = qp.product_id
GROUP BY p.id, p.name
ORDER BY average_rating DESC;
```

------

### **2. Contar cuántos productos ha calificado cada cliente**

```sql
SELECT 
  c.id AS customer_id,
  c.name AS customer_name,
  COUNT(DISTINCT qp.product_id) AS total_products_rated
FROM customers c
JOIN quality_products qp ON c.id = qp.customer_id
GROUP BY c.id, c.name
ORDER BY total_products_rated DESC;
```



------

### **3. Sumar el total de beneficios asignados por audiencia**

> ```sql
> SELECT 
>   a.id AS audience_id,
>   a.description AS audience_description,
>   COUNT(ab.benefit_id) AS total_benefits
> FROM audiences a
> LEFT JOIN audiencebenefits ab ON a.id = ab.audience_id
> GROUP BY a.id, a.description
> ORDER BY total_benefits DESC;
> ```

------

### **4. Calcular la media de productos por empresa**

```sql
SELECT 
  AVG(product_count) AS average_products_per_company
FROM (
  SELECT 
    company_id, 
    COUNT(*) AS product_count
  FROM companyproducts
  GROUP BY company_id
) AS company_product_counts;
```



------

### **5. Contar el total de empresas por ciudad**

```sql
SELECT 
  city_id, 
  COUNT(*) AS total_empresas
FROM companies
GROUP BY city_id;
```

------

### **6. Calcular el promedio de precios por unidad de medida**

```sql
SELECT 
  u.description AS unidad_medida,
  AVG(cp.price) AS promedio_precio
FROM companyproducts cp
JOIN unitofmeasure u ON cp.unitmeasure_id = u.id
GROUP BY cp.unitmeasure_id;
```

------

### **7. Contar cuántos clientes hay por ciudad**

```sql
SELECT 
  c.name AS ciudad,
  COUNT(*) AS total_clientes
FROM customers cu
JOIN citiesormunicipalities c ON cu.city_id = c.code
GROUP BY cu.city_id;
```

------

### **8. Calcular planes de membresía por periodo**

```sql
SELECT 
  p.name AS nombre_periodo,
  COUNT(*) AS total_planes
FROM membershipperiods mp
JOIN periods p ON mp.period_id = p.id
GROUP BY mp.period_id;
```

------

### **9. Ver el promedio de calificaciones dadas por un cliente a sus favoritos**

```sql
SELECT 
  f.customer_id,
  AVG(qp.rating) AS promedio_calificaciones
FROM favorites f
JOIN details_favorites df ON f.id = df.favorite_id
JOIN quality_products qp 
  ON df.product_id = qp.product_id AND f.customer_id = qp.customer_id
GROUP BY f.customer_id;
```

------

### **10. Consultar la fecha más reciente en que se calificó un producto**

```sql
SELECT 
  product_id,
  MAX(daterating) AS fecha_ultima_calificacion
FROM quality_products
GROUP BY product_id;
```

------

### **11. Obtener la desviación estándar de precios por categoría**

```sql
SELECT 
  p.category_id,
  STDDEV(cp.price) AS desviacion_estandar_precio
FROM companyproducts cp
JOIN products p ON cp.product_id = p.id
GROUP BY p.category_id;
```

------

### **12. Contar cuántas veces un producto fue favorito**

```sql
SELECT 
  product_id, 
  COUNT(*) AS veces_favorito
FROM details_favorites
GROUP BY product_id
ORDER BY veces_favorito DESC;
```

------

### **13. Calcular el porcentaje de productos evaluados**

```sql
SELECT 
  ROUND(
    (COUNT(DISTINCT qp.product_id) / (SELECT COUNT(*) FROM products)) * 100,
    2
  ) AS porcentaje_productos_evaluados
FROM quality_products qp;
```

------

### **14. Ver el promedio de rating por encuesta**

```sql
SELECT 
  poll_id,
  ROUND(AVG(rating), 2) AS promedio_rating
FROM rates
GROUP BY poll_id;
```

------

### **15. Calcular el promedio y total de beneficios por plan**

```sql
SELECT
  mb.membership_id,
  m.name AS membership_name,
  COUNT(*) AS total_beneficios
FROM membershipbenefits mb
JOIN memberships m ON mb.membership_id = m.id
GROUP BY mb.membership_id, m.name;
```

------

### **16. Obtener media y varianza de precios por empresa**

```sql
SELECT
  company_id,
  AVG(price) AS promedio_precio,
  VAR_POP(price) AS varianza_precio
FROM companyproducts
GROUP BY company_id;
```

------

### **17. Ver total de productos disponibles en la ciudad del cliente**

```sql
SELECT COUNT(DISTINCT cp.product_id) AS total_productos_disponibles
FROM customers c
JOIN companies co ON co.city_id = c.city_id
JOIN companyproducts cp ON cp.company_id = co.id
WHERE c.id = 10;
```

------

### **18. Contar productos únicos por tipo de empresa**

```sql
SELECT
  c.type_id AS tipo_empresa,
  COUNT(DISTINCT cp.product_id) AS productos_unicos
FROM companies c
JOIN companyproducts cp ON c.id = cp.company_id
GROUP BY c.type_id;
```

------

### **19. Ver total de clientes sin correo electrónico registrado**

```sql
SELECT COUNT(*) AS total_clientes_sin_email
FROM customers
WHERE email IS NULL OR email = '';
```

------

### **20. Empresa con más productos calificados**

```sql
SELECT c.id AS company_id, c.name AS company_name, COUNT(DISTINCT cp.product_id) AS productos_calificados
FROM companies c
JOIN companyproducts cp ON c.id = cp.company_id
JOIN quality_products qp ON cp.product_id = qp.product_id AND cp.company_id = qp.company_id
GROUP BY c.id, c.name
ORDER BY productos_calificados DESC
LIMIT 1;
```

