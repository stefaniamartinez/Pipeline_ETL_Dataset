M6: Pipeline ETL Completo: de datos crudos a modelo confiable en Power BI

Siguiendo el lineamiento de que eliminar nulos sin justificación no es correcto, se evaluó cada campo antes de decidir: en clientes, los nulos están en ciudad/email, campos no críticos, por lo que se reemplazaron por 'Sin datos'. En productos, el nulo en categoria es descriptivo, por lo que también se reemplazó por 'Sin categoría'; en cambio, el nulo en precio es crítico para calcular ingreso — ahí no se reemplazó por un valor inventado (para no falsear el cálculo) ni se eliminó la fila (para no perder el producto ni sus ventas asociadas): se dejó null y se excluye puntualmente al calcular el ingreso total.

Se dejó comentarios en el editor avanzado: 
1.-
    // Convierte cada columna al tipo de dato correcto para poder operar con ellas
    #"Tipo cambiado" = Table.TransformColumnTypes(#"Encabezados promovidos",{
        {"cantidad", Int64.Type},          // cantidad de unidades vendidas -> número entero
        {"precio_unitario", Currency.Type},// precio por unidad -> tipo moneda
        {"descuento", Percentage.Type},    // descuento aplicado -> tipo porcentaje
        {"total_venta", Currency.Type}     // total de la venta -> tipo moneda
    }),

2.-
    // Combina (join) la tabla de ventas con la tabla de dimensión de productos (Dim_Productos)
    // usando id_producto como clave, tipo LeftOuter (conserva todas las ventas aunque no matcheen)
    #"Consultas combinadas" = Table.NestedJoin(#"Tipo cambiado1", {"id_producto"}, Dim_Productos, {"id_producto"}, "Dim_Productos", JoinKind.LeftOuter),
    #"Se expandió Dim_Productos" = Table.ExpandTableColumn(#"Consultas combinadas", "Dim_Productos", {"nombre_producto", "categoria"}, {"Dim_Productos.nombre_producto", "Dim_Productos.categoria"})
