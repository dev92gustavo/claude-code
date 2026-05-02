# Arquitetura de Prompts Avançada

Para extrair o nível máximo de inteligência (Deep Thinking) do Claude Code ou Antigravity, a estrutura do seu prompt deve ser um "trilho" inquebrável. Abaixo estão os padrões extraídos de prompts de alta performance.

## 1. Identidade e Destino Claros
O agente precisa saber exatamente **quem ele é** e **onde o trabalho vai morar**.
*   **Agente:** Defina o papel (Ex: "Agente Antigravity Especialista Tributário").
*   **Cofre/Destino:** Defina o local exato onde o output deve ser salvo (Ex: `/C07 Reforma Tributária/`). Isso prepara a IA para formatar os links e metadados adequadamente.

## 2. A "Regra Zero" (Hard Constraints)
Coloque a restrição mais importante antes de qualquer instrução. Formate com alertas em maiúsculo (⚠️).
*   **Como fazer:** `REGRA ZERO — VALIDAÇÃO OBRIGATÓRIA ANTES DE QUALQUER AÇÃO`.
*   **O que incluir:** Lista de sites permitidos, regras de não-alucinação, proibições expressas (Ex: "REJEITE qualquer matéria de blog").

## 3. Escopo em Blocos (Módulos)
Nunca peça tudo de uma vez. Estruture em seções lógicas:
*   `MÓDULO 1 — BASE LEGAL`
*   `MÓDULO 2 — CONCEITOS BÁSICOS`
*   `MÓDULO 3 — IMPACTOS PRÁTICOS`
*   **Benefício:** Isso permite que o Agente faça *Tool Calls* separados para cada bloco, avaliando o sucesso de cada um isoladamente.

## 4. Constraint de Formato ("Formato de Saída")
Seja maníaco com o formato. Forneça o template exato que a IA deve preencher.
```markdown
### [NOME DO TÓPICO]
**Base Legal:** [Link]
**Explicação Simples:** [Máximo 5 linhas]
**Analogia:** [Comparação com o dia a dia]
**Para o Profissional:** [O que fazer na prática]
```

## 5. Instruções Finais de Comportamento (O Loop de Execução)
Na base do prompt, defina as regras operacionais (Como o robô deve "se mover"):
1.  **Sequencialidade:** "Percorra cada módulo na ordem apresentada."
2.  **Tratamento de Pendências:** "Se algo não estiver pronto no mundo real, marque como [PENDENTE] em vez de inventar."
3.  **Metadados:** "Salve com as tags X, Y, Z nas propriedades YAML."
