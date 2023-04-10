==================
EJERCICIOS CLASE 1
==================
1)Mostrar todos los productos dentro de la categoria electro junto con todos los detalles.

select * from stg.product_master
where categoria = 'Electro'

2)Cuales son los producto producidos en China?

select * from stg.product_master
where origen = 'China'

3)Mostrar todos los productos de Electro ordenados por nombre.

select * from stg.product_master
where categoria = 'Electro'
order by nombre asc

4)Cuales son las TV que se encuentran activas para la venta?

select * from stg.product_master
where subcategoria = 'TV' and is_active=true

5)Mostrar todas las tiendas de Argentina ordenadas por fecha de apertura de las mas antigua a la mas nueva.

select * from stg.store_master
where pais = 'Argentina'
order by fecha_apertura asc

6)Cuales fueron las ultimas 5 ordenes de ventas?

select * from stg.order_line_sale
order by fecha desc
limit 5

7)Mostrar los primeros 10 registros de el conteo de trafico por Super store ordenados por fecha.

select * from stg.super_store_count
order by fecha asc
limit 10

8)Cuales son los producto de electro que no son Soporte de TV ni control remoto.

select * from stg.product_master
where categoria = 'Electro' and subsubcategoria != 'Soporte' and subsubcategoria != 'Control remoto'

9)Mostrar todas las lineas de venta donde el monto sea mayor a $100.000 solo para transacciones en pesos.

select * from stg.order_line_sale
where venta > 100000 and moneda = 'ARS'

10)Mostrar todas las lineas de ventas de Octubre 2022.

select * from stg.order_line_sale
where fecha >=  '2022-10-01' and fecha <= '2022-10-31'

11)Mostrar todos los productos que tengan EAN.

select * from stg.product_master
where ean notnull

12)Mostrar todas las lineas de venta que que hayan sido vendidas entre 1 de Octubre de 2022 y 10 de Noviembre de 2022.

select * from stg.order_line_sale
where fecha >= '2022-10-01' and fecha <= '2022-11-10'


==================
EJERCICIOS CLASE 2
==================
1)Cuales son los paises donde la empresa tiene tiendas?

select distinct pais
from stg.store_master

2)Cuantos productos por subcategoria tiene disponible para la venta?

select 
	subcategoria, 
	count (codigo_producto) as cantidad_productos
from stg.product_master
group by subcategoria

3)Cuales son las ordenes de venta de Argentina de mayor a $100.000?

select  *
from stg.order_line_sale
where moneda = 'ARS' and venta > 100000

4)Obtener los decuentos otorgados durante Noviembre de 2022 en cada una de las monedas?

select 
	moneda,
	coalesce (sum(descuento),0)
from stg.order_line_sale
where fecha between '2022-11-01' and '2022-11-30'
group by moneda 

5)Obtener los impuestos pagados en Europa durante el 2022.

select sum(impuestos) as Total_Impuestos
from stg.order_line_sale
where moneda = 'EUR' and fecha between '2022-01-01' and '2022-12-31'

6)En cuantas ordenes se utilizaron creditos?

select count(distinct orden)
from stg.order_line_sale
where creditos is not null

7)Cual es el % de descuentos otorgados (sobre las ventas) por tienda?

select 
	tienda, 
	round(sum(descuento)* 100 /sum(venta),2) as Porcentaje_Descuentos_Otorgados
from stg.order_line_sale
group by tienda

8)Cual es el inventario promedio por dia que tiene cada tienda?

select 
	tienda,
	fecha,
	(sum(inicial)+sum(final))/2 as Media
from stg.inventory
group by tienda, fecha
order by tienda, fecha

9)Obtener las ventas netas y el porcentaje de descuento otorgado por producto en Argentina.

select 
	producto, 
	sum(venta) as venta, 
	round(coalesce (sum(descuento)*100/sum(venta),0),2) as Descuento
from stg.order_line_sale
where moneda = 'ARS'
group by producto
order by Descuento

10)Las tablas "market_count" y "super_store_count" representan dos sistemas distintos que usa la empresa para contar la cantidad de gente que 
ingresa a tienda, uno para las tiendas de Latinoamerica y otro para Europa. Obtener en una unica tabla, las entradas a tienda de ambos sistemas.

select 
	tienda, 
	cast(cast(fecha as text) as date), 
	conteo
from stg.market_count
UNION ALL
select 
	tienda,
	cast(fecha as date),
	conteo
from stg.super_store_count
order by tienda, fecha

11)Cuales son los productos disponibles para la venta (activos) de la marca Phillips?

select * from stg.product_master 
where nombre like '%PHILIPS%' and is_active = 'true'

12)Obtener el monto vendido por tienda y moneda y ordenarlo de mayor a menor por valor nominal.

select 
	tienda,
	moneda,
	sum(venta) as Monto
from stg.order_line_sale
group by tienda, moneda 
order by monto desc

13)Cual es el precio promedio de venta de cada producto en las distintas monedas? Recorda que los valores de venta, impuesto, descuentos y creditos 
es por el total de la linea.

select 
	producto, 
	moneda, 
	round(sum(venta)/sum(cantidad),2) as PromedioXunidad
from stg.order_line_sale
group by producto, moneda
order by producto, moneda

14)Cual es la tasa de impuestos que se pago por cada orden de venta?

select 
	orden, 
	round(sum(impuestos)*100/sum(venta),2) as tasa_impuestos
from stg.order_line_sale
group by orden


==================
EJERCICIOS CLASE 3
==================
1)Mostrar nombre y codigo de producto, categoria y color para todos los productos de la marca Philips y Samsung, mostrando la leyenda "Unknown" cuando 
no hay un color disponible

select 
	nombre,
	codigo_producto, 
	categoria,
	case 
		when color is null then 'Unknown' 
		else color end as color
from stg.product_master
where nombre like ('%PHILIPS%') or nombre like ('%Samsung%')

2)Calcular las ventas brutas y los impuestos pagados por pais y provincia en la moneda correspondiente.

select 
	pais, 
	provincia,
	case 
		when ols.moneda is null and pais = 'Argentina' then 'ARS' 
		when ols.moneda is null and pais = 'Uruguay' then 'URU' 
		when ols.moneda is null and pais = 'España' then 'EUR' 
		else moneda
	end as moneda,
	round(sum(cast(coalesce(venta,0) as numeric)),2) as ventas,
	round(sum(cast(coalesce(impuestos,0) as numeric)),2) as impuestos
from stg.store_master as sm
left join stg.order_line_sale as ols 
on ols.tienda = sm.codigo_tienda
group by pais, provincia, ols.moneda
order by pais, provincia

3)Calcular las ventas totales por subcategoria de producto para cada moneda ordenados por subcategoria y moneda.

select 
	pm.subcategoria, 
	moneda,
	sum(venta) as ventas
from stg.order_line_sale as ols
left join stg.product_master as pm 
on pm.codigo_producto = ols.producto
group by pm.subcategoria, moneda
order by pm.subcategoria, moneda

4)Calcular las unidades vendidas por subcategoria de producto y la concatenacion de pais, provincia; usar guion como separador y usarla para ordernar 
el resultado.

select 
	concat(sm.pais,'-',sm.provincia) as pais_provincia,
	pm.subcategoria, 
	sum(cantidad) as unidades
from stg.order_line_sale as ols
left join stg.product_master as pm 
	on pm.codigo_producto = ols.producto
left join stg.store_master as sm 
	on sm.codigo_tienda = ols.tienda
group by pm.subcategoria, sm.pais, sm.provincia
order by pais_provincia

5)Mostrar una vista donde sea vea el nombre de tienda y la cantidad de entradas de personas que hubo desde la fecha de apertura para el sistema "super_store".

select 
	sm.nombre as sucursal, 
	coalesce(sum(conteo),0) as cantidad_visitantes
from stg.store_master as sm
left join stg.super_store_count as ssc 
	on ssc.tienda = sm.codigo_tienda
group by sm.nombre

6)Cual es el nivel de inventario promedio en cada mes a nivel de codigo de producto y tienda; mostrar el resultado con el nombre de la tienda.

select 
	sm.nombre as tienda, 
	inv.sku as cod_producto,
	extract(month from inv.fecha) as mes,
	(sum(inicial)+sum(final))/2 as inventario_prom_mes
from stg.store_master sm
left join stg.inventory as inv 
on inv.tienda = sm.codigo_tienda
group by sm.nombre,inv.sku, sm.codigo_tienda, mes
order by sm.codigo_tienda, inv.sku

7)Calcular la cantidad de unidades vendidas por material. Para los productos que no tengan material usar 'Unknown', homogeneizar los textos si es necesario.

select 
	case 
		when lower(pm.material) is null then 'Unknown' 
		else lower(pm.material) 
	end as material,
	sum(cantidad) as cantidad
from stg.product_master as pm
left join stg.order_line_sale as ols 
on ols.producto = pm.codigo_producto
group by lower(pm.material)

8)Mostrar la tabla order_line_sales agregando una columna que represente el valor de venta bruta en cada linea convertido a dolares usando la tabla de 
tipo de cambio.

select 
	ols.*,
	round(ols.venta/(
		case 
			when moneda = 'ARS' then maf.cotizacion_usd_peso
			when moneda = 'EUR' then maf.cotizacion_usd_eur
			when moneda = 'URU' then maf.cotizacion_usd_uru
			else 0 
		end)
	,2) as valor_en_dolares
from stg.order_line_sale as ols 
left join stg.monthly_average_fx_rate as maf 
on extract(month from maf.mes) = extract(month from ols.fecha)

9)Calcular cantidad de ventas totales de la empresa en dolares.

select 
sum(ols.venta/(
	case 
		when moneda = 'ARS' then maf.cotizacion_usd_peso
		when moneda = 'EUR' then maf.cotizacion_usd_eur
		when moneda = 'URU' then maf.cotizacion_usd_uru
		else 0 
	end)) as vta_bruta_en_dolares
from stg.order_line_sale ols 
left join stg.monthly_average_fx_rate as maf 
on extract(month from maf.mes) = extract(month from ols.fecha)

10)Mostrar en la tabla de ventas el margen de venta por cada linea. Siendo margen = (venta - promociones) - costo expresado en dolares.

select 
	ols.*,
	(round((ols.venta-coalesce(ols.descuento,0))/(
		case 
			when moneda = 'EUR' then maf.cotizacion_usd_eur
			when moneda = 'ARS' then maf.cotizacion_usd_peso
			when moneda = 'URU' then maf.cotizacion_usd_uru
			else 0 
		end),2)) - costo.costo_promedio_usd as margen_ganancia_usd
from stg.order_line_sale ols 
left join stg.cost as costo 
on costo.codigo_producto = ols.producto
left join stg.monthly_average_fx_rate as maf 
on extract(month from maf.mes) = extract(month from ols.fecha) 

11)Calcular la cantidad de items distintos de cada subsubcategoria que se llevan por numero de orden.

select 
	orden, 
	subcategoria, 
	count(distinct producto) as distinct_products
from stg.order_line_sale as ols
left join stg.product_master as pm
on ols.producto = pm.codigo_producto
group by orden, subcategoria


==================
EJERCICIOS CLASE 4
==================
1)Crear un backup de la tabla product_master. Utilizar un esquema llamada "bkp" y agregar un prefijo al nombre de la tabla con la fecha del backup en forma 
de numero entero.

create schema bkp;
select *, current_date as backup_date 
into bkp.product_master_20230410
from stg.product_master

2)Hacer un update a la nueva tabla (creada en el punto anterior) de product_master agregando la leyendo "N/A" para los valores null de material y color. 
Pueden utilizarse dos sentencias.

update bkp.product_master_20230410
set color = 'N/A' 
where color is null;

update bkp.product_master_20230410
set material = 'N/A' 
where material is null;

3)Hacer un update a la tabla del punto anterior, actualizando la columa "is_active", desactivando todos los productos en la subsubcategoria "Control Remoto".

update bkp.product_master_20230410 
set is_Active = 'false'
where subsubcategoria = 'Control remoto'

4)Agregar una nueva columna a la tabla anterior llamada "is_local" indicando los productos producidos en Argentina y fuera de Argentina.

alter table bkp.product_master_20230410 	
add column is_local boolean;

update bkp.product_master_20230410 	
set is_local = false 
where origen != 'Argentina';

update bkp.product_master_20230410 	
set is_local = true 
where origen = 'Argentina';

5)Agregar una nueva columna a la tabla de ventas llamada "line_key" que resulte ser la concatenacion de el numero de orden y el codigo de producto.

alter table stg.order_line_sale 	
add column line_key varchar;
update stg.order_line_sale
set line_key = concat(orden,producto);

6)Eliminar todos los valores de la tabla "order_line_sale" para el POS 1.

delete from stg.order_line_sale
where pos = 1

7)Crear una tabla llamada "employees" (por el momento vacia) que tenga un id (creado de forma incremental), nombre, apellido, fecha de entrada, fecha salida, 
telefono, pais, provincia, codigo_tienda, posicion. Decidir cual es el tipo de dato mas acorde.

CREATE TABLE stg.employees (
    id serial PRIMARY KEY,
    nombre varchar(255),
    apellido varchar(255),
    fecha_entrada date,
    fecha_salida date,
    telefono varchar(255),
    pais varchar(255),
    provincia varchar(255),
    codigo_tienda varchar(255),
    posicion varchar(255)
)

8)Insertar nuevos valores a la tabla "employees" para los siguientes 4 empleados:
-Juan Perez, 2022-01-01, telefono +541113869867, Argentina, Santa Fe, tienda 2, Vendedor.
-Catalina Garcia, 2022-03-01, Argentina, Buenos Aires, tienda 2, Representante Comercial
-Ana Valdez, desde 2020-02-21 hasta 2022-03-01, España, Madrid, tienda 8, Jefe Logistica
-Fernando Moralez, 2022-04-04, España, Valencia, tienda 9, Vendedor.

insert into stg.employees (nombre,apellido,fecha_entrada,fecha_salida,telefono,pais,provincia,codigo_tienda,posicion)
values  
 	('Juan','Perez','2022-01-01',null,'+541113869867','Argentina','Santa Fe','tienda 2','Vendedor'),
 	('Fernando','Moralez','2022-04-04',null,null,'España','Valencia','tienda 9','Vendedor'),
	('Ana','Valdez','2022-02-21','2022-03-01',null,'España','Madrid','tienda 8','Jefe Logistica'),
	('Catalina','Garcia','2022-03-01',null,null,'Argentina','Buenos Aires','tienda 2','Representante Comercial')

9)Crear un backup de la tabla "cost" agregandole una columna que se llame "last_updated_ts" que sea el momento exacto en el cual estemos realizando 
el backup en formato datetime.

select *, now() as last_updated_ts 
INTO bkp.cost_20230410
FROM stg.cost

10)El cambio en la tabla "order_line_sale" en el punto 6 fue un error y debemos volver la tabla a su estado original, como lo harias?

Si se hizo un respaldo previo se puede recuperar de ahí. Sino lo que se puede hacer es volver a cargar los valores con INSERT INTO...
