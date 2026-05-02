# Code Agent tools setup prompt

Prompt para criação automática de ferramentas que otimizam 
o consumo de tokens em agentes de código.

A ideia é simples: em vez do agente explorar o projeto às cegas,
ele usa ferramentas pré-configuradas que filtram antes de ler.

## O que o prompt cria

- `search_symbol` — grep configurado pro projeto, sem ruído
- `find_usages` — referências a um símbolo com word boundary
- `list_changed_files` — arquivos modificados desde HEAD ou ref informada
- `summarize_file` — topo + fim do arquivo sem ler tudo

## Como usar

Cole o prompt no seu agente de código preferido.
Ele vai detectar o ambiente, fazer perguntas e criar as ferramentas automaticamente.

## Contribuindo

Achou uma melhoria? Faltou algo? Abre uma PR.
Esse repositório é uma proposta em construção, não uma solução definitiva.
