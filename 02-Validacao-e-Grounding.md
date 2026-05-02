# Validação e Grounding (Anti-Alucinação)

A maior fraqueza dos LLMs é a alucinação complacente (inventar respostas para agradar). O prompt analisado utiliza técnicas de engenharia robustas para "ancorar" (grounding) a IA na realidade. 

Aqui está como replicar isso em qualquer projeto:

## 1. Classificação de Fontes (Tier List)
Obrigue o Agente a usar a ferramenta de *Web Search* de forma hierárquica.
*   **Fontes Primárias (Obrigatórias):** Domínios oficiais (`.gov.br`, `.leg.br`). A IA é instruída a só confiar nessas fontes para a "verdade".
*   **Fontes Secundárias (Cruzamento):** Jornais oficiais, diários oficiais. Usados para confirmar datas e publicações.
*   **Fontes Proibidas:** Blogs, escritórios de advocacia, notícias. Elas podem ser lidas para entender o contexto, mas **NUNCA** usadas como fonte da verdade.

## 2. Resolução de Conflitos (Hierarquia de Leis)
A IA vai encontrar informações conflitantes durante a pesquisa (especialmente em temas em transição). Programe o raciocínio dela para esses momentos:
*   **Regra de Ouro Injetada:** *"Ao encontrar divergência entre fontes, priorize: texto da lei > decreto > instrução normativa > nota técnica."*
*   **Aplicação em outros projetos:** Se for código: *"Documentação oficial > Código fonte > StackOverflow"*.

## 3. Rastreabilidade Exigida
Para garantir que a IA realmente usou uma Tool de busca e não sua memória de treinamento (que pode estar desatualizada):
*   Exija o número da norma/versão.
*   Exija o Link Direto.
*   Exija a Data de Consulta.

## 4. Fallback Passivo (Tratamento de "Not Found")
A IA tem pavor de dizer "não encontrei" e acaba inventando. Dê a ela uma "saída honrosa":
*   *"Se um regulamento estiver aguardando publicação, não adivinhe o conteúdo. Registre como 'Pendente de Regulamentação' e marque a tarefa para revisão futura."*

## 5. Self-Reflection (Crítica Interna)
Embutir um passo de reflexão força a IA a reavaliar sua própria geração usando uma "persona" diferente (ex: O Crítico).
*   *"Ao final de cada módulo, pergunte internamente: Existe algo sobre este tema que um profissional da área poderia ter esquecido? Se sim, adicione um alerta."*
