![logo_ironhack_blue 7](https://user-images.githubusercontent.com/23629340/40541063-a07a0a8a-601a-11e8-91b5-2f13e4e6b441.png)

# Laboratorio | Subconsultas SQL

En este laboratorio, usarás la base de datos [Sakila](https://dev.mysql.com/doc/sakila/en/) de alquileres de películas. Crea las uniones apropiadas donde sea necesario.

### Entrega

Crea un archivo `.sql` donde recopilarás tus respuestas. Cópialas a medida que las pruebas en el workbench o códelas y ve a Archivo > Guardar script.

### Instrucciones

1. ¿Cuántas copias de la película _El Jorobado Imposible_ existen en el sistema de inventario?

SELECT COUNT(*) AS copias
FROM inventory
WHERE film_id = (
    SELECT film_id
    FROM film
    WHERE title = 'El Jorobado Imposible'
);

2. Lista todas las películas cuya duración sea mayor que el promedio de todas las películas.

SELECT title, length
FROM film
WHERE length > (
    SELECT AVG(length)
    FROM film
);

3. Usa subconsultas para mostrar todos los actores que aparecen en la película _Viaje Solo_.

SELECT first_name, last_name
FROM actor
WHERE actor_id IN (
    SELECT actor_id
    FROM film_actor
    WHERE film_id = (
        SELECT film_id
        FROM film
        WHERE title = 'Viaje Solo'
    )
);

4. Las ventas han estado disminuyendo entre las familias jóvenes, y deseas dirigir todas las películas familiares a una promoción.
Identifica todas las películas categorizadas como películas familiares.

SELECT f.title
FROM film f
JOIN film_category fc ON f.film_id = fc.film_id
JOIN category c ON fc.category_id = c.category_id
WHERE c.name = 'Family';


5. Obtén el nombre y correo electrónico de los clientes de Canadá usando subconsultas. Haz lo mismo con uniones. Ten en cuenta que para crear una unión, tendrás que identificar las tablas correctas con sus claves primarias y claves foráneas, que te ayudarán a obtener la información relevante.

SELECT first_name, last_name, email
FROM customer
WHERE address_id IN (
    SELECT address_id
    FROM address
    WHERE city_id IN (
        SELECT city_id
        FROM city
        WHERE country_id = (
            SELECT country_id
            FROM country
            WHERE country = 'Canada'
        )
    )
);


SELECT c.first_name, c.last_name, c.email
FROM customer c
JOIN address a ON c.address_id = a.address_id
JOIN city ci ON a.city_id = ci.city_id
JOIN country co ON ci.country_id = co.country_id
WHERE co.country = 'Canada';

6. ¿Cuáles son las películas protagonizadas por el actor más prolífico? El actor más prolífico se define como el actor que ha actuado en el mayor número de películas. Primero tendrás que encontrar al actor más prolífico y luego usar ese actor_id para encontrar las diferentes películas en las que ha protagonizado.

SELECT actor_id, COUNT(*) AS num_peliculas
FROM film_actor
GROUP BY actor_id
ORDER BY num_peliculas DESC
LIMIT 1;

SELECT f.title
FROM film f
JOIN film_actor fa ON f.film_id = fa.film_id
WHERE fa.actor_id = (
    SELECT actor_id
    FROM film_actor
    GROUP BY actor_id
    ORDER BY COUNT(*) DESC
    LIMIT 1
);

7. Películas alquiladas por el cliente más rentable. Puedes usar la tabla de clientes y la tabla de pagos para encontrar al cliente más rentable, es decir, el cliente que ha realizado la mayor suma de pagos.

SELECT customer_id, SUM(amount) AS total_pagado
FROM payment
GROUP BY customer_id
ORDER BY total_pagado DESC
LIMIT 1;


SELECT f.title
FROM rental r
JOIN inventory i ON r.inventory_id = i.inventory_id
JOIN film f ON i.film_id = f.film_id
WHERE r.customer_id = (
    SELECT customer_id
    FROM payment
    GROUP BY customer_id
    ORDER BY SUM(amount) DESC
    LIMIT 1
);

8. Obtén el `client_id` y el `total_amount_spent` de esos clientes que gastaron más que el promedio del `total_amount` gastado por cada cliente.

WITH total_gastos AS (
    SELECT customer_id, SUM(amount) AS total_amount
    FROM payment
    GROUP BY customer_id
)
SELECT customer_id, total_amount
FROM total_gastos
WHERE total_amount > (
    SELECT AVG(total_amount)
    FROM total_gastos
);
