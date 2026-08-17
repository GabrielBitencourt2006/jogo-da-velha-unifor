# Relatório de Prompting e Auditoria de Qualidade

**Projeto:** Jogo da Velha — UNIFOR
**Artefato soberano:** `docs/cdu_JogarJogodavelha.md` (CDU *Jogar Jogo da Velha*, v1.0)
**Ferramenta de IA Generativa utilizada:** **Claude (Anthropic)**, através da interface **Claude Code**
**Aluno:** Gabriel Pereira Bitencourt Nogueira — Matrícula 2520505
**Papel do aluno:** Engenheiro de Prompts e Auditor de Qualidade

---

## 1. Metodologia adotada

O desenvolvimento seguiu o ciclo **Spec-Driven Development**: nenhuma funcionalidade foi solicitada
por descrição livre ("faça um jogo da velha bonito"). Todo pedido à IA foi ancorado em um passo
numerado do CDU (P1–P7, A1–A3, E1), em um elemento de interface (UI-01 a UI-11), em um requisito da
matriz de rastreabilidade (RF-01 a RF-08) ou em um critério de aceite (CA-01 a CA-07).

O ciclo aplicado a cada iteração foi:

```
Especificação (CDU) → Prompt ancorado → Geração → Auditoria contra o CDU → Correção ordenada
```

A auditoria não se limitou à leitura do código: foi construída uma **suíte de verificação
automatizada em navegador real** (Chromium via Playwright), com asserções nomeadas segundo os
identificadores do próprio CDU. Isso transformou os critérios de aceite em testes executáveis.

---

## 2. Registro dos prompts enviados

### Prompt 1 — Contextualização e entrega da especificação

> Anexei o enunciado da atividade prática e o arquivo `cdu_JogarJogodavelha.md`. Construa a aplicação
> web do Jogo da Velha da UNIFOR usando a Especificação de Caso de Uso como guia soberano, respeitando
> a estrutura obrigatória do repositório (`docs/`, `src/index.html`, `README.md`,
> `RELATORIO_PROMPTS.md`).

**Intenção do prompt:** estabelecer o CDU — e não o gosto pessoal do desenvolvedor — como fonte única
de verdade, e fixar a árvore de diretórios exigida pela rubrica antes de qualquer linha de código.

**Resultado:** a IA leu a especificação integralmente, criou a estrutura de pastas, copiou o CDU para
`docs/` e gerou `src/index.html` com o estado interno nomeado exatamente conforme a **Seção 18 —
Dicionário de Dados** (`options`, `currentPlayer`, `running`, `winsX`, `winsO`, `currentRound`,
`modeSelect`, `formatSelect`), com comentários citando os passos do CDU.

---

### Prompt 2 — Ordem de auditoria executável

> Valide a lógica do jogo em um navegador real, com asserções nomeadas pelos identificadores do CDU
> (P4.1, A1.7.2, E1.4, CA-01…CA-07).

**Intenção do prompt:** impedir a autoavaliação por inspeção visual ("parece funcionar"), que é a
principal fonte de falso positivo em desenvolvimento assistido por IA.

**Resultado:** foi produzida uma suíte de **66 verificações** cobrindo o fluxo principal, os fluxos
alternativos A1/A2/A3, o fluxo de exceção E1 e os sete critérios de aceite. A suíte revelou os
defeitos descritos na seção 3.

---

### Prompt 3 — Correção da fidelidade textual ao CDU

> Os rótulos da interface estão sem acentuação. O CDU especifica literalmente `Partida Única`
> (UI-04). Corrija os textos visíveis ao usuário.

**Resultado:** os literais foram corrigidos para a grafia exata da especificação.

---

## 3. Erros cometidos pela IA e correções ordenadas

Esta seção registra as divergências reais identificadas na auditoria e a instrução de correção dada.

### Erro 1 — Condição de corrida: jogada da CPU sobrevivendo ao reinício

| | |
| --- | --- |
| **Passo violado** | A2.3 vs. A3.3 / A3.4 |
| **Gravidade** | Alta — corrompia o estado do tabuleiro |

**Descrição:** o fluxo A2.3 exige um intervalo de reflexão de 400 ms antes da jogada do Computador.
A IA implementou esse intervalo com `setTimeout`, mas **sem guardar a referência do temporizador**.
Se o Jogador clicasse em *Reiniciar Jogo* (A3.1) durante essa janela de 400 ms, o temporizador ainda
disparava e a CPU marcava um `'O'` **no tabuleiro recém-limpo**, violando A3.3 (tabuleiro limpo) e
A3.4 (Jogador X como primeiro a jogar).

**Correção ordenada:**

> Guarde a referência do `setTimeout` da CPU em uma variável de escopo e cancele-a em
> `prepararRodada()`, garantindo que A3.3 e A3.4 sejam respeitados mesmo com uma jogada pendente.

**Correção aplicada:** criada a variável `temporizadorCpu`, cancelada via `clearTimeout` sempre que a
rodada é preparada ou a partida reiniciada. O mesmo tratamento já existia para `temporizadorRodada`
(transição de 2 s de A1.7.2 / E1.4).

---

### Erro 2 — Perda de acentuação nos rótulos especificados

| | |
| --- | --- |
| **Elemento violado** | UI-04 (Seletor de Formato) |
| **Gravidade** | Baixa — fidelidade textual ao artefato |

**Descrição:** a IA gerou os textos visíveis sem acentuação (`Partida Unica`, `campeao`, `Celula`),
enquanto o CDU especifica literalmente `Partida Única` na Seção 13.

**Correção ordenada:** reescrever os literais visíveis ao usuário com a grafia exata da
especificação, mantendo `<meta charset="UTF-8">`.

---

### Erro 3 — Lacuna na especificação: fim do MD3 sem 2 vitórias

| | |
| --- | --- |
| **Passo envolvido** | A1.7.1 |
| **Gravidade** | Média — cenário não coberto pelo CDU |

**Descrição:** o passo A1.7.1 só define o desfecho quando **algum jogador atinge 2 vitórias**. Existe,
porém, um cenário alcançável que a especificação não cobre: *X vence a rodada 1, O vence a rodada 2 e
a rodada 3 termina empatada*. Nesse caso ninguém chega a 2 vitórias, mas as 3 rodadas se esgotaram.
A primeira versão gerada deixava a partida em estado indefinido.

**Correção ordenada:**

> Trate o encerramento do MD3 após a 3ª rodada mesmo sem 2 vitórias, declarando o campeão por maior
> número de vitórias ou empate geral, e documente essa decisão como lacuna do CDU.

**Correção aplicada:** implementada a função `mensagemResultadoFinalMD3()`, que compara `winsX` e
`winsO` e declara o campeão da partida — ou empate geral, quando as pontuações são iguais.

> 📌 **Recomendação de melhoria do artefato:** sugere-se incluir no CDU um passo **A1.7.3**
> ("Encerramento do MD3 ao término da 3ª rodada sem 2 vitórias").

---

### Erro 4 — Conflito entre RF-06 e o RNF de Portabilidade

| | |
| --- | --- |
| **Requisitos envolvidos** | RF-06 vs. Seção 10 (Portabilidade) |
| **Gravidade** | Média — decisão arquitetural |

**Descrição:** a matriz de rastreabilidade (RF-06) menciona *"Canvas Confetti"*, nome de uma
biblioteca de terceiros normalmente carregada via CDN. Isso **conflita** com o requisito não funcional
de Portabilidade ("execução completa contida em um único arquivo HTML/CSS/JS") e com a exigência do
enunciado de não haver dependências externas.

**Decisão de auditoria:** o requisito não funcional prevalece, pois *"Canvas Confetti"* descreve o
**efeito desejado**, não uma dependência obrigatória.

**Correção aplicada:** implementado um **motor de partículas próprio** em `<canvas>`
(`dispararConfetes` / `animarConfetes`), com gravidade, arrasto, rotação e esmaecimento, usando a
paleta institucional. O efeito de A1.3 é entregue com **zero dependências externas**.

---

### Erro 5 — Falsos positivos na própria suíte de auditoria

Durante a validação, três asserções falharam **sem que houvesse defeito na aplicação** — registro
mantido por transparência do processo de auditoria:

1. O teste tentava clicar em uma célula já preenchida, mas o navegador recusou o clique porque a
   aplicação **corretamente** desabilita a célula (UI-09). O teste foi reescrito para disparar o
   evento diretamente, provando que **a guarda lógica em JavaScript também barra a jogada** —
   defesa em profundidade para CA-02.
2. A opacidade da linha de vitória era lida no meio da transição CSS de 0,25 s. Adicionada espera.
3. A busca por arquivos `.mp3`/`.wav` casava com o **próprio comentário** do código que documenta a
   ausência deles. A expressão foi restringida a uso real (`<audio>`, `new Audio()`, `src=`).

---

## 4. Decisões de fidelidade ao CDU

| Decisão | Justificativa |
| --- | --- |
| A CPU escolhe uma célula vazia **aleatória**, sem estratégia | O passo A2.4 diz literalmente *"escolhe uma posição vazia no tabuleiro"*. Implementar um minimax invencível **excederia** a especificação e tornaria o CA-05 (MD3) quase inatingível para o jogador. |
| O empate no MD3 **não** incrementa a rodada | Cumprimento literal de E1.4, ainda que isso permita repetir a mesma rodada mais de uma vez. |
| Trocar qualquer seletor zera o placar | Exigido por A3.1 e A3.2 e pelas regras de UI-03 e UI-04, mesmo sendo pouco intuitivo para o usuário. |
| Bloqueio duplo das células (atributo `disabled` + guarda em JS) | Garante CA-02 e CA-03 mesmo diante de eventos sintéticos. |

---

## 5. Checklist de autoavaliação dos Critérios de Aceite

| ID | Critério | Status | Evidência de verificação |
| :-- | :-- | :--: | :-- |
| **CA-01** | **Fidelidade Visual** — paleta `#003366` / `#0056b3` e subtítulo *"UNIVERSIDADE DE FORTALEZA"* | ✅ | `getComputedStyle` confirmou `rgb(0,51,102)` no título, `rgb(0,86,179)` no subtítulo e placar X, `rgb(217,119,6)` no placar O e `rgb(244,246,249)` no fundo. Subtítulo presente em UI-01. |
| **CA-02** | **Regra de Ocupação** — não sobrescrever célula preenchida | ✅ | Clique forçado (evento sintético) em célula ocupada: símbolo preservado e turno inalterado. Célula também fica `disabled`. |
| **CA-03** | **Bloqueio pós-Fim de Jogo** | ✅ | Após vitória, empate e definição de campeão do MD3, cliques em células vazias não alteram o tabuleiro (`running == false` + células desabilitadas). |
| **CA-04** | **Comportamento do Modo CPU** | ✅ | No modo `cpu`, o status muda para *"Vez do Computador"*, nenhuma jogada ocorre antes dos 400 ms e um único `'O'` é marcado após a pausa. Partida completa contra a CPU concluída com desfecho coerente. |
| **CA-05** | **Regra do Melhor de 3** | ✅ | Contador exibe `1/3`; após vitória a rodada vai a `2/3` e o tabuleiro é limpo em 2 s; empate mantém `2/3` (E1.4); ao atingir 2 vitórias o campeão é declarado e nenhuma nova rodada inicia. |
| **CA-06** | **Efeitos Visuais de Vitória** | ✅ | Comprimento da linha medido em **246,7 px** contra distância real de **246,7 px** entre os centros das células vitoriosas (desvio < 3 px), com centralização vertical verificada. Confetes disparados no `<canvas>`. |
| **CA-07** | **Autonomia de Áudio** | ✅ | Nenhuma tag `<audio>`, `new Audio()` ou referência a `.mp3`/`.wav`/`.ogg` no projeto. `AudioContext` instanciado e som gerado por osciladores (`tocarTom`, `tocarSomVitoria`, `tocarSomEmpate`). |

**Resultado consolidado da suíte automatizada: 66 verificações executadas, 66 aprovadas, 0 falhas,
0 erros de JavaScript em console.**

---

## 6. Cobertura dos fluxos do CDU

| Fluxo | Descrição | Status | Implementação |
| :-- | :-- | :--: | :-- |
| **P1–P7** | Fluxo principal | ✅ | `prepararRodada()`, `aoClicarCelula()`, `executarJogada()`, `avaliarTabuleiro()` |
| **A1** | Fim de rodada por vitória | ✅ | `finalizarRodadaPorVitoria()` — linha, confetes, acorde, placar, status |
| **A1.7** | Regras do MD3 | ✅ | Campeão com 2 vitórias, avanço de rodada com pausa de 2 s |
| **A2** | Jogada do Computador | ✅ | `jogadaDoComputador()` — bloqueio, pausa de 400 ms, escolha e execução |
| **A3** | Reinício / alteração de parâmetros | ✅ | `reiniciarPartida()` ligada a UI-11, UI-03 e UI-04 |
| **E1** | Fim de rodada por empate | ✅ | `finalizarRodadaPorEmpate()` — som descendente e *"Rodada Empatada!"* |

---

## 7. Conclusão

O uso de IA Generativa foi produtivo, mas **não autônomo**. A ferramenta acelerou a escrita do código
e a construção da suíte de testes, porém introduziu um defeito de concorrência real (Erro 1) e
divergências de fidelidade textual (Erro 2), além de não resolver por conta própria uma lacuna
(Erro 3) e um conflito interno (Erro 4) da própria especificação.

A conclusão da auditoria é que **o valor do processo esteve na especificação e na verificação, não na
geração**: foi a leitura rigorosa do CDU que permitiu formular prompts ancorados em passos numerados,
e foi a suíte automatizada — derivada dos critérios de aceite — que expôs objetivamente as
divergências. O CDU funcionou simultaneamente como fonte de requisitos, roteiro de testes e critério
de aceitação final.
