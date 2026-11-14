# Skiils-AGENT-SUPER

Skill do Claude Code que monta um **conselho de 19 agentes**: um **Maestro** caro (Opus 5.5, esforço alto) pensa no que você pediu e distribui o trabalho para especialistas baratos (Haiku e Sonnet). No fim, um **juiz** dá o veredito.

## Como usar

```
/conselho-de-agentes quero criar um app de ...
/conselho-de-agentes --rapido quero criar um app de ...
```

- Sem flag: conselho completo (até 19 agentes; o Maestro pula quem não se aplica e avisa).
- `--rapido`: só 7 agentes (requisitos, projetos-similares, investimento, economizador-tokens, estrategista-skills, critico, juiz).
- A skill só roda quando você chama (`disable-model-invocation: true`), porque uma rodada completa custa. Para o Claude acioná-la sozinho, apague essa linha do `SKILL.md`.
- Se os agentes não aparecerem, rode `/agents` ou reinicie a sessão.

## Como funciona

```
Você ─► Maestro (Opus 5.5, esforço alto): interpreta o pedido e grava o brief
          ├─ Onda 1 · 13 agentes em paralelo (pesquisa e análise independentes)
          ├─ Onda 2 · 5 agentes que leem a onda 1 (economista, riscos, pré-mortem, verificador, crítico)
          ├─ Onda 3 · juiz: placar 0–10 e veredito
          └─ Maestro: decide e entrega o resumo final
```

Para gastar pouco, os agentes **não passam texto pelo Maestro**: cada um grava um relatório curto em `.conselho/<data>-<slug>/` e devolve só 2 linhas (caminho + resumo). O Maestro lê por inteiro apenas crítico, verificador e juiz.

## Elenco

| Agente | Modelo | Onda | Função |
|---|---|---|---|
| requisitos | Haiku | 1 | Escopo, requisitos, premissas, MVP |
| revisor-ideia | Sonnet | 1 | Revisa a ideia: clareza, coerência, diferencial |
| publico | Haiku | 1 | Personas, dor, alternativas, canais |
| mercado | Haiku | 1 | Demanda, tendências, preços |
| projetos-similares | Haiku | 1 | Concorrentes, alternativas, open source |
| casos-de-sucesso | Haiku | 1 | Ideias parecidas que deram certo e por quê |
| casos-de-fracasso | Haiku | 1 | Ideias parecidas que falharam e lições |
| investimento | Sonnet | 1 | Quanto custa, quanto gastar e **quando** gastar |
| estimador | Haiku | 1 | Esforço, prazo, marcos |
| arquiteto | Sonnet | 1 | Viabilidade técnica, stack, construir × comprar |
| estrategista-skills | Haiku | 1 | Skills e ferramentas para este e outros projetos |
| economizador-tokens | Haiku | 1 | Custo de IA, roteamento de modelos |
| compliance | Sonnet | 1 | Legal, privacidade, segurança, ética |
| economista | Sonnet | 2 | Economia unitária, cenários, break-even |
| riscos | Haiku | 2 | Matriz de riscos |
| premortem | Sonnet | 2 | "Fracassou: por quê?" |
| verificador | Sonnet | 2 | Checa em fontes os fatos decisivos |
| critico | Sonnet | 2 | Advogado do diabo |
| juiz | Sonnet | 3 | Placar e veredito |

## Arquivos

```
.claude/skills/conselho-de-agentes/SKILL.md   playbook do Maestro
.claude/agents/*.md                           os 19 especialistas (modelo, ferramentas e instruções de cada um)
.claude/settings.json                         libera só Write em .conselho/** e WebSearch
.conselho/                                    relatórios de cada rodada (ignorado pelo git)
```

## Ajustes comuns

- **Trocar modelos:** edite a linha `model:` do agente. Para usar apelidos que acompanham a versão mais nova: `sed -i 's/claude-haiku-5-5/haiku/; s/claude-sonnet-5-5/sonnet/' .claude/agents/*.md`
- **Subir o nível de um agente** (ex.: `critico` em Opus): troque o `model:` dele.
- **Ver o gasto:** `/cost` depois da rodada.
- **Resultados são hipóteses:** números de agentes baratos podem estar errados. O verificador checa os principais, e o Maestro marca no resumo o que é ✔ verificado e o que é ≈ estimativa.
