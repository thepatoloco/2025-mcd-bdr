# Base de datos relacionales

## Tarea 7

### Instrucciones

- Revisa inconsistencias en tu base de datos
- Haz modificaciones o ajustes que faciliten la manipulación de tu base de datos usando lenguaje SOL
- Utiliza subconsultas para responder preguntas relevantes de tus datos

### Revisión de inconsitencias

#### Verificar mensajes huérfanos (sin usuario válido)

```sql
SELECT m.id, m.content, m.user_id, m.group_id, m.created_at
FROM bdr_messages.messages m
LEFT JOIN bdr_messages.users u ON m.user_id = u.id
WHERE m.user_id IS NOT NULL AND u.id IS NULL;
```

#### Verificar mensajes en grupos inexistentes

```sql
SELECT m.id, m.content, m.user_id, m.group_id, m.created_at
FROM bdr_messages.messages m
LEFT JOIN bdr_messages.groups g ON m.group_id = g.id
WHERE g.id IS NULL;
```

#### Verificar relaciones group_user inconsistentes

```sql
SELECT gu.group_id, gu.user_id
FROM bdr_messages.group_user gu
LEFT JOIN bdr_messages.users u ON gu.user_id = u.id
WHERE u.id IS NULL;

SELECT gu.group_id, gu.user_id
FROM bdr_messages.group_user gu
LEFT JOIN bdr_messages.groups g ON gu.group_id = g.id
WHERE g.id IS NULL;
```

### Modificaciones y ajustes

#### Agregar restricciones de validación

```sql
-- Agregar check constraint para evitar contenido vacío
ALTER TABLE bdr_messages.messages 
ADD CONSTRAINT chk_content_or_image 
CHECK (content IS NOT NULL OR image_url IS NOT NULL);

-- Agregar check constraint para nombres no vacíos
ALTER TABLE bdr_messages.users 
ADD CONSTRAINT chk_user_name_not_empty 
CHECK (name IS NOT NULL AND LENGTH(TRIM(name)) > 0);

ALTER TABLE bdr_messages.groups 
ADD CONSTRAINT chk_group_name_not_empty 
CHECK (name IS NOT NULL AND LENGTH(TRIM(name)) > 0);
```

### Subconsultas para preguntas mas relevantes

#### Usuarios más activos por grupo

```sql
SELECT 
    g.name as group_name,
    subq.user_name,
    subq.message_count
FROM bdr_messages.groups g
JOIN (
    SELECT 
        m.group_id,
        u.name as user_name,
        COUNT(*) as message_count,
        ROW_NUMBER() OVER (PARTITION BY m.group_id ORDER BY COUNT(*) DESC) as rn
    FROM bdr_messages.messages m
    JOIN bdr_messages.users u ON m.user_id = u.id
    GROUP BY m.group_id, m.user_id, u.name
) subq ON g.id = subq.group_id
WHERE subq.rn = 1
ORDER BY subq.message_count DESC;
```

#### Mensajes que generaron más respuestas

```sql
SELECT 
    original.id,
    original.content,
    u.name as author_name,
    g.name as group_name,
    original.created_at,
    subq.reply_count
FROM bdr_messages.messages original
JOIN bdr_messages.users u ON original.user_id = u.id
JOIN bdr_messages.groups g ON original.group_id = g.id
JOIN (
    SELECT 
        answers_to,
        COUNT(*) as reply_count
    FROM bdr_messages.messages
    WHERE answers_to IS NOT NULL
    GROUP BY answers_to
    HAVING COUNT(*) >= 2 
) subq ON original.id = subq.answers_to
ORDER BY subq.reply_count DESC, original.created_at DESC;
```
