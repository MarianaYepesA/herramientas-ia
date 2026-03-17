# Technical Brief

## 1. Título de la tarea

Agente de revisión de bugs del motor tribuario de Tributi, una plataforma tecnológica (fintech) colombiana diseñada para facilitar la elaboración y presentación de la declaración de renta de personas naturales.

---

## 2. Contexto


El sistema actual de motor tributario en Tributi está construido en un excel de 39,588,982 bytes que contiene las reglas de activación de las preguntas de la app, así como el calculo de los diferentes items del formulario 210. El equipo de tech se encarga de leer este excel con sus reglas específicas para llevar la lógica al front de la aplicación. La hoja de INPUTS es donde se mapean las respuestas del usuario a los diferentes objetos, ej: Bancos, fondos de pensiones, deducciones por medicina prepagada etc, de ahí los valores se llevan a otra hoja BIENES, INGRESOS, DEUDAS o GASTOS, dependiendo de a lo que corresponda el valor, despues de estandarizados los calculos en estas hojas, se llevan los valores a las hojas de Rentas_de_Trabajo, Rentas_No_Laborales, Rentas_de_Capital o de pensiones dependiendo de lo que aplique y despues a la hoja Optimización, en donde se verifica si el valor del impuesto es más bajo con dependientes del 10% o de 72 UVT.

Esto genera problemas como dificil rastreo de los bugs gracias a las diferentes hojas en las que puede estar el error, demora en realizar features nuevos ya que se debe rastrear todas las dependencias del input y posibles errores de lógica en el despliegue de preguntas de la app.

El objetivo de esta tarea es desplegar un agente que entienda completamente la lógica del excel del motor tributario para que logre revisar las cards de bugs e indicar dónde está el error y cómo solucionarlo

---

## 3. Requerimientos técnicos

### Lenguaje / Stack

| Componente | Tecnología |
|---|---|
| Lenguaje | Python 3.11+ |
| Parsing Excel | openpyxl / xlrd |
| API REST | FastAPI |
| IA / LLM | Claude API (Anthropic) |
| Logging | Python logging + Sentry |
| Testing | pytest + pytest-asyncio |
| Base de Datos (Opcional) | PostgreSQL + SQLAlchemy |

---

### 4. Arquitectura

---

### 1. DOMAIN (Objetos / Datos)
* **Contenido:** Bug, Resultado
* **Responsabilidad:** Define las reglas de negocio y las entidades principales.

### 2. APPLICATION (Lógica de flujo)
* **Contenido:** AnalizarBugUseCase
* **Responsabilidad:** Coordina cómo se mueven los datos y qué acciones se ejecutan.

### 3. INFRASTRUCTURE (Detalles técnicos)
* **Contenido:** Excel, Claude API
* **Responsabilidad:** Implementaciones externas, bases de datos y APIs.

---
### Componentes principales

#### DOMAIN (Objetos básicos)
```python
# src/domain/Bug.py
class Bug:
    id: str
    description: str  # "El impuesto es mayor al esperado"
    expected_output: float
    actual_output: float
    affected_sheets: List[str]  # ["INGRESOS", "Rentas_de_Trabajo"]

# src/domain/AnalysisResult.py
class AnalysisResult:
    bug_id: str
    root_cause: str  # "Fórmula mal en D45"
    affected_cells: List[str]  # ["D45", "D48"]
    recommendations: List[str]  # ["Cambiar la fórmula así..."]
    confidence: float  # 0.95
```

#### APPLICATION (Flujo de trabajo)
```python
# src/application/AnalyzeBugUseCase.py
class AnalyzeBugUseCase:
    """
    El flujo principal:
    1. Recibe un bug
    2. Lee el Excel
    3. Envía a Claude
    4. Retorna el análisis
    """
    
    def execute(self, bug: Bug) -> AnalysisResult:
        # 1. Obtener datos del Excel
        excel_data = self.excel_service.load()
        
        # 2. Enviar a Claude para análisis
        analysis = self.ai_service.analyze(bug, excel_data)
        
        # 3. Retornar resultado
        return analysis
```

#### INFRASTRUCTURE (Lo técnico)
```python
# src/infrastructure/ExcelService.py
class ExcelService:
    """Lee el Excel y extrae la información"""
    
    def load(self) -> Dict:
        # Abre motor.xlsx
        # Lee todas las hojas
        # Extrae fórmulas y valores
        # Retorna todo en un diccionario
        pass

# src/infrastructure/ClaudeService.py
class ClaudeService:
    """Habla con Claude API"""
    
    def analyze(self, bug: Bug, excel_data: Dict) -> AnalysisResult:
        # Prepara el mensaje para Claude
        prompt = f"""
        El usuario reporta este bug:
        {bug.description}
        
        Valor esperado: {bug.expected_output}
        Valor actual: {bug.actual_output}
        
        Aquí está la información del Excel:
        {excel_data}
        
        ¿Dónde está el error y cómo se soluciona?
        """
        
        # Llama a Claude
        response = claude_client.messages.create(
            model="claude-opus",
            messages=[{"role": "user", "content": prompt}]
        )
        
        # Parsea la respuesta y retorna AnalysisResult
        return self._parse_response(response)
```
### Input esperado

Define los datos de entrada que el sistema recibe.

### BugRequest (Input del endpoint)
```python
class BugRequest:
    id: str
        # Identificador único del bug
        # Ejemplo: "BUG-001"
    
    description: str
        # Descripción del problema reportado
        # Ejemplo: "El impuesto calculado es mayor al esperado"
    
    expected_output: float
        # Valor que debería ser el resultado
        # Ejemplo: 5000000
    
    actual_output: float
        # Valor actual (incorrecto) que se obtiene
        # Ejemplo: 7500000
    
    affected_sheets: List[str]
        # Hojas del Excel donde el usuario cree que está el error
        # Ejemplo: ["INGRESOS", "Optimización"]
        # Opcional, por defecto []
```
### Ejemplo JSON de entrada
```json
{
  "id": "BUG-001",
  "description": "El impuesto calculado es mayor al esperado cuando se aplica el factor de dependiente",
  "expected_output": 5000000,
  "actual_output": 7500000,
  "affected_sheets": ["GASTOS", "Optimización"]
}
```


### 4. Constraints (Restricciones)
### Generales

- No usar librerías externas salvo las aprobadas: openpyxl, FastAPI, pytest, anthropic
- Implementar **type hints en todo el código** (obligatorio)
- No hay excepciones: todos los métodos deben tener tipos
### Code Quality

- Seguir principios SOLID:
  - **Single Responsibility**: Una clase, una responsabilidad
  - **Open/Closed**: Abierto para extensión, cerrado para modificación
  - **Liskov Substitution**: Las subclases deben ser sustituibles
  - **Interface Segregation**: Interfaces específicas, no generales
  - **Dependency Inversion**: Depender de abstracciones, no de implementaciones

- Buenas prácticas Python:
  - PEP 8: Nombres descriptivos, espacios, longitud de líneas < 100
  - Docstrings en todas las clases y métodos públicos
  - No usar `*` o `**` sin necesidad

### Seguridad

- La API Key de Claude NO debe estar en el código
- Usar variables de entorno (.env)
- Validar todos los inputs

---

## 8. Definition of Done (DoD)

El trabajo se considera terminado cuando **TODOS** los siguientes criterios se cumplen:

### Code Quality

- [ ] El código pasa **flake8** sin errores ni warnings
```bash
  flake8 src/ --max-line-length=100 --extend-ignore=E203,W503
```

- [ ] El código está formateado con **black**
```bash
  black src/ --line-length=100 --check
```

- [ ] Todos los métodos y clases públicas tienen **type hints** completos
```python
  def analyze(self, bug: Bug) -> AnalysisResult:
```

- [ ] Todos los métodos y clases públicas tienen **docstrings**
```python
  def analyze(self, bug: Bug) -> AnalysisResult:
      """
      Analiza un bug y retorna el resultado.
      
      Args:
          bug: Objeto Bug con la información del problema
      
      Returns:
          AnalysisResult con el análisis
      
      Raises:
          ValueError: Si el bug no tiene descripción
      """
```

- [ ] No hay código muerto o importaciones no utilizadas

### Testing

- [ ] Cobertura de tests unitarios **≥ 90%**
```bash
  pytest --cov=src --cov-report=html
```

- [ ] **100% de cobertura** en clases del Domain
  - Todas las clases en `src/domain/` deben tener cobertura completa

- [ ] Al menos **80% de cobertura** en servicios
  - `ExcelService`, `ClaudeService` deben tener ≥80%

- [ ] Todos los tests pasan correctamente
```bash
  pytest -v
```

- [ ] Los tests tienen nombres descriptivos
```python
  def test_analyze_bug_returns_analysis_result_when_valid_bug():
```

- [ ] Hay fixtures para datos de prueba reutilizables
```python
  @pytest.fixture
  def sample_bug() -> Bug:
      return Bug(...)
```

### Arquitectura

- [ ] Existe separación clara entre Domain, Application e Infrastructure
- [ ] Las clases Domain no importan nada de Infrastructure
- [ ] Los servicios reciben sus dependencias inyectadas
- [ ] Existe documentación del flujo en el README

### Documentation

- [ ] Existe **README.md** con:
  - Descripción del proyecto
  - Cómo instalar (requisitos, pip install)
  - Cómo ejecutar (python main.py)
  - Cómo correr tests (pytest)
  - Cómo configurar variables de entorno (.env)

- [ ] Existe **.env.example** con variables necesarias
```
  CLAUDE_API_KEY=sk-...
  EXCEL_PATH=./motor.xlsx
```

- [ ] Existe **requirements.txt** con todas las dependencias y versiones fijas

### Git & Collaboration

- [ ] El código está en un repositorio Git
- [ ] El commit message es descriptivo
```
  git commit -m "feat: implementar ExcelService para parseo de motor.xlsx"
```

- [ ] No hay archivos temporales (.pyc, __pycache__, .env, etc.) versionados
- [ ] Existe **.gitignore** apropiado para Python

### Security & Best Practices

- [ ] No hay API Keys o secrets en el código
- [ ] Todas las dependencias están especificadas en requirements.txt
- [ ] El código valida inputs antes de procesarlos
- [ ] Existe manejo de errores básico (try/except)

### Performance

- [ ] El agente responde en menos de 30 segundos
- [ ] No hay cargadas innecesarias del Excel en cada llamada

### Optional (Nice to have)

- [ ] Existe GitHub Actions o CI pipeline que verifica:
  - flake8
  - black --check
  - pytest with coverage

- [ ] El código está documentado en Notion o similar

---
