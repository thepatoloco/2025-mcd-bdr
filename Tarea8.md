# Base de datos relacionales

## Tarea 8

### Instrucciones

- Crear vistas (VIEW) sobre consultas significativas, recurrentes, etc. que: incluyan un JOIN, incluyan un LEFT JOIN, incluyan un RIGHT JOIN, incluyan una subconsulta
- Investigar y crear al menos un disparador (TRIGGER) de inserción, actualización o eliminación

### Creacion de vistas

#### Mensajes con contexto completo

```sql
CREATE OR REPLACE VIEW bdr_messages.v_messages_complete AS
SELECT 
    m.id as message_id,
    m.content,
    m.image_url,
    m.created_at,
    u.id as user_id,
    u.name as user_name,
    g.id as group_id,
    g.name as group_name,
    CASE 
        WHEN m.answers_to IS NOT NULL THEN 'answer'
        ELSE 'message'
    END as message_type
FROM bdr_messages.messages m
INNER JOIN bdr_messages.users u ON m.user_id = u.id
INNER JOIN bdr_messages.groups g ON m.group_id = g.id;

SELECT * FROM bdr_messages.v_messages_complete;
```

#### Grupos con nivel de actividad

```sql
CREATE OR REPLACE VIEW bdr_messages.v_groups_activity AS
SELECT 
    g.id as group_id,
    g.name as group_name,
    COUNT(m.id) as total_messages,
    COUNT(DISTINCT m.user_id) as active_users,
    MAX(m.created_at) as last_message_date,
    MIN(m.created_at) as first_message_date,
    CASE 
        WHEN COUNT(m.id) <= 0 THEN 'no activity'
        WHEN COUNT(m.id) < 100 THEN 'low activity '
        WHEN COUNT(m.id) < 150 THEN 'mid activity'
        ELSE 'high activity'
    END as activity_level
FROM bdr_messages.groups g
LEFT JOIN bdr_messages.messages m ON g.id = m.group_id
GROUP BY g.id, g.name
ORDER BY total_messages DESC;

SELECT * FROM bdr_messages.v_groups_activity;
```

#### Usuarios con nivel de actividad

```sql
CREATE OR REPLACE VIEW bdr_messages.v_users_participation AS
SELECT 
    u.id as user_id,
    u.name as user_name,
    COUNT(DISTINCT gu.group_id) as groups_joined,
    COUNT(m.id) as messages_sent,
    CASE 
        WHEN COUNT(m.id) <= 0 THEN 'ninja user'
        WHEN COUNT(m.id) < 30 THEN 'ghost user'
        WHEN COUNT(m.id) < 60 THEN 'mid user'
        ELSE 'super user'
    END as participation_level,
    MAX(m.created_at) as last_activity,
    CASE 
        WHEN COUNT(m.id) > (
            SELECT AVG(user_message_count) 
            FROM (
                SELECT COUNT(*) as user_message_count
                FROM bdr_messages.messages 
                WHERE user_id IS NOT NULL
                GROUP BY user_id
            ) avg_calc
        ) THEN 'over average'
        ELSE 'under average'
    END as compared_to_average
FROM bdr_messages.group_user gu
RIGHT JOIN bdr_messages.users u ON gu.user_id = u.id
LEFT JOIN bdr_messages.messages m ON u.id = m.user_id
GROUP BY u.id, u.name
ORDER BY messages_sent DESC;

SELECT * FROM bdr_messages.v_users_activity;
```

### Creación de trigger

Creación de un trigger que cuando se crea un usuario lo asigna a un grupo aleatorio.

```sql
-- Función trigger para asignar el ultimo usuario a un grupo
CREATE OR REPLACE FUNCTION bdr_messages.assign_random_group()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO bdr_messages.group_user (group_id, user_id)
    SELECT id, NEW.id
    FROM bdr_messages.groups
    ORDER BY RANDOM()
    LIMIT 1;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Trigger al crear usuario
CREATE TRIGGER trigger_assign_random_group
    AFTER INSERT ON bdr_messages.users
    FOR EACH ROW
    EXECUTE FUNCTION bdr_messages.assign_random_group();

-- Ejemplo para disparar trigger
INSERT INTO bdr_messages.users (id, name) VALUES (99, 'Pato Cantú');
```
