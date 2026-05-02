# Bianinho — Backup Completo

**Versão:** 1.0
**Data:** 2 de Maio de 2026
**Backup de:** Servidor de Álvaro Bianoi

---

## O Que Está neste Backup

| Componente | Conteúdo | Tamanho |
|---|---|---|
| `customizations.patch` | 4 commits de personalização ao Hermes Agent | 60KB |
| `skills/` | 70 skills do Bianinho | ~14MB |
| `config/` | Config, estado, autónomo, inbox | ~84KB |
| `autonomous/` | Ciclo autónomo (15min), mandate, inbox | ~50KB |
| **RAG Knowledge Base** | Memória vectorial (não está aqui) | ~2GB |

O **código do Hermes Agent** não está aqui porque é do NousResearch e pode ser clonado livremente:
```
https://github.com/NousResearch/hermes-agent
```

---

## Instalação no MacBook

### Pré-requisitos

```bash
# macOS
sw_vers

# Homebrew (se não tiver)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Python 3.11+
brew install python@3.11
brew link python@3.11 --force

# Verificar
python3 --version   # deve ser 3.11+
```

### Passo 1 — Clonar Hermes Agent (NousResearch)

```bash
mkdir -p ~/.hermes
cd ~/.hermes
git clone https://github.com/NousResearch/hermes-agent.git hermes-agent
cd hermes-agent
```

### Passo 2 — Obter Este Backup

```bash
# Clonar este repo
git clone https://github.com/AlvaroBiano/bianinho-backup-1777760438.git ~/Downloads/bianinho-backup
```

### Passo 3 — Aplicar Personalizações

```bash
cd ~/.hermes/hermes-agent
git apply /caminho/para/customizations.patch
```

### Passo 4 — Instalar Dependências

```bash
pip install -e ".[messaging,voice,acp]"
# Ou com Python do Homebrew:
python3 -m pip install -e ".[messaging,voice,acp]"
```

### Passo 5 — Copiar Skills

```bash
cp -r skills/ ~/.hermes/skills/
```

### Passo 6 — Copiar Config

```bash
cp -r config/* ~/.hermes/
```

### Passo 7 — Configurar .env

```bash
nano ~/.hermes/.env
```

Adicionar:
```env
MINIMAX_API_KEY=your_key_aqui
MINIMAX_BASE_URL=https://api.minimaxi.com/v1
TELEGRAM_BOT_TOKEN=your_telegram_token  # opcional
```

Guardar com `Ctrl+O`, `Enter`, `Ctrl+X`.

### Passo 8 — Verificar

```bash
hermes --version
hermes --chat
```

---

## Pasta do Backup — O Que Está Incluído

| Ficheiro | Descrição |
|---|---|
| `config/config.yaml` | Config principal do Hermes |
| `config/retry_guard.json` | Estado de retries |
| `config/inbox.json` | Inbox de tarefas |
| `config/self_improvement_state.json` | Estado de auto-melhoria |
| `config/proactive_suggestions_log.json` | Log de sugestões proativas |
| `config/rag_status.json` | Estado do RAG |
| `config/mandate.md` | Mandato do Bianinho |
| `config/inbox.py` | Script inbox |
| `config/cycle.py` | Ciclo autónomo |
| `config/state.py` | Estado |
| `autonomous/mandate.md` | Mandato (6º Pilar) |
| `autonomous/inbox.py` | Script inbox |
| `autonomous/cycle.py` | Ciclo de decisão |
| `autonomous/state.json` | Estado actual |

---

## RAG Knowledge Base (~2GB)

O RAG **não está neste backup** por causa do tamanho. No servidor, faz backup para Google Drive:

```bash
# Criar tarball do RAG
tar -czvf rag_backup.tar.gz ~/KnowledgeBase/

# Upload para Google Drive (via rclone)
rclone copy rag_backup.tar.gz gdrive:bianinho-backup/
```

No MacBook, download e extrair:
```bash
rclone copy gdrive:bianinho-backup/rag_backup.tar.gz ./
tar -xzvf rag_backup.tar.gz -C ~/
```

---

## Notas

- **Python >= 3.11** é obrigatório
- O Hermes Agent precisa da API key da MiniMax para funcionar
- As skills são compatíveis com ambas as instalações (servidor e MacBook)
- O RAG não é obrigatório para o agente funcionar — só para memória vetorial

---

## Repo Principal do Hermes Agent

O fork do Álvaro com as tuas personalizações:
**https://github.com/AlvaroBiano/hermes-agent**
