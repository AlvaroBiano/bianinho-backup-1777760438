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

## Instalação no MacBook — Resumo

```bash
# 1. Clonar Hermes Agent (NousResearch)
git clone https://github.com/NousResearch/hermes-agent.git ~/.hermes/hermes-agent
cd ~/.hermes/hermes-agent

# 2. Aplicar personalizações do Bianinho
git apply /caminho/para/customizations.patch

# 3. Instalar
pip install -e ".[messaging,voice,acp]"

# 4. Copiar skills
cp -r skills/ ~/.hermes/skills/

# 5. Copiar config
cp -r config/ ~/.hermes/

# 6. Configurar API Keys
nano ~/.hermes/.env
```

---

## Instalação Detalhada — MacBook

### Pré-requisitos

```bash
# macOS (verificar)
sw_vers

# Homebrew (instalar se necessário)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Python 3.11+ (Homebrew)
brew install python@3.11
brew link python@3.11 --force

# Git
brew install git

# Verificar
python3 --version   # deve ser 3.11+
git --version
```

### Passo 1 — Clonar Hermes Agent

```bash
mkdir -p ~/.hermes
cd ~/.hermes
git clone https://github.com/NousResearch/hermes-agent.git hermes-agent
cd hermes-agent
```

### Passo 2 — Obter Este Backup

**Opção A — Download directo** (mais fácil):
```bash
# Download do ZIP deste repo
curl -L https://github.com/AlvaroBiano/hermes-agent-custom/archive/refs/heads/main.zip -o backup.zip
unzip backup.zip
cd hermes-agent-custom-main
```

**Opção B — Git clone**:
```bash
git clone https://github.com/AlvaroBiano/hermes-agent-custom.git ~/Downloads/hermes-agent-custom
```

### Passo 3 — Aplicar Personalizações

```bash
cd ~/.hermes/hermes-agent
git apply /caminho/para/customizations.patch
```

Se houver conflitos (raro):
```bash
git apply --3way /caminho/para/customizations.patch
# Resolva conflitos manualmente se necessário
```

### Passo 4 — Instalar Dependências

```bash
pip install -e ".[messaging,voice,acp]"

# Ou com Homebrew Python:
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

### Passo 7 — Configurar .env (API Keys)

```bash
nano ~/.hermes/.env
```

Adicionar:
```env
# MiniMax API (obrigatório para o Bianinho funcionar)
MINIMAX_API_KEY=your_key_aqui
MINIMAX_BASE_URL=https://api.minimaxi.com/v1

# Opcional — Telegram
TELEGRAM_BOT_TOKEN=your_telegram_token
```

Guardar com `Ctrl+O`, `Enter`, `Ctrl+X`.

### Passo 8 — Verificar Instalação

```bash
hermes --version
hermes --chat
```

Se o Bianinho responder, está pronto!

---

## RAG Knowledge Base (~2GB)

O RAG (memória vetorial) **não está neste backup** por causa do tamanho.

### Para fazer backup do RAG no servidor:

```bash
# No servidor (já configurado no cron)
# O script já faz backup automático para Google Drive
# Ver: ~/KnowledgeBase/backup/
```

### Para restaurar o RAG no MacBook:

1. No servidor, criar tarball do RAG:
   ```bash
   tar -czvf rag_backup.tar.gz ~/KnowledgeBase/
   ```

2. Upload para Google Drive:
   ```bash
   # Via rclone (configurado)
   rclone copy rag_backup.tar.gz gdrive:bianinho-backup/
   ```

3. No MacBook, download e extrair:
   ```bash
   curl -L "link_do_google_drive" -o rag_backup.tar.gz
   tar -xzvf rag_backup.tar.gz -C ~/
   ```

---

## Pasta do Backup

O diretório `config/` inclui:

| Ficheiro | Descrição |
|---|---|
| `config.yaml` | Config principal do Hermes |
| `auth.json` | Auth state |
| `retry_guard.json` | Estado de retries |
| `inbox.json` | Inbox de tarefas |
| `self_improvement_state.json` | Estado de auto-melhoria |
| `proactive_suggestions_log.json` | Log de sugestões proativas |
| `rag_status.json` | Estado do RAG |
| `mandate.md` | Mandato do Bianinho |
| `inbox.py` | Script inbox |
| `cycle.py` | Ciclo autónomo |
| `state.py` | Estado |
| `state.json` | Estado actual |

---

## Notas Importantes

- **Python >= 3.11** é obrigatório
- O Hermes Agent precisa de API key da MiniMax para funcionar
- As skills são compatíveis com ambas as instalações (servidor e MacBook)
- O RAG não é obrigatório para o agente funcionar — só para memória vetorial

---

## Comandos do Bianinho (MacBook)

```bash
# Chat interactivo
hermes --chat

# TUI (interface visual)
hermes --tui

# Com provider específico
hermes --chat --provider minimax --model MiniMax-M2.7

# Ver logs
hermes logs --errors
```
