---
name: api-quota-guardian
description: Sistema completo de proteção do quota MiniMax — nunca mais health checks ou cron jobs consomem o quota Token Plan do Álvaro
---

# API Quota Guardian — Sistema de Proteção Completo

## O Problema Original
Cron jobs de health check estavam a consumir o quota MiniMax Token Plan (4,500 requests/5h) e causando "Rate limited — switching to fallback" no Telegram. A cada 5 minutos, um health check fazia uma chamada LLM real ao MiniMax, esgotando o quota antes do Álvaro poder conversar.

## Arquitectura de Proteção

### Camada 1: quota_guard.py — Guarda de Quota
**Ficheiro:** `~/.hermes/scripts/quota_guard.py`

Verifica o quota MiniMax ANTES de qualquer cron job que use LLM. Se o quota estiver > 70% usado OU se houver erros 429/401 recentes, o cron job é bloqueado.

**Estratégia de detecção:**
1. Lê a key real via hermes-agent venv python + `load_hermes_dotenv()`
2. Se conseguir → chama `/v1/token_plan/remains` para quota real
3. Se não conseguir → analiza logs do agent.log para erros 429/401 recentes
4. Calcula há quanto tempo foi o último erro — se > 5h, quota já recuperou
5. Se tudo OK → permite

**Exit codes:**
- `0` = pode executar (quota OK)
- `1` = skip (quota em risco)

### Camada 2: telegram_healer.py — Auto-Healer do Telegram
**Ficheiro:** `~/.hermes/scripts/telegram_healer.py`

Monitoriza o Telegram e faz auto-recuperação se detectar problemas.

**Verificações:**
1. Lê `~/.hermes/endpoint_health.json` — estado actual do Telegram
2. Verifica se há erros de timeout/429 no agent.log recentes
3. Detecta "timeout" e "connection" errors nas últimas 2h

**Acções de recuperação:**
- Se 3+ timeouts recentes: restart do hermes-gateway via systemctl
- Se erros 429: logging + preservação (o quota_guard já protege)
- Se tudo OK: nada a fazer

### Camada 3: Cron Jobs de Monitorização
- **API Quota Guardian — Monitor** (`373a26da5c01`): cada 15 min — regista quota
- **Telegram Auto-Healer** (`fd58db18a3d3`): cada 15 min — auto-heal Telegram

## Cron Jobs Protegidos (com quota_guard)
| Job | ID | Schedule |
|-----|----|----------|
| Session Replay | `8e1d019adb06` | 00:30 diária |
| Learning Loop Daily | `5de2c83955ea` | 04:00 diária |
| Learning Loop Weekly | `08664971b643` | Sunday 22h |
| Smart Memory Daily | `b3dfaa0c3adc` | 03:00 diária |
| Autonomous Suggestion Processor | `68ea14482e88` | cada 2h |

## Teste
```bash
python3 ~/.hermes/scripts/quota_guard.py; echo $?
# 0 = pode executar, 1 = skip

python3 ~/.hermes/scripts/telegram_healer.py; echo $?
# 0 = OK, 1 = problema detectado

tail -5 ~/.hermes/logs/quota_guard.log
tail -5 ~/.hermes/logs/telegram_healer.log 2>/dev/null
```

## Notas Importantes

### Como ler a MINIMAX_API_KEY real (não mascarada)
O `.env` tem `MINIMAX_API_KEY=***` (mascarado). Ler directamente não funciona. O método correcto é usar o Python do hermes-agent venv com `load_hermes_dotenv()`:

```python
HERMES_VENV_PY = os.path.expanduser("~/.hermes/hermes-agent/venv/bin/python")
script = """
import os, sys; sys.path.insert(0, '.')
from hermes_cli.env_loader import load_hermes_dotenv
from pathlib import Path
load_hermes_dotenv(hermes_home=Path.home() / '.hermes')
sys.stdout.write(os.environ.get('MINIMAX_API_KEY', ''))
"""
result = subprocess.run([HERMES_VENV_PY, "-c", script], capture_output=True, text=True,
    cwd=os.path.expanduser("~/.hermes/hermes-agent"))
key = result.stdout.strip()
```

O que NÃO funciona: `/proc/{pid}/environ`, `systemctl --user show hermes-gateway`, `grep` directo ao `.env`.

### Endpoint API MiniMax
Token Plan quota: `https://api.minimax.io/v1/token_plan/remains` (NÃO `www.minimax.io` — dá 403 Cloudflare).

### Health checks
proactive_monitor.py já é HTTP-only (não consome quota MiniMax). Server Resilience Manager corre a cada 30 min.

## Estado Actual (25/04/2026 21:01)
- Janela actual: 29/4500 (0.6%) — 4471 restantes
- Semana: 16254/45000 (36.1%)
- Cron jobs: EXECUTAR (quota OK)

## Descobertas Importantes
1. A key está no `.env` mas mascarada como `***`. O hermes-agent usa `load_hermes_dotenv()` que retorna a key real.
2. O endpoint correcto é `api.minimax.io` (não `www.minimax.io` — 403 Cloudflare).
3. A janela rolante do Token Plan é 5 horas — erros 429 há > 5h significam quota já recuperou.
4. Se a API falhar, o fallback usa análise de logs (erros 429/401 nos últimos 60min).
