# CLAUDE.md

Este archivo proporciona orientación a Claude Code (claude.ai/code) cuando trabaja con el código de este repositorio.

## Naturaleza del Proyecto

Este es un repositorio de **Framework PRP (Prompt de Requisito de Producto)**, no un proyecto de software tradicional. El concepto central: **"PRP = PRD + inteligencia curada de la base de código + agente/manual de ejecución"**, diseñado para permitir a los agentes de IA entregar código listo para producción en el primer intento.

## Arquitectura Principal

### Sistema Dirigido por Comandos

-   **Comandos preconfigurados de Claude Code** en `.claude/commands/`.
-   Comandos organizados por función:
    -   `PRPs/` - Flujos de trabajo de creación y ejecución de PRP.
    -   `development/` - Utilidades de desarrollo principales (prime-core, onboarding, debug).
    -   `code-quality/` - Comandos de revisión y refactorización.
    -   `rapid-development/experimental/` - Creación paralela de PRP y herramientas para hackatones.
    -   `git-operations/` - Resolución de conflictos y operaciones inteligentes de git.

### Metodología Basada en Plantillas

-   Las **Plantillas de PRP** en `PRPs/templates/` siguen un formato estructurado con bucles de validación.
-   **Enfoque Rico en Contexto**: Cada PRP debe incluir documentación completa, ejemplos y problemas conocidos.
-   **Diseño Orientado a la Validación Primero**: Cada PRP contiene puertas de validación ejecutables (sintaxis, pruebas, integración).

### Curación de Documentación de IA

-   `PRPs/ai_docs/` contiene documentación curada de Claude Code para la inyección de contexto.
-   `claude_md_files/` proporciona ejemplos de `CLAUDE.md` específicos del framework.

## Comandos de Desarrollo

### Ejecución de PRP

```bash
# Modo interactivo (recomendado para desarrollo)
uv run PRPs/scripts/prp_runner.py --prp [nombre-del-prp] --interactive

# Modo sin supervisión (para CI/CD)
uv run PRPs/scripts/prp_runner.py --prp [nombre-del-prp] --output-format json

# JSON en streaming (para monitoreo en tiempo real)
uv run PRPs/scripts/prp_runner.py --prp [nombre-del-prp] --output-format stream-json
```

### Comandos Clave de Claude

-   `/prp-base-create` - Genera PRPs completos con investigación.
-   `/prp-base-execute` - Ejecuta PRPs contra la base de código.
-   `/prp-planning-create` - Crea documentos de planificación con diagramas.
-   `/prime-core` - Prepara a Claude con el contexto del proyecto.
-   `/review-staged-unstaged` - Revisa los cambios de git usando la metodología PRP.

## Patrones Críticos de Éxito

### La Metodología PRP

1.  **El Contexto es Rey**: Incluye TODA la documentación, ejemplos y advertencias necesarias.
2.  **Bucles de Validación**: Proporciona pruebas/linting ejecutables que la IA pueda correr y corregir.
3.  **Denso en Información**: Usa palabras clave y patrones de la base de código.
4.  **Éxito Progresivo**: Empieza simple, valida y luego mejora.

### Requisitos de la Estructura de un PRP

-   **Objetivo**: Estado final específico y deseos.
-   **Por qué**: Valor de negocio e impacto para el usuario.
-   **Qué**: Comportamiento visible para el usuario y requisitos técnicos.
-   **Todo el Contexto Necesario**: URLs de documentación, ejemplos de código, problemas conocidos, patrones.
-   **Plan de Implementación**: Pseudocódigo con detalles críticos y listas de tareas.
-   **Bucle de Validación**: Comandos ejecutables para sintaxis, pruebas, integración.

### Puertas de Validación (Deben ser Ejecutables)

```bash
# Nivel 1: Sintaxis y Estilo
ruff check --fix && mypy .

# Nivel 2: Pruebas Unitarias
uv run pytest tests/ -v

# Nivel 3: Integración
uv run uvicorn main:app --reload
curl -X POST http://localhost:8000/endpoint -H "Content-Type: application/json" -d '{...}'

# Nivel 4: Despliegue
# servidores mcp, u otras formas creativas de autovalidación
```

## Anti-Patrones a Evitar

-   L No crear prompts con contexto mínimo - el contexto lo es todo - el PRP debe ser completo y autocontenido, referenciando documentación y ejemplos relevantes.
-   L No saltarse los pasos de validación - son críticos para el éxito en un solo intento - Cuanto mejor sea la IA ejecutando el bucle de validación, más probable es que tenga éxito.
-   L No ignorar el formato estructurado de PRP - ha sido probado en batalla.
-   L No crear nuevos patrones cuando las plantillas existentes funcionan.
-   L No codificar valores fijos que deberían estar en la configuración.
-   L No capturar todas las excepciones - sé específico.

## Trabajando con este Framework

### Al Crear nuevos PRPs

1.  **Proceso de Contexto**: Los nuevos PRPs deben consistir en secciones de contexto, ¡El Contexto es Rey!
2.  ...

### Al Ejecutar PRPs

1.  **Cargar PRP**: Leer y entender todo el contexto y los requisitos.
2.  **ULTRAPENSAR**: Crear un plan completo, desglosarlo en tareas, usar subagentes, herramientas por lotes, etc. revisar `prps/ai_docs/`.
3.  **Ejecutar**: Implementar siguiendo el plan.
4.  **Validar**: Ejecutar cada comando de validación, corregir los fallos.
5.  **Completar**: Asegurarse de que todos los elementos de la lista de verificación estén hechos.

### Uso de Comandos

-   Leer el directorio `.claude/commands`.
-   Acceder a través del prefijo `/` en Claude Code.
-   Los comandos son autodocumentados con marcadores de posición para los argumentos.
-   Usar comandos de creación paralela para un desarrollo rápido.
-   Aprovechar los comandos existentes de revisión y refactorización.

## Comprensión de la Estructura del Proyecto

```
PRPs-agentic-eng/
.claude/
  commands/           # Más de 28 comandos de Claude Code
  settings.local.json # Permisos de herramientas
PRPs/
  templates/          # Plantillas de PRP con validación
  scripts/           # Ejecutor de PRP y utilidades
  ai_docs/           # Documentación curada de Claude Code
   *.md               # PRPs activos y de ejemplo
 claude_md_files/        # Ejemplos de CLAUDE.md específicos del framework
 pyproject.toml         # Configuración de paquetes de Python
```

Recuerda: Este framework se trata del **éxito de la implementación en un solo paso a través de un contexto y validación completos**. Cada PRP debe contener el contexto exacto para que un agente de IA implemente con éxito código funcional en un único intento.