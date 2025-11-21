# project-management-ute
Esta pequeña aplicacion esta enfocada en ayudar a organizar los proyectos personales del usuario, usando la metodologia agil de gestion de proyectos kanban.

1. Reglas de Jerarquía y Estructura

Estas reglas definen dónde vive el trabajo y cómo se relacionan los contenedores.

Regla de Pertenencia Única: Todo trabajo debe pertenecer a un Workspace (Organización). No existen "tareas huérfanas" fuera de un entorno de trabajo.

La Regla del Contenedor de Tareas: Una Task (Tarea) siempre debe pertenecer directamente a una List (Lista).

Las tareas no pueden vivir directamente en una Folder (Carpeta) ni en un Space (Espacio).

La Regla de la Carpeta Opcional: Una List puede existir directamente dentro de un Space, o puede estar agrupada dentro de una Folder. El campo folder_id es opcional (nullable).

Integridad de Subtareas:

Una subtarea debe residir en la misma List que su tarea padre (para evitar conflictos de estados/permisos complejos).

Eliminación en Cascada: Si se elimina una tarea padre, todas sus subtareas deben eliminarse (o archivarse) automáticamente.

Nivel de Profundidad: (Opcional) Se recomienda limitar la anidación a 2 niveles.

2. Reglas de Estados y Flujos de Trabajo (Status Workflow)

Esta es la lógica más crítica para la consistencia visual (Kanban/Listas).

Herencia de Estados (The Inheritance Logic):

Los estados se definen a nivel de Space (por defecto) o a nivel de List (específico).

Lógica de Override: Si una Lista tiene registros de estadus vinculados a su id, el sistema ignora los estados del Espacio y usa solo los de la Lista.

Si la Lista no tiene estados propios, hereda obligatoriamente los del Espacio.

Categorización Obligatoria: Todo estado personalizado (ej. "En Revisión", "QA", "Bloqueado") debe mapearse a una de las tres "Categorías Meta":

TO DO (Pendiente/No iniciado)

DOING (Activo/En progreso)

DONE (Completado/Cerrado)

Estados Cerrados: Una tarea se considera "completa" solo si su estado actual pertenece a la categoría DONE.

3. Reglas de Datos Híbridos (Split-Brain Management)

Reglas que gobiernan la sincronización entre SQL (PostgreSQL) y NoSQL (MongoDB/JSONB).

Creación Atómica: No puede existir un documento en NoSQL (task_attributes) sin su correspondiente fila en SQL (tasks). Ambas deben crearse en la misma transacción lógica.

Inmutabilidad de Definiciones:

La definición de un Campo Personalizado (custom_field_definitions) vive en SQL (nombre, tipo, config).

El valor del campo vive en NoSQL (custom_fields_values).

Si se borra la definición en SQL, el valor en NoSQL se vuelve "fantasma" (debe limpiarse o ignorarse en el frontend).

Tipado de Campos Personalizados:

Si un campo es tipo NUMBER, el sistema debe validar que el valor en el JSON sea numérico antes de guardar.

4. Reglas de Interacción y Vistas (Views)

Reglas para la capa de usuario y filtrado.

Persistencia de Vistas: Las Saved Views (Vistas Guardadas) son configuraciones de UI. No alteran los datos reales de las tareas.

Privacidad de Vistas: Una vista puede ser Privada (solo visible por el user_id creador) o Pública (visible por todos en la Lista/Espacio).

Filtros de Atributos: Los filtros aplicados en una vista operan principalmente sobre la colección NoSQL (ej. "filtrar por fecha > hoy"). Para rendimiento, los índices en Mongo/JSONB deben coincidir con los campos más filtrados (priority, assignee, due_date).

5. Reglas de Asignación y Seguridad

Acceso a Tareas: Un usuario solo puede ver una tarea si tiene permiso de lectura sobre el Workspace y el Space donde reside la tarea.

Asignación Múltiple: Una tarea puede tener múltiples responsables (assignees en NoSQL es un Array), pero solo un creador (created_by en SQL es un UUID único).

Etiquetas (Tags): Las etiquetas se definen a nivel de Space (Biblioteca de etiquetas). Una tarea en una Lista A y una tarea en una Lista B pueden compartir la misma etiqueta si ambas listas están en el mismo Espacio.

Resumen Técnico para Validadores (Validators Summary)

Si implementas esto en código, tus validadores deben chequear:

Al crear Lista: ¿Hereda estados o define nuevos?

Al mover Tarea: ¿La nueva Lista tiene el mismo status_id? Si no, hay que mapear el estado antiguo al estado equivalente en la nueva lista.

Al asignar Campo Personalizado: ¿Existe la definition_id en el contexto (Espacio/Lista) de esta tarea?

¿Te gustaría que escriba los validadores de Pydantic (@field_validator) para alguna de estas reglas específicas?