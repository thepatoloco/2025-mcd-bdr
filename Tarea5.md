# Base de datos relacionales

## Tarea 5

### Instrucciones

- Agregar datos ficticios o de otras fuentes de manera automática

### Creación de valores ficticios

```postgresql
-- Crear Usuarios
INSERT INTO bdr_messages.users (id, name)
SELECT
    i,
    md5(random()::text)
FROM generate_series(1,50) i;

-- Crear grupos
INSERT INTO bdr_messages.groups (id, name)
SELECT
    i,
    md5(random()::text)
FROM generate_series(1,10) i;

-- Añadir usuarios a grupos
INSERT INTO bdr_messages.group_user (group_id, user_id)
SELECT DISTINCT
    g.id AS group_id,
    u.id AS user_id
FROM (SELECT id FROM bdr_messages.users ORDER BY random()) u,
    LATERAL (SELECT id FROM bdr_messages.groups ORDER BY random()) g
WHERE random() < 0.4
LIMIT 200
ON CONFLICT DO NOTHING;

-- Crear mensajes
INSERT INTO bdr_messages.messages (content, group_id, user_id)
SELECT
    md5(random()::text) AS content,
    gu.group_id AS group_id,
    gu.user_id AS user_id
FROM generate_series(1,1000) i,
    LATERAL (
        SELECT group_id, user_id 
        FROM bdr_messages.group_user 
        ORDER BY random() 
        LIMIT 1 
        OFFSET (i-1) % (SELECT count(*) FROM bdr_messages.group_user)
    ) gu;

-- Crear mensajes respuesta
INSERT INTO bdr_messages.messages (content, group_id, user_id, answers_to)
SELECT
    md5(random()::text) AS content,
    gu.group_id AS group_id,
    gu.user_id AS user_id,
    m.id AS answers_to
FROM generate_series(1,300) i,
    LATERAL (
        SELECT group_id, user_id 
        FROM bdr_messages.group_user 
        ORDER BY random() 
        LIMIT 1 
        OFFSET (i-1) % (SELECT count(*) FROM bdr_messages.group_user)
    ) gu,
    LATERAL (
        SELECT id 
        FROM bdr_messages.messages 
        ORDER BY random() 
        LIMIT 1 
        OFFSET (i-1) % (SELECT count(*) FROM bdr_messages.messages)
    ) m;
```
