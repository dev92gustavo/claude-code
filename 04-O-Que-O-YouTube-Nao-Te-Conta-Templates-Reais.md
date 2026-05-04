# Claude Code: O que o YouTube não te conta (com templates reais)

> Extraído diretamente dos prompts internos vazados em `agentic-ai-prompt-research/prompts/` e do código-fonte do repositório `claw-code`. Aqui não há resumo de conceitos, mas receitas copiáveis.

---

## PARTE 1 — Como Criar uma Skill (Processo Oficial Interno)

Uma **Skill** é um fluxo de trabalho com nome próprio que você pode invocar com `/nome-da-skill`. O processo oficial de criação tem dois caminhos.

### Caminho A: Criar manualmente um arquivo `SKILL.md`

Crie o arquivo em `.claude/skills/` (dentro do seu projeto) ou em `~/.claude/skills/` (global, para todos os projetos).

**Formato exato que a Anthropic usa internamente:**

```markdown
---
name: nome-da-skill
description: Uma linha descrevendo o que ela faz e quando usar.
---

[Aqui entram as instruções detalhadas, em linguagem imperativa.]

## Pré-requisitos
- O que precisa existir antes de executar.

## Passos
1. Primeiro, faça X.
2. Então, faça Y usando o arquivo Z localizado em [caminho].
3. Confirme o resultado com o comando [comando].

## Resultado esperado
Descreva como fica ao final.

## Casos de borda
- Se acontecer A, faça B.
```

**Regras que a Anthropic aplica internamente (extraídas do código):**
- Mantenha a skill focada em **um único processo repetível**.
- Inclua caminhos de arquivo, comandos e padrões de código específicos.
- Use linguagem imperativa ("Faça", "Verifique", "Rode").
- Seja genérica o suficiente para funcionar em projetos similares, não apenas no atual.

### Caminho B: Pedir ao agente para criar (comando `/skillify`)

O comando `/skillify` já está embutido no Claude Code. Você o usa assim:

> "Quero criar uma skill para [descreva o processo]."

O agente vai te entrevistar com estas perguntas exatas:
1. O que você quer capturar como skill?
2. Quando ela deve ser ativada (o gatilho)?
3. Quais são os passos envolvidos?
4. Quais ferramentas ou comandos são usados?
5. Qual é o resultado esperado?
6. Existem restrições ou casos especiais?

Ao final, ele gera o arquivo `SKILL.md` no local correto.

---

## PARTE 2 — Como Criar um Agente (Template Oficial Interno)

Este é o prompt real que a Anthropic usa para criar novos agentes. O agente criador retorna um objeto JSON.

### O prompt que você usa para pedir a criação de um agente:

```
Crie um agente com esta finalidade: [descreva o que ele deve fazer]

Contexto do projeto: [cole aqui as convenções do seu projeto, se tiver]

Retorne um JSON com exatamente estes campos:
- identifier: nome-curto-com-hifens (2 a 4 palavras)
- whenToUse: "Use este agente quando..."
- systemPrompt: O prompt de sistema completo do agente.
```

### Regras internas que o arquiteto de agentes segue:

O `systemPrompt` gerado deve incluir:
1. **Persona de especialista** — quem o agente é e sua área de domínio.
2. **Limites de comportamento** — o que ele faz e o que recusa.
3. **Metodologia** — como ele executa a tarefa (frameworks, checklists).
4. **Verificação** — como ele confirma que o trabalho está correto antes de reportar.
5. **Saída esperada** — o formato do resultado final.

**Exemplo real de bom agente vs. ruim (do código-fonte):**

```
❌ RUIM: "Agente de análise fiscal que verifica documentos"

✅ BOM:
{
  "identifier": "fiscal-sped-reviewer",
  "whenToUse": "Use este agente quando precisar analisar arquivos SPED Fiscal para inconsistências de CFOP, verificar se os registros C100/C170 estão alinhados com as NF-e emitidas, ou validar a apuração de ICMS antes do fechamento mensal.",
  "systemPrompt": "Você é um contador especialista em EFD-ICMS/IPI com 15 anos de experiência em auditoria fiscal. Sua função é revisar arquivos SPED Fiscal identificando erros de CFOP, inconsistências entre C100 e C170, e divergências na apuração do Registro E110. Metodologia: (1) Leia o arquivo linha a linha verificando a estrutura de cada registro; (2) Cruze os CFOPs do C170 com a tabela CFOP vigente; (3) Confirme os valores do E110 com a soma dos registros C100 do período; (4) Liste todas as inconsistências com número de linha e valor esperado vs encontrado. Sempre finalize com um sumário executivo: total de registros analisados, quantidade de inconsistências, e criticidade (bloqueante/atencional)."
}
```

---

## PARTE 3 — Como Criar um Enxame de Agentes (Orquestração Paralela)

Este é o padrão mais valioso do repositório. Ele mostra como o Claude Code coordena múltiplos agentes.

### O modelo mental: Coordenador + Trabalhadores

```
Você (Usuário)
     │
     ▼
COORDENADOR (orquestra, não executa)
     │
     ├──► TRABALHADOR A (pesquisa)
     ├──► TRABALHADOR B (pesquisa)  ← rodam em paralelo
     ├──► TRABALHADOR C (pesquisa)
     │
     ▼ (sintetiza os resultados)
     │
     ├──► TRABALHADOR D (implementa com base na síntese)
     │
     ▼
AGENTE VERIFICADOR (tenta quebrar o que foi feito)
```

### Como pedir a criação de um enxame (fórmula direta):

```
Quero orquestrar múltiplos agentes para [objetivo geral].

Fase 1 - Pesquisa paralela:
- Agente 1: [tarefa de pesquisa específica]
- Agente 2: [outra tarefa de pesquisa específica]
- Agente 3: [outra tarefa de pesquisa específica]

Fase 2 - Você (coordenador) sintetiza os achados e define o plano.

Fase 3 - Implementação:
- Um agente implementa com base no plano sintetizado.

Fase 4 - Verificação:
- Um agente verificador testa o resultado de forma adversarial.
```

### As três regras de ouro do Coordenador (extraídas do prompt interno):

**Regra 1: Paralelismo é o superpoder.**
> *"Parallelism is your superpower. Workers are async. Launch independent workers concurrently whenever possible."*

Tarefas de leitura/pesquisa: rode em paralelo, sem restrição.
Tarefas de escrita: uma por vez, por conjunto de arquivos.

**Regra 2: Sintetize — nunca delegue a compreensão.**

```
❌ ANTI-PADRÃO (o código-fonte chama de "lazy delegation"):
"Com base nos seus achados, corrija o problema."

✅ CORRETO (sintetizado):
"Corrija a inconsistência no registro C170 linha 482. O CFOP 5.102
está sendo usado para uma operação interestadual que deveria usar
6.102. Atualize o campo específico e confirme que a soma do E110
se mantém."
```

**Regra 3: Continuar vs. Disparar novo agente.**

| Situação | O que fazer |
|---|---|
| O pesquisador já leu exatamente os arquivos que precisam ser editados | **Continuar** o mesmo agente com o plano sintetizado |
| A pesquisa foi ampla mas a implementação é cirúrgica | **Novo agente**, contexto limpo |
| Corrigindo uma falha anterior | **Continuar** — o agente tem o histórico do erro |
| Verificação de código que outro agente escreveu | **Novo agente** — olhos frescos |

---

## PARTE 4 — O Agente Verificador (Anti-aprovação Automática)

Este é um dos prompts mais ricos do repositório. O verificador é explicitamente proibido de aprovar sem rodar testes.

### Como pedir uma verificação real:

```
Verifique esta implementação de forma adversarial:

- Tarefa original: [descreva o que foi pedido]
- Arquivos alterados: [liste os arquivos]
- Abordagem usada: [descreva brevemente]

Regras:
1. Não modifique nenhum arquivo do projeto.
2. Rode testes, não apenas leia o código.
3. Tente quebrar a implementação com casos de borda.
4. Finalize com VERDICT: PASS, FAIL ou PARTIAL.
```

### As armadilhas que o verificador é treinado a evitar (do prompt real):

O prompt interno lista literalmente as "racionalizações" que um agente ruim usa:

```
❌ "O código parece correto com base na minha leitura." → Leitura não é verificação. Execute.
❌ "Os testes do implementador já passam." → O implementador também é uma IA. Verifique independentemente.
❌ "Provavelmente está certo." → "Provavelmente" não é verificado. Execute.
❌ "Não tenho um navegador para testar." → Você verificou se as ferramentas de automação estão disponíveis?
```

### Formato obrigatório de cada verificação (do código-fonte):

```markdown
### Verificação: [o que está sendo verificado]
**Comando executado:** [o comando exato]
**Saída observada:** [a saída real do terminal — não parafraseada]
**Resultado: PASS** (ou FAIL — com Esperado vs. Encontrado)

VERDICT: PASS
```

---

## PARTE 5 — Memória Persistente (Como Ensinar o Agente a Aprender)

### Hierarquia oficial de memória (do prompt `/remember`):

| Arquivo | Localização | Propósito |
|---|---|---|
| `CLAUDE.md` | Raiz do projeto | Regras compartilhadas com o time, vai para o git |
| `CLAUDE.local.md` | Raiz do projeto | Regras privadas do seu ambiente, no `.gitignore` |
| `~/.claude/CLAUDE.md` | Pasta home | Preferências globais, valem para todos os projetos |

### Como usar o comando `/remember`:

O agente vai:
1. Ler as memórias capturadas automaticamente em `MEMORY.md`.
2. Para cada memória, decidir: vale a pena guardar permanentemente?
3. Sugerir se deve ir para `CLAUDE.md` (compartilhado) ou `CLAUDE.local.md` (privado).
4. Aplicar após sua confirmação.

### Como escrever uma boa instrução para o `CLAUDE.md`:

```markdown
# CLAUDE.md — Regras do Projeto ASERCO

## Identidade
- Idioma: Português (PT-BR). Nunca use inglês nas respostas ao usuário.
- Tom: Técnico e direto. Sem preâmbulo, sem elogios, sem bajulação.
- Zero travessões. Use vírgulas, ponto e vírgula ou parênteses.

## Convenções do Projeto
- Toda nota fiscal deve ter os campos: CNPJ, CFOP, CST, valor base, alíquota.
- CFOPs interestaduais iniciam com 6. Se o código iniciar com 5, é operação interna.
- Nunca apague conteúdo existente em notas; apenas adicione ou corrija.

## Ferramentas Proibidas
- Não use a ferramenta Bash para leitura de arquivos. Use sempre a ferramenta Read.
- Não crie arquivos temporários fora da pasta `.claude/scratch/`.
```

---

## PARTE 6 — Modo Autônomo (Agente que Trabalha Sozinho)

O modo proativo (`PROACTIVE`) faz o agente trabalhar em background sem intervenção contínua.

### Como ativar e instruir o modo autônomo:

```
Quero que você trabalhe autonomamente nesta tarefa enquanto estou ausente:

[Descrição da tarefa]

Regras de autonomia:
- Leia arquivos, pesquise, rode testes e explore sem pedir permissão.
- Faça alterações e confirme quando chegar a um ponto de parada natural.
- Se o terminal não estiver focado (estou ausente): tome decisões e avance.
- Se o terminal estiver focado (estou presente): me consulte antes de mudanças grandes.
- Quando não houver nada útil para fazer, aguarde — não me mande mensagens de status.
- Nunca narre o que está prestes a fazer. Apenas faça.
```

### O que o agente autônomo faz vs. não faz (do prompt interno):

| Faz sem pedir | Exige confirmação |
|---|---|
| Ler e pesquisar arquivos | Deletar arquivos |
| Rodar testes e linters | Fazer push para repositórios remotos |
| Explorar o projeto | Modificar pipelines de CI/CD |
| Fazer commits locais | Enviar mensagens externas (email, Slack) |
| Escolher entre duas abordagens razoáveis | Ações com alto impacto e difíceis de reverter |

---

## PARTE 7 — O Sistema de Permissões por Wildcard

A maneira correta de configurar o que o agente pode e não pode fazer.

### Formato real das regras (do código-fonte):

```json
{
  "autoMode": {
    "allow": [
      "Read(*)",
      "Bash(git status)",
      "Bash(git log *)",
      "Bash(npm test)",
      "Glob(*)"
    ],
    "soft_deny": [
      "Bash(rm *)",
      "Bash(git push *)",
      "Bash(git reset --hard)"
    ],
    "environment": "Ambiente de desenvolvimento local. Projeto de contabilidade fiscal. Nunca enviar dados para APIs externas sem confirmação explícita."
  }
}
```

**Sintaxe do wildcard:**
- `Read(*)` — permite leitura de qualquer arquivo.
- `Bash(git *)` — permite qualquer comando git.
- `Bash(npm test)` — permite apenas `npm test`, nada mais.
- `FileEdit(/src/*)` — permite editar apenas arquivos dentro de `/src/`.

---

## Receita Completa: Do Zero ao Agente Funcional

```
1. Crie o CLAUDE.md do projeto com suas regras e convenções.

2. Peça a criação do agente:
   "Crie um agente chamado [nome] que [finalidade].
    Contexto: [cole seu CLAUDE.md aqui].
    Retorne o JSON com identifier, whenToUse e systemPrompt."

3. Salve o JSON gerado em .claude/agents/[identifier].json

4. Para tarefas complexas, ative o enxame:
   "Preciso de [objetivo]. Divida em pesquisa paralela primeiro,
    sintetize os achados, depois implemente e verifique adversarialmente."

5. Após a execução, rode /remember para organizar o que foi aprendido.

6. Atualize o CLAUDE.md com as novas regras descobertas.
```
