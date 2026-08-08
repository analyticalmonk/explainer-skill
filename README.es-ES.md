

# Explain this 🔌💡

Un paquete de Agent Skills para crear explicadores interactivos al estilo distill: páginas `index.html` individuales y autosuficientes con un diseño fijo de dos columnas, figuras de Canvas creadas a mano y prosa conversacional, similares a los explicadores de [distill.pub](https://distill.pub). Cero dependencias, sin paso de compilación.

## Las habilidades (skills)

- **creating-explainers** - el núcleo. Convierte un artículo, publicación de blog, transcripción o informe de investigación en un explicador, o investiga un tema desde cero, o ambas cosas. Controla la plantilla, las figuras, el tono y el flujo de trabajo por etapas.
- **explaining-codebases** - explica un repositorio o conjunto de archivos fuente: una visión general de incorporación sobre cómo está estructurado un proyecto, o un análisis profundo sobre cómo funciona un mecanismo. Mismo formato de salida, con navegación de código y figuras específicas para código.
- **fact-checking-explainers** - un filtro por el que pasan las otras dos antes de la entrega. Cada afirmación verificable debe rastrearse hasta una fuente (o, para código, la implementación real), corregirse o eliminarse.

Cada habilidad se activa automáticamente cuando describes el objetivo correspondiente. También puedes invocar la verificación de hechos de forma independiente.

## Ejemplos

Dos explicadores interactivos creados con este conjunto de habilidades. Ábrelos y experimenta con las figuras: avanzan por pasos, se arrastran y se alternan directamente en el navegador.

**[Superpowers: The Anatomy of an Agent Skill](https://www.akashtandon.in/interactive-explainers/superpowers/)**

[![Superpowers explainer](docs/images/superpowers.png)](https://www.akashtandon.in/interactive-explainers/superpowers/)

**[DSPy: Programming - Not Prompting - Language Models](https://www.akashtandon.in/interactive-explainers/dspy/)**

[![DSPy explainer](docs/images/dspy.png)](https://www.akashtandon.in/interactive-explainers/dspy/)

## Requisitos

Sin claves API, sin herramientas de compilación. Las habilidades utilizan las propias herramientas de archivos y web de tu agente: la investigación inicial necesita búsqueda/obtención web disponibles, y la salida es un solo archivo `index.html` autosuficiente que abres en un navegador.

## Instalación

### Claude Code

explain-this es un complemento de Claude Code, por lo que se instala a través del mercado de complementos. Agregar el mercado e instalar el complemento incorpora las tres habilidades de una sola vez:

```
/plugin marketplace add analyticalmonk/explain-this
/plugin install explain-this@explain-this
```

El primer comando apunta Claude Code a este repositorio en GitHub; el segundo instala el complemento `explain-this` desde él. Las habilidades se instalan en `~/.claude/skills/` y se activan automáticamente.

¿Prefieres un clon local o el repositorio aún no está publicado? Agrega el mercado desde una ruta local en su lugar:

```
git clone https://github.com/analyticalmonk/explain-this.git
/plugin marketplace add ./explain-this
/plugin install explain-this@explain-this
```

**Actualización:** Claude Code aún no actualiza los complementos automáticamente. Para obtener una versión más reciente, actualiza el mercado e reinstala:

```
/plugin marketplace update explain-this
/plugin install explain-this@explain-this
```

### OpenAI Codex

Este repositorio también es un paquete de complemento para Codex. El manifiesto de Codex se encuentra en `.codex-plugin/plugin.json` y apunta a los mismos tres directorios de habilidades bajo `skills/`.

Agrega el mercado e instala el complemento desde la CLI de Codex:

```bash
codex plugin marketplace add analyticalmonk/explain-this
codex plugin add explain-this@explain-this
```

El primer comando apunta Codex a este repositorio en GitHub; el segundo instala el complemento `explain-this` desde ese mercado. Inicia un nuevo hilo después de instalar para que Codex detecte las habilidades incluidas. También puedes explorar los complementos instalados y disponibles desde la TUI de Codex con `/plugins`.

¿Prefieres un clon local o el repositorio aún no está publicado? Clona o crea un enlace simbólico de este repositorio a `~/plugins/explain-this`, luego apunta una entrada del mercado de Codex a esa fuente del complemento. Una entrada de mercado personal mínima se ve así:

```json
{
  "name": "personal",
  "interface": {
    "displayName": "Personal"
  },
  "plugins": [
    {
      "name": "explain-this",
      "source": {
        "source": "local",
        "path": "./plugins/explain-this"
      },
      "policy": {
        "installation": "AVAILABLE",
        "authentication": "ON_INSTALL"
      },
      "category": "Education"
    }
  ]
}
```

Colócalo en `~/.agents/plugins/marketplace.json`, reinicia Codex y luego abre `/plugins` en la CLI de Codex o la vista de Complementos en la aplicación de Codex e instala **Explain This**. Codex resuelve `./plugins/explain-this` relativo a tu directorio de inicio para el mercado personal.

Si solo quieres las habilidades sin la instalación del complemento, copia o crea enlaces simbólicos de los directorios a una ubicación de habilidades de Codex como `$HOME/.agents/skills/`:

```bash
mkdir -p "$HOME/.agents/skills"
cp -R skills/creating-explainers \
      skills/explaining-codebases \
      skills/fact-checking-explainers \
      "$HOME/.agents/skills/"
```

Codex luego puede invocarlas explícitamente con `$creating-explainers`, `$explaining-codebases` o `$fact-checking-explainers`.

### Cualquier agente, a través de la CLI skills.sh

El repositorio sigue el formato abierto de [Agent Skills](https://agentskills.io) (`skills/<name>/SKILL.md`), por lo que la CLI [skills.sh](https://www.skills.sh/) puede instalar las habilidades en Claude Code, Codex, Cursor, OpenCode, OpenClaw, Gemini CLI, GitHub Copilot y más de 60 agentes más con un solo comando:

```bash
npx skills add analyticalmonk/explain-this --all
```

La CLI encuentra las tres habilidades y pregunta para qué agentes instalarlas. Variantes útiles:

```bash
npx skills add analyticalmonk/explain-this --list                       # vista previa sin instalar
npx skills add analyticalmonk/explain-this --skill creating-explainers # instalar una habilidad
npx skills add analyticalmonk/explain-this --all -g                     # global en lugar de por proyecto
npx skills update                                                       # actualizar habilidades instaladas más tarde
```

### OpenClaw

Las tres habilidades están publicadas en [ClawHub](https://docs.openclaw.ai/clawhub), el registro de habilidades de OpenClaw, por lo que la instalación más sencilla es desde el directorio de tu espacio de trabajo de OpenClaw:

```bash
npx clawhub install creating-explainers
npx clawhub install explaining-codebases
npx clawhub install fact-checking-explainers
```

Cada habilidad se instala en la carpeta `skills/` del espacio de trabajo. La CLI de clawhub requiere una versión reciente de Node (declara Node 22+).

OpenClaw también lee el formato crudo de Agent Skills, por lo que dos métodos más funcionan: instala a través de skills.sh como se indica arriba (selecciona OpenClaw cuando la CLI pregunte qué agentes apuntar), o copia los directorios de habilidades en un directorio que OpenClaw escanee: `~/.openclaw/skills/` para todos tus agentes, o la carpeta `skills/` de tu espacio de trabajo para un solo agente:

```bash
git clone https://github.com/analyticalmonk/explain-this.git
cp -R explain-this/skills/* ~/.openclaw/skills/
```

OpenClaw captura las habilidades al inicio de la sesión, por lo que inicia una nueva sesión después de instalarlas. Las habilidades no necesitan configuración adicional: sin claves API, binarios ni variables de entorno, por lo que no hay que configurar restricción de `metadata.openclaw`.

### Otros agentes que leen Agent Skills

Para una herramienta que lea habilidades `SKILL.md` pero no esté cubierta por skills.sh, la instalación es manual: clona el repositorio y apunta tu agente a los tres directorios de habilidades, o cópialos en el directorio desde el cual tu agente carga habilidades.

```bash
git clone https://github.com/analyticalmonk/explain-this.git
#   skills/creating-explainers/
#   skills/explaining-codebases/
#   skills/fact-checking-explainers/
```

Una advertencia importante: estas habilidades fueron escritas y probadas en Claude Code y Codex. Algunas referencias aún mencionan nombres de herramientas de Claude Code (Read, Edit, Bash, WebSearch / WebFetch), por lo que pueden necesitar una ligera adaptación en otras herramientas compatibles con Agent-Skills.

### No compatible

No hay instalación para entornos sin un mecanismo de habilidades: las aplicaciones web de chat (claude.ai, ChatGPT y Gemini en el navegador) y las APIs de modelos nativos. No pueden cargar habilidades `SKILL.md` en absoluto. La única forma de usar el flujo de trabajo allí es pegar manualmente el contenido de una habilidad en la conversación, lo cual no está realmente soportado y pierde las referencias de carga diferida que mantienen las habilidades ligeras.

## Uso

Tu agente selecciona automáticamente la habilidad correcta cuando dices cosas como:

- "Haz un explicador interactivo sobre RLHF" (entrada de investigación)
- "Convierte este artículo en un explicador al estilo distill" (entrada de archivos)
- "Explica cómo funciona el programador de este repositorio, como una guía interactiva" (código base)

Sin importar la ruta que elijas, el explicador se verifica antes de entregarse: cada afirmación se rastrea hasta su fuente o el código real, y cualquier cosa sin soporte se corrige o elimina.

## Qué hay en este repositorio

```
.codex-plugin/
  plugin.json                       # Manifiesto del complemento de Codex
.claude-plugin/
  plugin.json                       # Manifiesto del complemento de Claude Code
  marketplace.json                  # Entrada del mercado para /plugin marketplace add
skills/
  creating-explainers/
    SKILL.md
    agents/openai.yaml              # Metadatos de la IU de Codex
    assets/article-template.html    # Esqueleto HTML completo, copia y rellena {{PLACEHOLDERS}}
    references/                      # Entrada (archivos / investigación), figuras, tono, plantilla, paletas
  explaining-codebases/
    SKILL.md
    agents/openai.yaml
    references/                      # Entrada de código, arquetipos de figuras específicas para código
  fact-checking-explainers/
    SKILL.md
    agents/openai.yaml
    references/verification-report-format.md
evals/
  evals.json                        # 5 prompts de referencia contra los cuales se desarrollan las habilidades
```

Las referencias se cargan de forma diferida: tu agente las lee solo cuando son relevantes, por lo que las habilidades no consumen contexto de antemano.

## Licencia

MIT - ver [LICENSE](LICENSE).
