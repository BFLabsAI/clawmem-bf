# BF Labs — Fork do ClawMem

Fork de manutenção da BF Labs sobre o **upstream** [`yoloshii/ClawMem`](https://github.com/yoloshii/ClawMem).
O objetivo é acompanhar o upstream e carregar, por cima, apenas as alterações da BF que fazem sentido.

## O que este fork muda em relação ao upstream

- **`Authorization: Bearer` nas requisições de LLM remoto** (`src/llm.ts`).
  O upstream envia o header de auth apenas nas chamadas de *embedding*; a chamada de LLM
  (expansão de query) ia sem auth. Adicionamos o campo `remoteLlmApiKey` (env `CLAWMEM_LLM_API_KEY`),
  espelhando o padrão já usado no embedding. Necessário para usar provedores de LLM em nuvem
  (ex.: OpenRouter) que exigem chave na chamada de chat completions.

## Instalação

```bash
# direto do upstream (vanilla):
npm i -g clawmem

# ou a partir deste fork (com a alteração da BF):
git clone https://github.com/BFLabsAI/clawmem-bf.git
cd clawmem-bf && npm i -g .
```

## Configuração (wrapper de providers — exemplo)

O ClawMem precisa de modelos de embedding/LLM/rerank. Mantenha a chave **fora** dos configs dos
agentes usando um wrapper que injeta a config e sobe o MCP. **Use placeholders — nunca commite chave real.**

```bash
#!/usr/bin/env bash
set -euo pipefail
KEY="${OPENROUTER_API_KEY:?defina OPENROUTER_API_KEY}"   # <-- placeholder; defina no ambiente

export CLAWMEM_EMBED_URL="https://openrouter.ai/api"   CLAWMEM_EMBED_MODEL="qwen/qwen3-embedding-8b"  CLAWMEM_EMBED_API_KEY="$KEY"
export CLAWMEM_LLM_URL="https://openrouter.ai/api"     CLAWMEM_LLM_MODEL="qwen/qwen3.6-flash"          CLAWMEM_LLM_API_KEY="$KEY"
export CLAWMEM_RERANK_URL="https://openrouter.ai/api"  CLAWMEM_RERANK_MODEL="cohere/rerank-v3.5"        CLAWMEM_RERANK_API_KEY="$KEY"
export CLAWMEM_NO_LOCAL_MODELS="true"

exec clawmem mcp
```

> `CLAWMEM_LLM_API_KEY` é justamente o que a alteração deste fork passa a respeitar na chamada de LLM.

## Sincronizar com o upstream (manter o fork em dia)

```bash
git remote add upstream https://github.com/yoloshii/ClawMem.git   # uma vez
git fetch upstream
git rebase upstream/main      # reaplica os commits da BF por cima da última versão do upstream
# resolver conflitos (se houver), depois:
git push --force-with-lease origin main
```
