# Tabela de Conceitos — Claude Architect Foundations
## Dia 1 (Arquitetura de Agentes) + Dia 2 (Claude Code)

---

## 1. Mecanismos de Enforcement e Controle

| Conceito | O que é | Garantia | Quando usar | Cuidado / Pegadinha |
|---|---|---|---|---|
| **CLAUDE.md** | Arquivo Markdown lido automaticamente toda sessão; conteúdo é anexado (append) ao prompt | Nenhuma — é *guidance*, o modelo segue "na maioria das vezes" | Convenções gerais, sempre relevantes (naming, stack, comandos) | Cresce demais → compete por atenção consigo mesmo → Claude ignora partes. Não é bug, é como o mecanismo funciona |
| **Hooks** | Código real que roda em pontos fixos do lifecycle (evento) | **Total** — determinístico, sempre executa | Regra crítica que não pode ser pulada (ex: nunca push na main, bloquear segredo) | Não é "instrução seguida", é "código que roda" — essa é a diferença central com CLAUDE.md/Skills |
| **Skills** | Pasta com `SKILL.md` + arquivos auxiliares opcionais | Nenhuma formalmente, mas confiável se a *description* for boa | Procedimento específico, usado ocasionalmente (ex: gerar relatório semanal) | Dispara por match de **descrição**, não por comando manual (geralmente) |
| **Subagents** | Agente que roda em contexto isolado, próprio context window | Depende do prompt/tarefa dado a ele | Tarefa que gera ruído (exploração, revisão) e você só quer o resultado final | Não herda nada automaticamente do agente principal — tudo precisa ser explícito |
| **Permission Modes** | Nível de autonomia geral da sessão (o que roda sem perguntar) | Variável por modo (Auto usa classificador, não é 100% garantido) | Ajustar quanto de "mão livre" dar ao Claude na sessão toda | Auto mode classificador checa **intenção**, não **correção** — não é substituto de hook |

**Regra de ouro para decidir onde uma regra vai (do texto de Skill de Verificação):**
> Convenção sempre válida → CLAUDE.md. Procedimento ligado a um tipo de tarefa → Skill. Regra que NÃO pode ser pulada → Hook (código que roda, não instrução que se segue).

---

## 2. CLAUDE.md — Detalhes

| Aspecto | Descrição |
|---|---|
| Mecanismo técnico | Conteúdo é **anexado (append)** ao prompt toda sessão |
| Hierarquia (4 níveis, carregados todos juntos, empilhados) | **Managed policy** (org, não exclui) → **User** (pessoal, todos projetos) → **Project** (repo, versionado, time) → **Local** (git-ignored, só seu, só nesse repo) |
| Imports (`@arquivo.md`) | Organizam o arquivo em partes, mas **NÃO reduzem contexto** — tudo é expandido inline no launch |
| Regra boa | Específica e checável ("routes em src/api/handlers", não "follow best practices") + nomeia o substituto ("use named exports", não só "don't use default exports") |
| Ênfase (IMPORTANT/YOU MUST) | É um **orçamento limitado** — gastar em tudo anula o efeito; usar só nas 2-3 regras que mais doem se quebradas |
| Workflow recomendado | Começar **sem** o arquivo → observar onde corrige repetidamente → gerar com `/init` → revisar continuamente como "bug report" |

---

## 3. Gerenciamento de Sessões Longas e Contexto

| Comando/Conceito | O que faz | Quando usar |
|---|---|---|
| **Plan Mode** | Read-only; pesquisa e propõe plano sem editar nada | Escopar antes de começar a execução |
| **`/compact [instrução]`** | Resume a conversa até aquele ponto, libera espaço, mantém memória do que foi feito | Continuar a **mesma** tarefa/feature após bater no limite de contexto |
| **`/clear`** | Apaga tudo, zero memória da sessão anterior | Começar uma feature **nova**, não relacionada (evita viés da conversa anterior) |
| **`/context`** | Diagnóstico: tamanho, categorias que mais consomem, gráfico visual | Decidir se/quando compactar |
| **Rewind (duplo Esc)** | Volta a um checkpoint (cada prompt cria um) | Corrigir rumo sem precisar re-prompt |
| **`/goal`** | Define condição de conclusão; Claude continua até um avaliador confirmar via **transcript** | Quando "pronto" é mais fácil de descrever que os passos |
| **`/loop`** | Roda um prompt em intervalo entre turnos | Monitorar algo externo (CI, deploy) e agir na mudança |
| **Worktrees** | Cada sessão paralela get sua própria árvore de arquivos isolada | Rodar múltiplos agentes no mesmo codebase sem conflito |
| **`.worktreeinclude`** | Lista arquivos git-ignored a copiar pra cada worktree (ex: `.env`) | — |

**Dicas de economia de contexto:**
- Prompt **específico** custa menos contexto no total que um vago (vago força exploração/raciocínio extra)
- Gerenciar MCP servers ativos (desligar os não usados)
- Preferir CLI a MCP quando existe equivalente
- Usar Skills em vez de manter tudo carregado
- Delegar exploração a Subagents

---

## 4. Subagents — Detalhes

| Aspecto | Descrição |
|---|---|
| Isolamento | Contexto próprio, não herda nada automaticamente da conversa principal |
| Criação | Via `/agents` (menu guiado) **ou** pedindo direto na conversa — ambos geram o mesmo artefato: `.md` + YAML frontmatter |
| Uso | Automático (Claude decide baseado na descrição) ou manual (você menciona o nome) |
| Persistent memory | Retém memória entre conversas diferentes (bom para uso recorrente no mesmo projeto) |
| Preload de skills | Chave `skill` no frontmatter — mas a skill carrega **inteira** dentro do subagent (diferente do comportamento enxuto no contexto principal) |
| Code review com subagent | Roda com "olhos frescos", sem o viés de quem escreveu o código; **restringir a tools read-only**; configuração deve ser **versionada no repo** (project-level) |

---

## 5. MCP (Model Context Protocol)

| Aspecto | Descrição |
|---|---|
| O que é | Padrão aberto que conecta Claude a ferramentas/dados externos — dá capacidade de **ação**, não só resposta em texto |
| Adicionar servidor | `claude mcp add` |
| Tipos | **HTTP** (remoto, hospedado pelo provedor) / **Stdio** (processo local) |
| Gerenciar | `/mcp` — ver conectados, status, desativar |
| Escopos (mesma lógica de CLAUDE.md) | **Local** (você, 1 projeto) → **User** (você, todos projetos) → **Project** (`.mcp.json`, versionado, time todo) |
| Custo de contexto | Definições de tool ficam carregadas **mesmo sem uso ativo** |
| Mitigação 1 | Desativar servidores não usados (`/mcp`) |
| Mitigação 2 | Preferir **CLI** quando existe equivalente (`gh`, `aws`) — não adiciona definição persistente |
| Mitigação 3 | Preferir **Skill** — carrega só nome/descrição até precisar |
| Tool Search Mode | Se tools MCP passam de **10%** do context window, Claude Code muda automaticamente pra descoberta sob demanda — porém **menos confiável** |

---

## 6. Skills — Detalhes

| Aspecto | Descrição |
|---|---|
| Estrutura | Pasta com `SKILL.md` (obrigatório) + `scripts/`, `references/`, `assets/` (opcionais) |
| Localização | `.claude/skills/` (projeto) ou `~/.claude/skills/` (pessoal) |
| Carregamento em 2 estágios | (1) Nome + description sempre no contexto → (2) conteúdo completo do `SKILL.md` só quando o Claude decide usar |
| Custo de conteúdo não acessado | **Zero** — pode ter documentação enorme sem penalidade se não for usada |
| Scripts | Executados (não carregados como texto) — só o **output** consome tokens; garante confiabilidade determinística |
| Gatilho | **Description** do YAML frontmatter — bate com o pedido → dispara sozinha |
| Comando manual | Existe às vezes (ex: `/commit-push-pr`), depende de configuração — não é a regra geral |
| Criação | Manual (você escreve a pasta) ou pedindo ao Claude para gerar — ambos válidos |
| Regra de quando criar | Mesma instrução multi-passo digitada 2x → vira skill |
| Skill de Verificação (exemplo âncora) | Dispara após mudança de código; roda testes; lê diff; confirma que **nenhum teste foi enfraquecido**; reporta com evidência. "Done" = gates executados e observados, não "código parece certo" |

---

## 7. Hooks — Detalhes Técnicos

| Aspecto | Descrição |
|---|---|
| Eventos principais | `PreToolUse` (antes da tool — enforcement) / `PostToolUse` (depois, sucesso — formatação/lint) / `Stop` (fim do turno — pode recusar) / `SubagentStop` / `PreCompact` / `PostCompact` / `InstructionsLoaded` / `SessionStart` / `UserPromptSubmit` / `Notification` |
| `PreToolUse` — decisão JSON | Campo `permissionDecision`: `allow` / `deny` / `ask` (+ `defer`, raro, só para `-p` não-interativo) |
| `updatedInput` | Permite **reescrever** a chamada (ex: redigir um segredo) em vez de só bloquear — mas **substitui o objeto inteiro**, precisa ecoar campos não alterados |
| Exit code 0 | Sucesso. JSON no stdout é parseado; texto puro só é adicionado ao contexto em `SessionStart`, `UserPromptSubmit`, `UserPromptExpansion` |
| Exit code 2 | **Bloqueia** — stderr volta pro Claude como feedback/contexto |
| Exit code 1 (ou outro) | **NÃO bloqueia** — é só logado, Claude segue em frente (pegadinha clássica: 1 parece erro mas não bloqueia) |
| `PostToolUse` e bloqueio | Tarde demais pra impedir a call (já rodou), mas ainda pode alimentar texto de volta |
| `Stop` e bloqueio | Exit 2 pode recusar o fim do turno — "você não terminou ainda" (usado com testes) |
| Eventos que ignoram bloqueio | `Notification` e `SessionStart` — sempre mostram stderr e seguem, não importa o exit code |
| Reinjetar contexto pós-compactação | Usar **`SessionStart` com matcher `compact`** — **NÃO** `PostCompact` |
| Escopo/compartilhamento | `.claude/settings.json` = project-level, versionável; usar `CLAUDE_PROJECT_DIR` para paths de script robustos a working directory |
| Configuração | Via `/hooks` ou editando `settings.json` diretamente |

---

## 8. Permission Modes

| Modo | Comportamento | Uso típico |
|---|---|---|
| **Manual** | Só leitura sem perguntar; resto pede confirmação | Trabalho supervisionado de perto |
| **Accept edits** | Roda leitura, edição de arquivo, comandos bash comuns sem perguntar | Iterar em código revisado depois |
| **Plan** | Só leitura; propõe sem editar | Escopar antes de agir |
| **Auto** | Aceita tudo, mas um **classificador separado** revisa cada ação antes de rodar (checa **intenção**) | Trabalho "mãos livres", combinado com stop hook para checar **correção** |
| **Don't ask** | Só tools pré-aprovadas rodam; resto é auto-negado, sem prompt | Pipelines de CI, jobs agendados, sem humano disponível |
| **Bypass permissions** | Pula todas as checagens (= `dangerously-skip-permissions`) | **Só** em container/VM isolada |

Ciclar entre os 4 do dia a dia (Manual, Accept edits, Plan, Auto): **shift-tab**.

⚠️ **Auto mode ≠ garantia total.** O classificador é probabilístico e checa intenção, não correção. Para uma regra 100% inegociável, a resposta correta continua sendo **hook**.

---

## 9. Conceitos do Dia 1 (Arquitetura de Agentes) — Referência Rápida

| Conceito | Ponto-chave |
|---|---|
| **Agentic loop** | Decisão de continuar/parar vem do `stop_reason` estrutural da API (`tool_use` vs `end_turn`) — nunca de parsing de texto livre ou contador fixo de iterações |
| **Hub-and-spoke** | Coordinator decompõe, delega, trata erro, agrega. Falha de escopo/cobertura incompleta = quase sempre problema de **decomposição do coordinator**, não dos subagents ou da síntese |
| **Contexto isolado dos subagents** | Nada é herdado automaticamente — tudo que o subagent precisa deve estar explícito na Task |
| **Hook vs instrução no prompt** | Consequência financeira/crítica → hook (garantido). Comportamento geral → prompt (probabilístico) é aceitável |

---

## 10. Fio Condutor para a Prova

Em praticamente toda questão de cenário, a resposta certa **corrige a causa raiz**, não empilha uma camada de infraestrutura em cima de um design furado:

- Descrição de tool ruim → melhore a descrição, não crie um classificador por cima
- Regra perigosa → hook programático, não reforço no prompt
- Contexto faltando pro subagent → empacote explicitamente, não invente "memória compartilhada mágica"
- Teste passando mas código errado → verifique se o teste foi enfraquecido, não confie só no "verde"
- Classificador de Auto mode não pega bug de lógica → combine com stop hook, não confie só no classificador
