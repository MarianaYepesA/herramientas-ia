# Checklist de Revisión

## 1. Alucinaciones de librerías

### Pregunta clave

¿Ese import realmente existe?

La IA puede inventar:

- librerías
- funciones
- APIs
- métodos

**Verificar:**

- documentación oficial
- existencia real del paquete
- compatibilidad con la versión usada

### Checklist

- [ ] Los imports existen
- [ ] Las funciones usadas son reales
- [ ] La API corresponde a la versión actual

## 2. Lógica de negocio sutil

### Pregunta clave

¿La lógica es realmente correcta?

**Errores comunes generados por IA:**

- cálculos incorrectos
- redondeos incorrectos
- manejo incorrecto de fechas
- uso de float para dinero

### Checklist

- [ ] Los cálculos son correctos
- [ ] El manejo de fechas es consistente
- [ ] No se usa float para dinero
- [ ] Edge cases están contemplados

## 3. Seguridad

### Pregunta clave

¿El código introduce riesgos de seguridad?

La IA puede generar código funcional pero inseguro.

**Revisar:**

- validación de inputs
- inyección (SQL, comandos, etc.)
- exposición de secretos
- manejo de datos sensibles

### Checklist

- [ ] Inputs están validados
- [ ] No hay riesgo de inyección
- [ ] No se exponen credenciales
- [ ] No se filtran datos sensibles

## 4. Context window

### Pregunta clave

¿La IA olvidó algo del brief?

Cuando el contexto es largo la IA puede ignorar:

- constraints
- decisiones arquitectónicas
- requisitos técnicos
- Definition of Done

### Checklist

- [ ] El código respeta el brief original
- [ ] Se cumplen los constraints definidos
- [ ] No se agregaron dependencias innecesarias
- [ ] Se cumple la Definition of Done

## 5. Punto personalizado del proyecto

Este punto depende de tu stack o arquitectura.

**Ejemplos comunes:**

- cobertura de tests
- métricas y observabilidad
- logging
- performance
- compatibilidad con arquitectura

### Checklist

- [ ] Tests cubren el código nuevo
- [ ] Logs no exponen datos sensibles
- [ ] Performance es aceptable
- [ ] Métricas funcionan correctamente

## Resultado del Review

- [ ] El código pasa los 5 puntos del protocolo
- [ ] Se hicieron correcciones necesarias
- [ ] El código está listo para commit

**Reviewer:** ____________________  
**Fecha:** ____________________

---

## Uso dentro del repositorio

**Estructura recomendada:**

```
herramientas-ia/
│
├─ 01-planning/
│   ├─ plantilla-brief-ia.md
│   └─ protocolo-review-ia.md
│
└─ reflexion-semana-1.md
```
