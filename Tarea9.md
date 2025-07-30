# Base de datos relacionales

## Tarea 9

### Instrucciones

- Usa lo aprendido hasta ahora para crear al menos dos funciones o procedimientos almacenados que calculen alguno de los siguientes resultados (o equivalentes):
- correlación entre dos conjuntos de datos, o
- regresión lineal entre dos variables, o
- distancia de levenshtein entre cadenas de caracteres, o
- cantidad de elementos de un arreglo, o
- seasonal naive para series de tiempo.

### Función para contar mensajes por grupo

```sql
CREATE OR REPLACE FUNCTION bdr_messages.count_group_messages(p_group_id INTEGER)
RETURNS INTEGER AS $$
DECLARE
    message_count INTEGER;
BEGIN
    SELECT COUNT(*) 
    INTO message_count
    FROM bdr_messages.messages 
    WHERE group_id = p_group_id;
    
    RETURN message_count;
END;
$$ LANGUAGE plpgsql;

SELECT bdr_messages.count_group_messages(3);
```

### Función para distancia levenshtein entre el contenido de 2 mensajes

```sql
CREATE EXTENSION IF NOT EXISTS fuzzystrmatch;

CREATE OR REPLACE FUNCTION bdr_messages.compare_messages(message_id1 INTEGER, message_id2 INTEGER)
RETURNS INTEGER AS $$
DECLARE
    content1 TEXT;
    content2 TEXT;
    distance INTEGER;
BEGIN
    SELECT content INTO content1 FROM bdr_messages.messages WHERE id = message_id1;
    SELECT content INTO content2 FROM bdr_messages.messages WHERE id = message_id2;
    
    IF content1 IS NULL THEN
        RAISE EXCEPTION 'could not find message %', message_id1;
    ELSIF content2 IS NULL THEN
        RAISE EXCEPTION 'could not find message %', message_id1;
    END IF;
    
    distance := levenshtein(content1, content2);
    
    RETURN distance;
END;
$$ LANGUAGE plpgsql;

SELECT bdr_messages.compare_messages(10, 11);
```
