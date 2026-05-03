
# Setup: Tools & Agent Configuration

## IMPORTANTE — Antes de qualquer ação, entre em modo de planejamento.

Não execute nada ainda. Leia este documento inteiro, faça as perguntas necessárias
ao operador e só então execute o plano acordado.

---

## Fase 1 — Reconhecimento do ambiente

Identifique silenciosamente (sem perguntar):

1. **OS e shell disponível**
   - Detecte: Windows puro, WSL, Linux ou macOS
   - Identifique o shell: bash, zsh, PowerShell, cmd
   - Isso definirá a sintaxe dos scripts a serem criados

2. **Linguagens e frameworks do projeto**
   - Analise arquivos presentes na raiz: package.json, *.sln, *.csproj, tsconfig.json,
     pom.xml, go.mod, etc.
   - Identifique extensões predominantes em src/ ou app/
   - Identifique pastas a ignorar: node_modules, bin, obj, dist, .git, migrations,
     __pycache__, vendor, etc.

3. **Agentes de código presentes**
   - Verifique existência de arquivos de instrução conhecidos:
     - Claude Code: CLAUDE.md
     - OpenCode: opencode.md ou .opencode/
     - Copilot: .github/copilot-instructions.md
     - Cursor: .cursorrules ou .cursor/rules/
     - Outros: qualquer *.md na raiz com padrão de instrução de agente
   - Liste o que encontrou e o que está ausente

---

## Fase 2 — Perguntas ao operador

Com base no reconhecimento, apresente um resumo e faça as perguntas abaixo.
Agrupe tudo em uma única mensagem, não pergunte uma por vez.

**Resumo detectado:**
- OS/shell: [detectado]
- Projeto: [tipo detectado]
- Agentes com config existente: [lista]
- Agentes sem config: [lista]

**Perguntas:**

1. Os agentes que não têm arquivo de instrução devem ter um criado?
   Ou você quer configurar apenas um subconjunto? Quais?

2. Onde devo salvar o arquivo de instrução de cada agente?
   (Confirme ou corrija o padrão detectado para cada um)

3. A pasta `tools/` deve ficar na raiz do projeto?
   Se o projeto tiver monorepo ou subpastas relevantes, onde faz mais sentido?

4. Os scripts devem ter versão **dupla** (ex: `.sh` + `.ps1`) ou apenas o shell
   nativo do ambiente detectado?
   Se WSL: prefere `.sh` executável via WSL ou também quer `.ps1` para PowerShell nativo?

---

## Fase 3 — Criação das ferramentas

Após confirmação do operador, crie a pasta `tools/` com os seguintes scripts.
Adapte **sintaxe, extensão e comandos** ao ambiente detectado.

### 1. `search_symbol` — busca de símbolo no projeto

**Objetivo:** grep configurado para o projeto, sem ruído.

Deve:
- Buscar recursivamente em `./src` (ou equivalente detectado)
- Incluir apenas extensões relevantes ao projeto (detectadas na Fase 1)
- Excluir pastas de build/dependência detectadas
- Ignorar linhas que são apenas comentários
- Exibir: caminho, número da linha e trecho com contexto de 2 linhas (antes e depois)
- Aceitar o termo de busca como argumento posicional ($1 / %1)

### 2. `find_usages` — referências a um símbolo

**Objetivo:** encontrar onde um símbolo é *usado*, não apenas onde é definido.

Deve:
- Buscar o termo como palavra inteira (word boundary)
- Exibir apenas: arquivo e número da linha (formato compacto)
- Separar visualmente resultados por arquivo
- Aceitar o símbolo como argumento posicional

### 3. `list_changed_files` — arquivos modificados

**Objetivo:** retornar lista compacta de arquivos alterados para focar o contexto do agente.

Deve:
- Verificar se está em repositório git; se não, avisar e sair
- Por padrão: `git diff --name-only HEAD` (mudanças não commitadas)
- Aceitar argumento opcional: branch ou commit para comparar ($1)
- Filtrar automaticamente as pastas de build/dependência
- Exibir contagem total no final

### 4. `summarize_file` — resumo rápido de um arquivo

**Objetivo:** trazer contexto de um arquivo sem ler tudo — reduz tokens drasticamente.

Deve:
- Exibir: primeiras 40 linhas + últimas 20 linhas + total de linhas do arquivo
- Mostrar separador visual claro entre topo e fim
- Aceitar caminho do arquivo como argumento posicional
- Validar se o arquivo existe antes de executar

---

## Fase 4 — Arquivo de instrução dos agentes

Para cada agente confirmado pelo operador, crie ou atualize o arquivo de instrução
com a seguinte seção (adapte ao formato do agente):

```
## Tools disponíveis em /tools

Antes de explorar o projeto manualmente, sempre verifique se um dos scripts abaixo
resolve a necessidade. Eles são otimizados para este projeto e reduzem consumo de contexto.

- `search_symbol <termo>`    — grep configurado: extensões certas, sem node_modules/bin/obj
- `find_usages <símbolo>`    — referências a um símbolo (word boundary, saída compacta)
- `list_changed_files [ref]` — arquivos modificados desde HEAD ou ref informada
- `summarize_file <caminho>` — topo + fim do arquivo sem ler tudo

### Regra de uso obrigatória
1. Use `search_symbol` ou `find_usages` antes de abrir qualquer arquivo
2. Use `summarize_file` antes de `read_file` para decidir se vale ler tudo
3. Use `list_changed_files` no início de tarefas de revisão ou debugging
4. Só leia arquivos completos quando as ferramentas acima não forem suficientes

### Fluxos recomendados

#### Análise de impacto
1. `find_usages <símbolo>` → mapeia referências
2. `summarize_file` nos arquivos de definição → revela overloads e tipos
3. `Read` com offset nos call sites do meio de arquivo → expõe tipo da variável### Fluxos recomendados

#### Análise de impacto
1. `find_usages <símbolo>` → mapeia referências
2. `summarize_file` nos arquivos de definição → revela overloads e tipos
3. `Read` com offset nos call sites do meio de arquivo → expõe tipo da variável
4. **`summarize_file` nos tipos dos parâmetros → revela hierarquia**
   - **⚠️ GATE OBRIGATÓRIO:** Se `find_usages` ou `search_symbol` encontrarem **mais de uma definição** com o mesmo nome (ex: dois `GetTotal` em arquivos diferentes), você DEVE verificar a hierarquia de tipos ANTES de concluir a análise.
   - Verifique: o tipo do parâmetro é uma classe concreta própria do projeto, ou uma interface implementada em múltiplos projetos?
   - Em um monorepo, um método com o mesmo nome pode ser **dois métodos totalmente independentes** em projetos diferentes. Nunca assuma que são a mesma coisa.
5. Se o escopo do usuário for restrito a um projeto (ex: "no projeto X"), mas você encontrar definições do mesmo símbolo em outros projetos do monorepo, documente: **"Este relatório cobre apenas o projeto X. Foram detectadas definições homônimas no projeto Y (caminho), mas elas não foram analisadas."

#### Debugging / investigação de bug
1. `list_changed_files` → foca nos arquivos alterados recentemente
2. `search_symbol <termo relacionado>` → localiza onde o comportamento é definido
3. `summarize_file` nos candidatos → decide quais merecem leitura completa

#### Refatoração
1. `search_symbol <símbolo>` → onde está definido
2. `find_usages <símbolo>` → onde é usado
3. `summarize_file` nos arquivos de uso → entende o contexto antes de mudar

#### Onboarding em código desconhecido
1. `list_changed_files` → o que mudou recentemente
2. `search_symbol <conceito central>` → onde a lógica principal vive
3. `summarize_file` nos arquivos-chave → visão geral sem ler tudo

#### Debugging / investigação de bug
1. `list_changed_files` → foca nos arquivos alterados recentemente
2. `search_symbol <termo relacionado>` → localiza onde o comportamento é definido
3. `summarize_file` nos candidatos → decide quais merecem leitura completa

#### Refatoração
1. `search_symbol <símbolo>` → onde está definido
2. `find_usages <símbolo>` → onde é usado
3. `summarize_file` nos arquivos de uso → entende o contexto antes de mudar

#### Onboarding em código desconhecido
1. `list_changed_files` → o que mudou recentemente
2. `search_symbol <conceito central>` → onde a lógica principal vive
3. `summarize_file` nos arquivos-chave → visão geral sem ler tudo

```

---

## Fase 5 — Validação

Após criar tudo, execute cada script sem argumentos ou com argumento inválido
e confirme que:
- Exibe mensagem de uso (não falha silenciosamente)
- Tem permissão de execução (chmod +x em ambientes Unix)
- Está acessível pelo agente no contexto do projeto

Apresente ao operador um resumo do que foi criado, onde foi salvo e
qualquer passo manual necessário (ex: adicionar tools/ ao PATH, configurar .gitignore).
