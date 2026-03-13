¿Cómo crear un protocolo de review para código generado por IA?
El segundo artefacto clave es el Protocolo de Review, una checklist fija de 5 puntos críticos que revisas antes de hacer commit.

Su propósito es simple: detectar errores comunes en código generado por IA antes de que lleguen al repositorio.

Cada punto incluye:

Una pregunta clave.
Qué revisar exactamente.
¿Cómo detectar alucinaciones de librerías?
Las alucinaciones ocurren cuando la IA usa librerías o funciones inexistentes.

Preguntas clave:

¿Ese import realmente existe?
¿La librería es segura y mantenida?
Qué revisar:

Documentación oficial.
Repositorio del paquete.
Compatibilidad con tu entorno.
¿Cómo revisar errores en lógica de negocio?
La IA suele fallar en detalles matemáticos o reglas de negocio.

Ejemplos frecuentes:

Uso incorrecto de floats para dinero.
Redondeos erróneos.
Cálculos acumulativos incorrectos.
Qué revisar:

Fórmulas críticas.
Condiciones límite.
Casos edge.
¿Qué revisar en seguridad antes de hacer commit?
La seguridad debe verificarse incluso si el código parece correcto.

Busca problemas como:

Inyección SQL.
Inputs sin validación.
Credenciales expuestas.
Manejo incorrecto de autenticación.
La IA puede generar código funcional pero inseguro.

¿Cómo detectar pérdida de contexto en la respuesta de la IA?
A veces la IA olvida partes del brief cuando el contexto es largo.

Preguntas clave:

¿Se respetaron todos los constraints?
¿La solución sigue los requerimientos del brief?
Qué revisar:

Dependencias permitidas.
Estructura esperada.
Reglas del proyecto.
¿Qué quinto punto debes personalizar según tu stack?
El último punto debe adaptarse a tu entorno técnico.

Ejemplos:

Verificar que los tests nuevos se ejecuten correctamente.
Revisar que no se registren datos sensibles en logs.
Confirmar que el código respete estándares de arquitectura del proyecto.
Este punto convierte el protocolo en una herramienta alineada con tu stack real.
