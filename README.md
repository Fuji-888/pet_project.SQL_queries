# SQL запросы

Я буду делать различные манипуляции с этой базой данных, чтобы показать свои навыки владения языком SQL . 

##  Описание базы данных

База данных состоит из 4х таблиц:
* Product(maker, model, type)
* PC(code, model, speed, ram, hd, cd, price)
* Laptop(code, model, speed, ram, hd, price, screen)
* Printer(code, model, color, type, price)

	* Таблица Product представляет производителя (maker), номер модели (model) и тип ('PC' - ПК, 'Laptop' - ПК-блокнот или 'Printer' - принтер). Предполагается, что номера моделей в таблице Product уникальны для всех производителей и типов продуктов. 
	* В таблице PC для каждого ПК, однозначно определяемого уникальным кодом – code, указаны модель – model (внешний ключ к таблице Product), скорость - speed (процессора в мегагерцах), объем памяти - ram (в мегабайтах), размер диска - hd (в гигабайтах), скорость считывающего устройства - cd (например, '4x') и цена - price (в долларах). 
	* Таблица Laptop аналогична таблице РС за исключением того, что вместо скорости CD содержит размер экрана -screen (в дюймах). 
	* В таблице Printer для каждой модели принтера указывается, является ли он цветным - color ('y', если цветной), тип принтера - type (лазерный – 'Laser', струйный – 'Jet' или матричный – 'Matrix') и цена - price.

## SQL запросы

#### Задание: 1
Найдите номер модели, скорость и размер жесткого диска для всех ПК стоимостью менее 500 дол. Вывести: model, speed и hd

```sql
select
	model, speed, hd
from
	pc
where
	price < 500
```

#### Задание: 2
Найдите производителей принтеров. Вывести: maker
```sql
select
	distinct maker
from
	product
where
	type = 'printer'
```

#### Задание: 3
Найдите номер модели, объем памяти и размеры экранов ПК-блокнотов, цена которых превышает 1000 дол.
```sql
select
	model, ram, screen
from
	laptop
where
	price>1000
```

#### Задание: 4
Найдите все записи таблицы Printer для цветных принтеров.
```sql
select
	*
from
	printer
where
	color = 'y'
```

#### Задание: 5 
Найдите номер модели, скорость и размер жесткого диска ПК, имеющих 12x или 24x CD и цену менее 600 дол.
```sql
select
	model, speed, hd
from
	pc
where
	cd in ('12x','24x')
	and
	price<600
```

#### Задание: 6 
Для каждого производителя, выпускающего ПК-блокноты c объёмом жесткого диска не менее 10 Гбайт, найти скорости таких ПК-блокнотов. Вывод: производитель, скорость.
```sql
select
	distinct maker,
	speed
from
	product pr
inner join
	laptop l
	   on pr.model = l.model
where
	hd>=10
```

#### Задание: 7
Найдите номера моделей и цены всех имеющихся в продаже продуктов (любого типа) производителя B (латинская буква).
```sql
SELECT
	 model,price
FROM 
	(SELECT model, price 
	 FROM PC
	 UNION
	 SELECT model, price 
	 FROM Laptop
	 UNION
	 SELECT model, price 
	 FROM Printer) AS a
WHERE a.model IN 
	(SELECT model
	FROM product
	where maker='B')
```

#### Задание: 8 
Найдите производителя, выпускающего ПК, но не ПК-блокноты.
```sql
select
	DISTINCT maker
from
	product
where
	type = 'PC'
	and
	maker not in (select maker from product where type = 'Laptop')
```

#### Задание: 9 
Найдите производителей ПК с процессором не менее 450 Мгц. Вывести: Maker
```sql
select
	distinct maker
from
	product pr
inner join
	pc
	   on pr.model = pc.model
where
	speed>=450
```

#### Задание: 10 
Найдите модели принтеров, имеющих самую высокую цену. Вывести: model, price
```sql
select
	model, price
from
	printer
where
	price in (select max(price) from printer)
```

#### Задание: 11 
Найдите среднюю скорость ПК.
```sql
select
	AVG(speed) as sredn_skor
from
	pc
```

#### Задание: 12
Найдите среднюю скорость ПК-блокнотов, цена которых превышает 1000 дол.
```sql
select avg(speed)
from ( select
	speed
	from laptop
	where price>1000) as t
```

#### Задание: 13 
Найдите среднюю скорость ПК, выпущенных производителем A.

```sql
select
	AVG(speed)
FROM(SELECT
	speed
	FROM
	product pr
	INNER JOIN
	pc ON pr.model=pc.model
	WHERE
	maker='A') as tf
```

#### Задание: 15 
Найдите размеры жестких дисков, совпадающих у двух и более PC. Вывести: HD
```sql
select
	hd
from
	PC 
GROUP BY
	hd
HAVING
	count(code) >1
```

#### Задание: 17 
Найдите модели ПК-блокнотов, скорость которых меньше скорости каждого из ПК.
Вывести: type, model, speed
```sql
SELECT
	DISTINCT 'Laptop' AS type, model, speed
FROM
	Laptop
WHERE
	speed < (SELECT MIN(speed) FROM PC)
```

#### Задание: 18 
Найдите производителей самых дешевых цветных принтеров. Вывести: maker, price
```sql
SELECT
	product.maker,minzn.min_price
FROM
	(SELECT
	MIN(pr.price) min_price
	FROM
	printer pr
	WHERE pr.color='y') minzn
JOIN printer pr2 on minzn.min_price=pr2.price and color='y'
JOIN product ON pr2.model=product.model
```

#### Задание: 19
Для каждого производителя, имеющего модели в таблице Laptop, найдите средний размер экрана выпускаемых им ПК-блокнотов.
Вывести: maker, средний размер экрана.

```sql
SELECT
	maker,AVG(screen)
FROM
	product
JOIN
	laptop on product.model=laptop.model
where maker IN (SELECT 
			DISTINCT maker
		FROM
			product
		WHERE
			type='laptop')
GROUP BY
maker
```

#### Задание: 20 
Найдите производителей, выпускающих по меньшей мере три различных модели ПК. Вывести: Maker, число моделей ПК.
```sql
SELECT
	maker, COUNT(model) as cont
FROM
	(SELECT
	distinct model, type, maker
FROM
	product) pr1
WHERE type='pc'
GROUP BY
	maker
HAVING
	COUNT(model) >=3
```

#### Задание: 21 
Найдите максимальную цену ПК, выпускаемых каждым производителем, у которого есть модели в таблице PC.
Вывести: maker, максимальная цена.
```sql
SELECT
	pr.maker, max(pc2.maxpr) as max_sale
FROM
	product pr
JOIN (
	SELECT 
		model, MAX(price) as maxpr
	FROM 
		PC
	GROUP BY
		model) PC2 ON pr.model=pc2.model
GROUP BY
	pr.maker
```

#### Задание: 22 
Для каждого значения скорости ПК, превышающего 600 МГц, определите среднюю цену ПК с такой же скоростью. Вывести: speed, средняя цена.
```sql
SELECT
	speed, AVG(price) as avg_price
FROM
	pc
WHERE
	speed>600
GROUP BY
speed
```

#### Задание: 23 
Найдите производителей, которые производили бы как ПК
со скоростью не менее 750 МГц, так и ПК-блокноты со скоростью не менее 750 МГц.
Вывести: Maker

```sql
SELECT
	pr.maker
FROM
	product pr
LEFT JOIN PC ON pr.model=pc.model and pc.speed>=750
LEFT JOIN laptop lp ON pr.model=lp.model and lp.speed>=750
WHERE
	pc.code IS NOT NULL OR lp.code IS NOT NULL
GROUP BY
	pr.maker
HAVING
	count(pc.model) >0 and count(lp.model)>0


select
	distinct maker
from
	Product a
JOIN PC b ON a.model =b.model
where
	speed > = 750
intersect
select
	 distinct maker
from
	Product a
JOIN Laptop b ON a.model =b.model
where
	 speed > = 750
```

#### Задание: 25 
Найдите производителей принтеров, которые производят ПК с наименьшим объемом RAM и с самым быстрым процессором среди всех ПК, имеющих наименьший объем RAM. Вывести: Maker
—Найти производителей ПК с заданными характеристиками и проверить какие из этих производителей делают принтеры.
```sql
SELECT DISTINCT maker
FROM product
WHERE type='printer' AND maker IN
	(SELECT maker
	FROM product
	WHERE model IN
		(SELECT model
		FROM PC
		WHERE speed=(
			SELECT max(speed)
			FROM PC 
			WHERE speed IN
				(SELECT speed 
				FROM PC 
				WHERE ram=(SELECT MIN(ram)
		         	 	  FROM PC)))
		AND ram=(SELECT MIN(ram)
		         FROM PC)
		))
```


#### Задание: 26 
Найдите среднюю цену ПК и ПК-блокнотов, выпущенных производителем A (латинская буква). Вывести: одна общая средняя цена.
```sql
SELECT AVG(a.price)
FROM
	(SELECT pc1.price
	FROM PC pc1
	WHERE pc1.model IN
		(SELECT pr1.model
		FROM product pr1
		WHERE pr1.maker='A' AND pr1.type='PC')
UNION ALL
SELECT
	lp.price
FROM
	laptop lp
WHERE lp.model IN
	(SELECT pr2.model
	FROM product pr2
	WHERE pr2.maker='A' AND pr2.type='laptop')) a
```

#### Задание: 27
Найдите средний размер диска ПК каждого из тех производителей, которые выпускают и принтеры. Вывести: maker, средний размер HD.
```sql
SELECT
	maker, AVG(hd)
FROM
	(
	SELECT maker,hd
	FROM
		(
		SELECT maker,model
			FROM product as pr
			WHERE maker IN
				(
				SELECT DISTINCT maker
				FROM product
				WHERE maker IN (SELECT maker
				FROM product
				WHERE type='printer')
				AND type='PC'
				)
		) a
	JOIN PC ON a.model=pc.model
	) as a2
GROUP BY maker
```

#### Задание: 28
Используя таблицу Product, определить количество производителей, выпускающих по одной модели.
```sql
select
	count(\*)
from
	(
	select maker, count(\*) as countmod
	from product
	group by maker
	having count(\*)=1
	) a
```

#### Задание: 29 
В предположении, что приход и расход денег на каждом пункте приема фиксируется не чаще одного раза в день \[т.е. первичный ключ (пункт, дата)\], написать запрос с выходными данными (пункт, дата, приход, расход). Использовать таблицы Income_o и Outcome_o.

```sql
select 
	CASE
		when a.point is null
		then cast (b.point as int)
		else cast(a.point as int)
	end point,
	CASE
		when a.date is null
		then cast (b.date as datetime)
		else cast(a.date as datetime)
	end date,
	b.inc,a.out
from
	outcome_o a
full join income_o b on a.date=b.date and a.point=b.point
```

#### Задание: 30 
В предположении, что приход и расход денег на каждом пункте приема фиксируется произвольное число раз (первичным ключом в таблицах является столбец code), требуется получить таблицу, в которой каждому пункту за каждую дату выполнения операций будет соответствовать одна строка.
Вывод: point, date, суммарный расход пункта за день (out), суммарный приход пункта за день (inc). Отсутствующие значения считать неопределенными (NULL).

```sql
SELECT
	point_, date_,SUM(out),SUM(inc)
FROM
	(select i.point,
	case
		when i.point is NULL
		then cast(o.point as int)
		else cast(i.point as int)
	end point_,
	case
		when i.date is NULL
		then cast(o.date as datetime)
		else cast(i.date as datetime)
	end date_,
	i.inc,o.out
	from (select point,date,sum(inc) as inc from income group by point,date) as i
	full join (select point,date,sum(out) as out from outcome group by point,date) as o on i.date=o.date and i.point=o.point
	) as tabl
group by
	 point_,date_
```
