# Jogo da Velha — UNIFOR

Aplicação web do **Jogo da Velha da Universidade de Fortaleza**, desenvolvida sob a abordagem de
**Desenvolvimento Guiado por Requisitos (Spec-Driven Development)**, tendo como guia soberano a
Especificação de Caso de Uso [`docs/cdu_JogarJogodavelha.md`](docs/cdu_JogarJogodavelha.md).

---

## Identificação

| Campo | Valor |
| --- | --- |
| **Aluno** | Gabriel Pereira Bitencourt Nogueira |
| **Matrícula** | 2520505 |
| **Instituição** | Universidade de Fortaleza (UNIFOR) |
| **Disciplina** | Requisitos e mod de sistemas |
| **Artefato-guia** | CDU *Jogar Jogo da Velha* — v1.0 (08/08/2026) |

---

## 🔗 Aplicação publicada (GitHub Pages)

**https://gabrielbitencourt2006.github.io/jogo-da-velha-unifor/src/index.html**

> Para ativar: **Settings → Pages → Source: Deploy from a branch**, selecionando a branch que contém
> este código e a pasta `/ (root)`. A publicação leva cerca de um minuto na primeira vez.

---

## 📁 Estrutura do repositório

```
jogo-da-velha-unifor/
├── docs/
│   └── cdu_JogarJogodavelha.md    (Especificação de Caso de Uso — artefato fornecido)
├── src/
│   └── index.html                 (Código-fonte completo: HTML + CSS + JS em arquivo único)
├── README.md                      (Este arquivo)
└── RELATORIO_PROMPTS.md           (Relatório da interação com a IA + checklist de aceite)
```

---

## ▶️ Como executar

**Opção 1 — Navegador (recomendado):** acesse o link do GitHub Pages acima.

**Opção 2 — Localmente:** clone o repositório e abra o arquivo diretamente. Não há build,
dependências, `npm install` nem back-end — a aplicação é um único arquivo estático.

```bash
git clone https://github.com/GabrielBitencourt2006/jogo-da-velha-unifor.git
cd jogo-da-velha-unifor
# Abra src/index.html no navegador (duplo clique) ou sirva localmente:
python3 -m http.server 8000   # depois acesse http://localhost:8000/src/index.html
```

> 🔊 **Áudio:** os navegadores só liberam a Web Audio API após a primeira interação do usuário.
> O som passa a tocar a partir do primeiro clique no tabuleiro — comportamento esperado.

---

## 🎮 Funcionalidades

| Recurso | Descrição | Rastreabilidade |
| --- | --- | --- |
| **Modo de Jogo** | *2 Jogadores (PVP)* ou *Contra o Computador* | RF-01 / UI-03 |
| **Formato da Partida** | *Partida Única* ou *Melhor de 3 (MD3)* | RF-02 / UI-04 |
| **Marcação de jogada** | Célula preenchida com `X` ou `O` e desabilitada | RF-03 / UI-09 |
| **Áudio sintetizado** | Tons distintos para X, O, vitória e empate — sem arquivos externos | RF-04 |
| **Detecção de resultado** | Avaliação das 8 combinações vitoriosas a cada movimento | RF-05 / UI-08 |
| **Linha de vitória** | Overlay rotacionado sobre as 3 células vencedoras | RF-06 / UI-10 |
| **Confetes** | Motor de partículas próprio em `<canvas>`, sem biblioteca externa | RF-06 |
| **Placar e rodadas** | Pontuação acumulada e contador `Atual/Total` | RF-07 / UI-05, UI-06, UI-07 |
| **Reinício geral** | Zera placar, rodadas, tabuleiro e devolve o turno ao Jogador X | RF-08 / UI-11 |

### Regras de partida implementadas

- **Partida Única:** a rodada encerra na vitória ou no empate e o tabuleiro fica bloqueado até o
  reinício manual (A1.8).
- **Melhor de 3 (MD3):**
  - Vitória de um jogador → placar incrementado; se alguém atingir **2 vitórias**, é declarado
    campeão definitivo e a partida encerra (A1.7.1).
  - Caso contrário e havendo rodada disponível, o número da rodada é incrementado, o sistema aguarda
    **2 segundos** e limpa o tabuleiro para a próxima rodada (A1.7.2).
  - **Empate** repete a rodada após 2 segundos **sem incrementar** o contador (E1.4).

---

## 🎨 Identidade visual (RNF)

Paleta institucional da UNIFOR aplicada via variáveis CSS:

| Cor | Hex | Uso |
| --- | --- | --- |
| Azul UNIFOR | `#003366` | Título principal, botão Reiniciar, símbolos base |
| Azul Destaque | `#0056b3` | Subtítulo institucional, Jogador X, foco dos seletores |
| Laranja | `#d97706` | Jogador O / CPU, linha de vitória, status final |
| Fundo | `#f4f6f9` | Plano de fundo da aplicação |

---

## 🧱 Requisitos não funcionais atendidos

- **Portabilidade:** execução completa contida em **um único arquivo** `src/index.html`
  (HTML + CSS + JS), sem back-end e sem etapa de build.
- **Zero dependência externa:** nenhum CDN, biblioteca, fonte remota ou arquivo de mídia local.
  Os confetes são renderizados por um motor de partículas próprio em `<canvas>`.
- **Áudio nativo:** todos os efeitos sonoros são sintetizados em tempo real com osciladores da
  **Web Audio API** — não há `.mp3`, `.wav` ou elemento `<audio>` no projeto.
- **Responsividade:** layout fluido com `clamp()` e `aspect-ratio`, adequado a telas de celular e
  desktop.
- **Acessibilidade:** células são `<button>` reais (navegáveis por teclado) com `aria-label`, e o
  status do jogo usa `role="status"` com `aria-live="polite"`.

---

## ✅ Validação

A implementação foi verificada por uma suíte automatizada executada em navegador real (Chromium via
Playwright), cobrindo o fluxo principal, os fluxos alternativos **A1/A2/A3**, o fluxo de exceção
**E1** e os critérios de aceite **CA-01 a CA-07** — **66 verificações, todas aprovadas**.

O detalhamento da auditoria, o registro dos prompts e o checklist de aceite estão em
[`RELATORIO_PROMPTS.md`](RELATORIO_PROMPTS.md).
