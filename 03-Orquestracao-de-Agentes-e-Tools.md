# Orquestração de Agentes e Tool Calling

Para extrair performance máxima do Antigravity ou Claude Code, precisamos ditar não apenas *o que* ele deve fazer, mas *como* ele deve orquestrar suas próprias ferramentas (Browser, Web Search, Execução de Comandos, File System).

## 1. Gatilhos de Validação Pré-Escrita
A IA não deve usar ferramentas apenas para coletar dados, mas como **Gatekeepers** (validadores) antes de executar ações destrutivas ou criar arquivos definitivos.
*   **Padrão Extraído:** *"Para CADA dado, valide no portal oficial ANTES de registrar no Markdown."*
*   **Na prática com Tools:** O agente é forçado a rodar um `search_web` ou usar o `browser_subagent`, ler a resposta e só então usar o `write_to_file`.

## 2. Conexão de Conhecimento (Grafo de Informação)
Agentes perdem a noção do todo ao focar nas partes. O padrão "Mapa de Conexões" é vital na orquestração:
*   Crie um "Módulo Final" no prompt cujo único objetivo seja ler todos os módulos anteriores gerados e mapear as conexões entre eles.
*   **O que isso ensina ao Agente:** Ele aprende a usar ferramentas de leitura (`view_file` ou `grep_search`) no próprio trabalho que acabou de gerar para criar links bidirecionais (estilo Obsidian) de altíssimo valor.

## 3. Tool Calling Eficiente e Minimizado
Instruções aprendidas para evitar loops infinitos ou uso burro de ferramentas:
*   Agentes devem consolidar pesquisas. Em vez de fazer 10 buscas separadas, devem buscar a raiz.
*   O uso de tags e metadados no final do processo (`#IBS`, `#Faseamento`) garante que os arquivos salvos pelo Agente via `write_to_file` sejam perfeitamente indexáveis e orquestráveis por outros scripts no futuro.

## 4. A Técnica do "Feynman Prompting" no Output
Mesmo pesquisando documentos altamente técnicos (Leis, Logs de Servidor, Documentações de API), a orquestração da saída deve forçar a "Tradução Cognitiva".
*   Sempre exija que a IA traduza a tecnicidade para uma **Explicação Simples** e uma **Analogia**.
*   **Impacto na Performance:** Isso prova que o Agente "entendeu" o conceito via Tools, em vez de simplesmente copiar e colar trechos rasos da internet (web scraping sem processamento). Se ele não consegue fazer a analogia, o Agente detecta que a pesquisa foi fraca e pesquisa novamente.
