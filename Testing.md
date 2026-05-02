# Testando as ferramentas

Passo a passo usado pra validar a diferença de consumo de tokens 
com e sem as ferramentas.

## A tarefa

Mapear todos os lugares onde um método é chamado e avaliar 
o impacto de mudar a assinatura.

## Pré-requisitos

- Claude Code instalado e rodando na raiz do projeto
- Rodada 2 requer as ferramentas já criadas pelo prompt de setup

## Rodada 1 — Sem ferramentas

Abre o Claude Code na raiz do projeto e cola:

```
Quero entender o impacto de mudar a assinatura do método [nome do método].

Encontre todos os lugares onde ele é chamado no projeto.

Ao final apresente um relatório com:
- Lista de todos os arquivos que você abriu durante a busca
- Ferramentas que utilizou e quantas vezes cada uma foi chamada
- Sua estimativa de tokens consumidos nessa tarefa
```

Anota os números e tira print do relatório final.

## Rodada 2 — Com ferramentas

Abre uma sessão nova (contexto zerado) e cola:

```
Quero entender o impacto de mudar a assinatura do método [nome do método].

Antes de qualquer coisa verifique se existe uma pasta tools/ na raiz 
do projeto. Se existir, use as ferramentas disponíveis lá pra mapear 
onde o método é chamado. Não abra arquivos antes de usar as ferramentas.

Ao final apresente um relatório com:
- Lista de todos os arquivos que você abriu durante a busca
- Ferramentas que utilizou e quantas vezes cada uma foi chamada
- Sua estimativa de tokens consumidos nessa tarefa
```

Anota os mesmos números e tira print do relatório final.

## O que comparar

- Total de chamadas de ferramenta
- Arquivos abertos via Read
- Tokens estimados

## Observação importante

A análise sem ferramentas tende a ser mais profunda semanticamente.
A análise com ferramentas tende a ser mais eficiente em tokens.

Se precisar de profundidade semântica sem abrir mão da eficiência:
use o `find_usages` pra mapear os arquivos, depois o `summarize_file` 
em cada resultado, e só então abra os que realmente precisam 
de leitura completa.
