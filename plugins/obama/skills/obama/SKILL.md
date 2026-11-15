---
name: obama
description: Classifica uma demanda do fluxo Spec Kit e escolhe o modelo (Haiku, Sonnet, Opus ou Fable) para executá-la em agente de background. Use antes de delegar fase ou tarefa do speckit.
---

# Obama — roteador de modelos para o Spec Kit

Obama lê uma demanda, classifica e decide em qual modelo ela deve rodar. Depois dispara um agente em background com esse modelo. O objetivo é **gastar o mínimo de modelo que entrega o resultado certo de primeira**: subdimensionar sai caro em retrabalho, e superdimensionar sai caro em custo e latência.

## Quando acionar

- Antes de delegar qualquer fase do Spec Kit (`/speckit.constitution`, `specify`, `clarify`, `plan`, `tasks`, `analyze`, `checklist`, `implement`) a um agente.
- Para cada tarefa de `tasks.md` durante o `/speckit.implement`, principalmente as marcadas com `[P]` (paralelizáveis).
- Quando o usuário chamar `/obama <demanda>`.

## Os quatro níveis

| Nível | Alias no Agent | Para quê | Sinal típico |
|---|---|---|---|
| N1 | `haiku` | Mecânico, repetitivo, escopo fechado | Renomear, formatar, gerar boilerplate, CRUD a partir de um padrão que já existe, docstrings, checklist simples |
| N2 | `sonnet` | **Padrão** de implementação | Feature com spec clara, testes unitários, endpoint novo seguindo a arquitetura, refactor local |
| N3 | `opus` | Raciocínio pesado, ambiguidade, impacto transversal | Arquitetura, modelo de dados, plano técnico, debugging sem causa óbvia, mudança em vários módulos, análise de consistência spec↔plan↔tasks |
| N4 | `fable` | Fronteira: problema longo, aberto e de alto valor | Desenho de sistema novo do zero com muitas restrições em conflito, problema em que N3 já falhou, pesquisa técnica profunda |

Regras do N4:
- Só use quando a demanda pontuar ≥ 22 **ou** quando o N3 já tiver falhado nessa mesma demanda.
- Fable tem salvaguardas extras em cibersegurança e biologia. Por isso, demandas de segurança ofensiva/defensiva, pentest, análise de exploit e auditoria de vulnerabilidade vão para **N3 (`opus`)**, não para Fable.
- Se o alias `fable` não estiver disponível no ambiente, caia para `opus` e registre isso no log.

## Passo 1 — Extrair a demanda

Monte um resumo de 1 a 3 linhas com:
- **Fase Spec Kit** (constitution, specify, clarify, plan, tasks, analyze, checklist, implement)
- **Artefatos tocados** (arquivos, módulos, tabelas). Use `spec.md`, `plan.md`, `tasks.md` e `data-model.md` da feature atual em `specs/<feature>/`.
- **Critério de pronto** (o que precisa estar verdadeiro no fim)

Se não der para escrever o critério de pronto, a demanda é ambígua: some +3 em Ambiguidade.

## Passo 2 — Pontuar (0 a 4 em cada eixo)

| Eixo | 0 | 2 | 4 |
|---|---|---|---|
| **Complexidade de raciocínio** | Transformação direta | Lógica com alguns casos de borda | Algoritmo, concorrência, trade-offs de design |
| **Ambiguidade** | Spec fecha tudo | Algumas decisões locais | Requisito aberto, precisa inferir intenção |
| **Abrangência** | 1 arquivo | 2 a 5 arquivos do mesmo módulo | Vários módulos/camadas, contratos públicos |
| **Risco / reversibilidade** | Trivial de desfazer | Afeta comportamento testado | Migração de dados, auth, pagamentos, contrato de API externa |
| **Contexto necessário** | Só a tarefa | Spec + 1 a 2 arquivos | Spec, plan, data-model e boa parte do código |
| **Novidade** | Existe padrão no repo para copiar | Padrão parecido existe | Nada parecido no repo |
| **Histórico** | Primeira tentativa | 1 falha anterior | 2 ou mais falhas anteriores |

Total possível: 0 a 28.

## Passo 3 — Mapear para o nível

| Pontuação | Nível |
|---|---|
| 0–6 | N1 `haiku` |
| 7–14 | N2 `sonnet` |
| 15–21 | N3 `opus` |
| 22–28 | N4 `fable` (respeitando as regras do N4) |

Pisos obrigatórios por fase (o nível nunca fica abaixo do piso):

| Fase | Piso | Comentário |
|---|---|---|
| constitution | N3 | Define princípios que governam todo o resto |
| specify | N2 | Sobe para N3 se a ideia for vaga ou envolver vários atores |
| clarify | N2 | |
| plan | N3 | Arquitetura e escolhas técnicas |
| tasks | N2 | |
| analyze | N3 | Consistência entre artefatos exige visão ampla |
| checklist | N1 | |
| implement | por tarefa | Use só a pontuação |

Ajustes adicionais:
- Risco = 4 → piso N3, seja qual for o total.
- Tarefa com `[P]` e pontuação ≤ 6 → mantenha N1 para paralelizar barato.

## Passo 4 — Overrides

O que for explícito vence a classificação:
1. Tag na própria tarefa em `tasks.md`: `[model:haiku|sonnet|opus|fable]`
2. Pedido do usuário na conversa ("roda isso no opus")
3. Arquivo `.specify/obama.config.json`, se existir (ver abaixo)

Se o override contrariar um piso de risco, obedeça, mas escreva `"override_contra_piso": true` no log.

## Passo 5 — Registrar a decisão

Antes de disparar, emita este bloco e grave uma linha em `.specify/obama/routing-log.jsonl`:

```json
{
  "ts": "<ISO-8601>",
  "feature": "<pasta em specs/>",
  "fase": "implement",
  "tarefa": "T014",
  "resumo": "Endpoint POST /pedidos com validação e testes",
  "scores": {"raciocinio": 2, "ambiguidade": 1, "abrangencia": 2, "risco": 2, "contexto": 2, "novidade": 1, "historico": 0},
  "total": 10,
  "nivel": "N2",
  "modelo": "sonnet",
  "motivo": "Feature com spec clara seguindo padrão de outros endpoints",
  "override": null,
  "tentativa": 1
}
```

`motivo` precisa ser uma frase concreta, nunca "complexidade média".

## Passo 6 — Disparar o agente em background

Chame a ferramenta Agent com:
- `model`: o alias escolhido
- `run_in_background`: `true`
- `description`: `obama:<fase>:<tarefa>`, curto
- `prompt`: autossuficiente, porque o agente começa sem contexto. Inclua:
  - caminho da feature e dos artefatos que ele deve ler (`spec.md`, `plan.md`, `tasks.md`)
  - a tarefa exata e o critério de pronto
  - restrições da constitution (`.specify/memory/constitution.md`)
  - o que entregar de volta: arquivos alterados, comandos de teste rodados e resultado, pendências
  - instrução para marcar a tarefa como `[X]` em `tasks.md` só se o critério de pronto foi atendido

Tarefas `[P]` independentes podem ser disparadas juntas. Tarefas sem `[P]` respeitam a ordem de `tasks.md`.

## Passo 7 — Quando o agente terminar

1. Confira o critério de pronto: rode os testes citados e leia o diff.
2. **Passou** → registre `"resultado": "ok"` no log.
3. **Falhou** → suba **um** nível (Histórico +2), registre `"resultado": "escalado"` e dispare de novo com o erro anterior no prompt.
4. Limite de 2 escalonamentos por tarefa. Depois disso, pare e devolva para o usuário com o diagnóstico.
5. Nunca rebaixe o nível de uma tarefa que já falhou.

## Configuração opcional — `.specify/obama.config.json`

```json
{
  "default": "sonnet",
  "modelos_disponiveis": ["haiku", "sonnet", "opus", "fable"],
  "limiares": {"N1": 6, "N2": 14, "N3": 21},
  "pisos_fase": {"plan": "opus", "analyze": "opus"},
  "max_escalonamentos": 2,
  "forcar": {"migrations/**": "opus", "docs/**": "haiku"}
}
```

`forcar` usa glob nos arquivos tocados: se qualquer arquivo bater, aplica o modelo como piso.

## Calibração

A cada ~30 decisões, leia `routing-log.jsonl` e procure:
- **Escalonamentos frequentes no mesmo tipo de tarefa** → o limiar está alto demais. Baixe ou crie uma regra em `forcar`.
- **Muito N3/N4 passando de primeira em tarefas com nota 15–16** → teste rebaixar a faixa para N2.

Proponha os ajustes ao usuário. Não altere a config sozinho.

## Exemplos rápidos

- "Adicionar campo `telefone` no DTO e no formulário seguindo `email`" → total 4 → **haiku**
- "Implementar T014: endpoint de pedidos com validação" → total 10 → **sonnet**
- "`/speckit.plan` para módulo de faturamento multiempresa" → piso N3 → **opus**
- "Migração que divide a tabela `clientes` em duas" → risco 4 → **opus**
- "Auditar RLS do Supabase" → segurança → **opus** (nunca fable)
- "Motor de otimização de escala com restrições conflitantes; opus falhou 2x" → **fable**
