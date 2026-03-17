
---

# Technical Brief

## 1. Título de la tarea
**Agente de Pruebas (Test Agent)** para Tributi y Contadia.

---

## 2. Contexto
Durante la pre-temporada, el equipo de operaciones de Tributi realiza más de 2,000 pruebas manuales para asegurar que el motor tributario calcule correctamente los formularios (210, 110, etc.) bajo múltiples condiciones. 

Actualmente, este proceso es extremadamente repetitivo y manual. Para probar un solo caso (ej. Pensión Voluntaria), un operador debe:
1. Crear un correo de prueba.
2. Registrar un nuevo usuario en Tributi o Contadia.
3. Navegar por la app y llenar docenas de campos (muchas veces poniendo "0" repetitivamente solo para completar la información).
4. Configurar variables clave (ej. residencia en el exterior vs. Colombia) que cambian radicalmente el árbol de preguntas desplegado.

El objetivo de esta tarea es construir un Agente de IA (Test Agent) basado en código que entienda instrucciones en lenguaje natural (ej. *"Créame un usuario en Contadia, que viva en Colombia y tenga Pensión Voluntaria"*). El agente usará Tools (Herramientas) para crear el correo `@tributi.com`, registrar la cuenta y, de manera crucial, utilizar **Plantillas JSON (Fixtures)** predefinidas para llenar la data sin alucinar campos, modificando solo los parámetros que el operador solicitó.

---

## 3. Requerimientos técnicos

### Lenguaje / Stack

| Componente | Tecnología |
|---|---|
| Lenguaje | Python 3.11+ |
| Framework API | FastAPI |
| IA / LLM | Claude API (Anthropic) u OpenAI (para el ruteo de la intención) |
| Framework de Agentes | LangChain, LlamaIndex, o un ruteador semántico propio |
| Clientes HTTP | `httpx` (para comunicarse con el backend de Tributi/Contadia) |
| Testing | pytest + pytest-mock |
| Almacenamiento | Sistema de archivos local o S3 para los JSON de Plantillas (Fixtures) |

---

## 4. Arquitectura (Hexagonal)

### 1. DOMAIN (Objetos / Datos / Plantillas)
* **Contenido:** `TestScenario`, `UserFixture`, `TemplateMetadata`
* **Responsabilidad:** Define las estructuras de datos de las pruebas. Aquí viven los modelos que representan las plantillas con ceros por defecto (ej. `PensionVoluntariaTemplate`).

### 2. APPLICATION (Lógica del Agente y Flujo)
* **Contenido:** `TestAgentOrchestrator`, `ApplyTemplateUseCase`
* **Responsabilidad:** Recibe el *prompt* del operador, usa el LLM para extraer las variables (Intención = Crear, Plataforma = Contadia, Contexto = Nacional, Objeto = PV), selecciona la plantilla correcta y ejecuta el flujo.

### 3. INFRASTRUCTURE (Tools y Detalles Técnicos)
* **Contenido:** `TributiAPIClient`, `ContadiaAPIClient`, `EmailGeneratorTool`, `LLMIntentParser`
* **Responsabilidad:** Las herramientas reales que el agente ejecuta. Conexión con las APIs de Staging para registrar cuentas e inyectar las plantillas modificadas.

---

### Componentes principales (Esquema de Código)

#### DOMAIN (Entidades)
```python
# src/domain/TestScenario.py
from pydantic import BaseModel
from typing import Dict, Any

class TestScenario(BaseModel):
    platform: str  # "tributi" o "contadia"
    residency: str # "colombia" o "exterior"
    tax_objects: list[str] # ["pension_voluntaria", "dependientes"]
    overrides: Dict[str, Any] # Cambios específicos sobre la plantilla
```

#### APPLICATION (Las Tools del Agente)
```python
# src/application/tools/ProvisioningTools.py
class ProvisioningTools:
    
    def generate_tributi_email(self, scenario_name: str) -> str:
        """Genera un correo tipo test-pv-exterior-1234@tributi.com"""
        pass

    def load_and_modify_template(self, scenario: TestScenario) -> dict:
        """
        Carga la plantilla JSON (ej. PV) que tiene todo en 0, 
        y le cambia el flag de 'residencia_colombia' según el TestScenario.
        """
        pass
```

#### APPLICATION (El Orquestador)
```python
# src/application/TestAgentOrchestrator.py
class TestAgentOrchestrator:
    """Interpreta el prompt del operador de Operaciones y ejecuta las Tools"""
    
    def execute_from_prompt(self, prompt: str) -> str:
        # 1. LLM extrae variables estructuradas (TestScenario)
        scenario = self.llm_parser.parse_intent(prompt)
        
        # 2. Se genera el email
        email = self.tools.generate_tributi_email(scenario.tax_objects[0])
        
        # 3. Se crea el usuario en la plataforma correcta (Tributi o Contadia)
        user_id = self.api_client.register_user(scenario.platform, email)
        
        # 4. Se prepara la plantilla (ej. llena de ceros pero con residencia=exterior)
        payload = self.tools.load_and_modify_template(scenario)
        
        # 5. Se inyecta la data en el backend
        self.api_client.inject_user_data(user_id, payload)
        
        return f"¡Listo! Cuenta {email} creada en {scenario.platform} con los datos inyectados."
```

---

## 5. Input esperado

### Desde el Frontend Interno (Lo que envía el Operador)
```json
{
  "operator_prompt": "Créame una cuenta en Contadia para alguien en el exterior, necesita tener el objeto de Pensión Voluntaria y Rentas de Capital.",
  "environment": "staging"
}
```

### Lo que el LLM del Agente debe parsear (Structured Output)
```json
{
  "platform": "contadia",
  "residency": "exterior",
  "tax_objects": ["pension_voluntaria", "rentas_capital"]
}
```

---

## 6. Constraints (Restricciones)

### Restricciones de Dominio y Negocio
- **No Alucinación de Datos:** El LLM **NUNCA** debe generar el JSON de los activos o ingresos desde cero. **Siempre** debe basarse en las plantillas predefinidas en el repositorio.
- **Dominios Controlados:** Todos los correos generados deben terminar obligatoriamente en `@tributi.com`.
- **Ambientes Seguros:** El agente solo tendrá credenciales y acceso a las APIs de los ambientes de `Staging`, `QA` o `Desarrollo`. NUNCA en producción.

### Code Quality
- Seguir principios SOLID y la arquitectura hexagonal descrita.
- **Type hints obligatorios** en todo el proyecto.
- Manejo estricto de errores (ej. ¿Qué pasa si el operador pide una plantilla de un objeto que no existe?).

---

## 7. Definition of Done (DoD)

### Calidad de Código y Arquitectura
- [ ] El código pasa `flake8` y `black`.
- [ ] Implementación estricta de Type Hints en todas las capas.
- [ ] Separación clara entre los prompts del LLM, las Tools, y la lógica de negocio.

### Plantillas (Fixtures)
- [ ] Existe un directorio `src/infrastructure/templates/` con al menos las plantillas base JSON para flujos críticos (ej. `pension_voluntaria.json`, `exterior_base.json`).

### Testing
- [ ] Cobertura de tests unitarios ≥ 90% (`pytest --cov=src`).
- [ ] El parseo de intención del LLM está probado (mockeando la respuesta de Claude/OpenAI para asegurar que retorna el `TestScenario` correcto).
- [ ] Tests de integración que validen que una plantilla se modifica correctamente antes de enviarse al motor.

### Entregable Funcional
- [ ] El Agente es capaz de procesar el prompt: *"Crea un usuario en Contadia en el exterior con PV"* de principio a fin, retornando credenciales válidas en menos de 10 segundos.

---

**Siguiente paso:** Teniendo este "contrato" listo, el desarrollo real suele empezar por la Infraestructura y el Dominio. ¿Te gustaría que escribamos un ejemplo real de cómo se vería la plantilla JSON `pension_voluntaria.json` y la función de Python que modifica el campo de "residencia" basada en el prompt del usuario?
