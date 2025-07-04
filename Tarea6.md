# Base de datos relacionales

## Tarea 6

### Instrucciones

- Conteo de frecuencias o agregación
- Mínimos o máximos
- Cuantil cuyo resultado sea distinto a la mediana
- Moda

### Querys

#### Cantidad de mensajes por usuario

```postgresql
SELECT user_id, COUNT(*) AS message_quantity
FROM bdr_messages.messages
GROUP BY user_id
ORDER BY message_quantity DESC;
```

Resultado:

```text
user_id|message_quantity|
-------+----------------+
      4|              45|
      6|              44|
     19|              43|
     46|              40|
     44|              40|
     24|              39|
     11|              38|
     14|              37|
     20|              36|
... (50 row(s) fetched)
```

#### Fecha mínima y máxima de mensaje

```postgresql
SELECT MIN(created_at) AS minimum_date, MAX(created_at) AS maximum_date FROM bdr_messages.messages;
```

Resultado:

```text
minimum_date                 |maximum_date                 |
-----------------------------+-----------------------------+
2025-07-03 19:47:06.886 -0600|2025-07-03 19:48:56.946 -0600|
```

#### Cuartiles de cantidad de mensajes por grupo

```postgresql
SELECT
    percentile_cont(0.25) WITHIN GROUP (ORDER BY group_messages.message_quantity) AS q1,
    percentile_cont(0.50) WITHIN GROUP (ORDER BY group_messages.message_quantity) AS median,
    percentile_cont(0.75) WITHIN GROUP (ORDER BY group_messages.message_quantity) AS q3
FROM (
    SELECT group_id, COUNT(*) AS message_quantity
    FROM bdr_messages.messages
    GROUP BY group_id
) AS group_messages;
```

Resultado:

```text
q1   |median|q3   |
-----+------+-----+
103.0| 130.5|156.5|
```

#### El grupo con mas miembros

```postgresql
SELECT MODE() WITHIN GROUP (ORDER BY group_id) AS mode_group_id
FROM bdr_messages.group_user;
```

Resultado:

```text
mode_group_id|
-------------+
           10|
```
