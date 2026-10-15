# Guia REA em português: analisar páginas de vendas, funis e rastreamento

REA (https://github.com/morluto/rea) é um MCP/CLI que inspeciona páginas e apps **localmente** e devolve evidências.
Use só em páginas **suas** ou que você tenha autorização para analisar (veja `DISCLAIMER.md` do projeto).

## 1. Instalação

```bash
npx rea-agents setup        # registra o MCP no seu agente (Claude Code, Cursor, Codex...)
# reinicie o agente depois
```
Requisito: Node.js 22.19+ (ou 24.11+ / 26+). Para sites, basta um navegador Chrome/Chromium.
Para só usar o terminal: `npx -y rea-agents@latest <comando>`.

## 2. Passo a passo: analisar uma página de vendas

**a) Abra o Chrome com a porta de depuração (CDP):**
```bash
chromium --remote-debugging-port=9222 --user-data-dir=/tmp/perfil-rea "https://SUA-PAGINA"
# no Windows: chrome.exe --remote-debugging-port=9222 --user-data-dir=C:\perfil-rea
```

**b) Liste as abas e pegue o `target_id`:**
```bash
npx -y rea-agents@latest list-browser-targets http://127.0.0.1:9222 --format json
```
Atenção: use `127.0.0.1` (não `localhost`).

**c) Inspecione a página (DOM, scripts, rede, console, chaves de storage):**
```bash
npx -y rea-agents@latest inspect-web-page http://127.0.0.1:9222 <TARGET_ID> \
  --allowed-origins https://sua-pagina.com \
  --observation-ms 5000 --include-storage-keys --include-console-text --format json
```
Importante: `--allowed-origins` recebe **uma origem exata** (sem caminho e sem aspas de array).

**d) Analise os scripts (rotas, endpoints, bibliotecas):**
```bash
npx -y rea-agents@latest analyze-web-bundle http://127.0.0.1:9222 <TARGET_ID> \
  --allowed-origins https://sua-pagina.com --observation-ms 3000 --format json
```

**e) Compare duas versões (A/B) ou duas capturas:** `compare-web-captures`, `compare-web-screenshots`.

## 3. O que cada análise mostra para marketing

| Quero saber... | Onde olhar no resultado |
|---|---|
| Quais scripts de rastreamento a página carrega (Meta, TikTok, GA4, Hotmart/Kiwify) | `scripts`, `resources` (URLs e origens) |
| Quais eventos disparam (PageView, InitiateCheckout...) | `console` (com `--include-console-text`), código em `--include-script-sources` |
| Para onde a página chama APIs | `observations.endpoints` na análise de bundle |
| Parâmetros de UTM nos links e botões de compra | `metadata.dom_urls` |
| Cookies/IDs guardados (só os **nomes**) | `storage.local_storage_keys` (valores ficam ocultos) |
| Estrutura de títulos, botões, formulários | `dom` e `accessibility` |

## 4. Teste real feito nesta sessão

Página de vendas fictícia local com pixel simulado. O REA identificou, sem executar nada por conta própria:
- 2 scripts (`pixel.js`, `app.js`) com tamanho e hash;
- o endpoint `/api/offer` chamado via `fetch`;
- o link de compra com `utm_source=meta&utm_campaign=teste`;
- as chaves de storage `_fbp` e `lead_id` (sem os valores);
- o log de console `ttq ViewContent`, ou seja, o evento do pixel TikTok.

Limitações observadas: o REA só captura a rede **depois** de anexar à aba (recarregue a página com a observação ativa para pegar o que carrega no início), e não guarda cabeçalhos, cookies nem corpos de resposta.

## 5. Pedidos prontos para o agente (com o MCP instalado)

- "Use o REA para listar todos os pixels e scripts de rastreamento da página https://... e diga quais eventos eles disparam."
- "Analise o bundle desta página e liste os endpoints de API e rotas, com a evidência de cada um."
- "Compare a captura da versão A e da versão B da minha página de vendas e resuma as diferenças."
- "Verifique se meu checkout dispara InitiateCheckout e Purchase e se os UTMs estão sendo repassados."
- "Mostre quais bibliotecas e fornecedores externos a página carrega e o peso de cada script."

## 6. Boas práticas

- Analise **suas** páginas, ou páginas públicas para entender mercado; não copie código ou material protegido de terceiros.
- Mantenha o REA atualizado: `npx rea-agents@latest setup`.
- Em caso de erro `target_not_allowed`: a aba não está na origem informada em `--allowed-origins` (ou a página não carregou, por exemplo bloqueio de rede).
- Rode `npx -y rea-agents@latest doctor` para diagnosticar o ambiente. Hopper/Ghidra/IDA só são necessários para binários nativos, não para sites.
