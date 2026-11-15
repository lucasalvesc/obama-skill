<p align="center">
  <img src="assets/obama-futebol.gif" alt="Animação pixelada: o autor e o robô do obama trocando passes com uma bola de futebol" width="504">
</p>

# obama-skill

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

Plugin do Claude Code com a skill **obama**: um roteador de modelos para o fluxo [Spec Kit](https://github.com/github/spec-kit).

A skill lê cada demanda (fase do speckit ou tarefa do `tasks.md`), dá uma nota em 7 eixos e escolhe o modelo do agente em background:

| Nota | Modelo |
|---|---|
| 0–6 | `haiku` |
| 7–14 | `sonnet` |
| 15–21 | `opus` |
| 22–28 | `fable` |

Ela também aplica pisos por fase (`plan`, `analyze` e `constitution` rodam no mínimo em `opus`), aceita overrides e escala o modelo quando uma tarefa falha. Cada decisão fica registrada em `.specify/obama/routing-log.jsonl`.

## Instalação

Dentro do Claude Code, no projeto:

```
/plugin marketplace add lucasalvesc/obama-skill
/plugin install obama@obama-skill
```

### Instalação automática por projeto

Para que todo mundo que clonar o projeto receba a skill, commite isto em `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "obama-skill": {
      "source": { "source": "github", "repo": "lucasalvesc/obama-skill" }
    }
  },
  "enabledPlugins": {
    "obama@obama-skill": true
  }
}
```

### Atualizar

```
/plugin marketplace update obama-skill
```

## Uso

- Automático: antes de delegar uma fase do speckit ou uma tarefa do `/speckit.implement`.
- Manual: `/obama <descrição da demanda>`.

### Overrides

- Na tarefa em `tasks.md`: `[model:opus]`
- Por projeto: `.specify/obama.config.json` (veja o exemplo na [SKILL.md](plugins/obama/skills/obama/SKILL.md))

## Estrutura

```
.claude-plugin/marketplace.json
plugins/obama/.claude-plugin/plugin.json
plugins/obama/skills/obama/SKILL.md
```

## Contribuindo

Issues e pull requests são bem-vindos. Para propor mudanças nas regras de roteamento (limiares, pisos por fase, eixos de pontuação), abra uma issue com exemplos de demandas e o modelo que você esperava.

## Licença

[MIT](LICENSE) © 2026 Lucas Alves Castro
