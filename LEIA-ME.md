# Frota Jornada

Sistema de lançamento e acompanhamento em tempo real da jornada do motorista,
com registro automático em Google Sheets.

Três peças:
- **Code.gs** — backend (Google Apps Script), a "API" que lê e grava na planilha.
- **index.html** — app do motorista (celular), preenchimento progressivo do dia.
- **painel.html** — painel de acompanhamento (celular ou PC) para auxiliar,
  encarregado e supervisão, atualizando sozinho a cada 15 segundos.

---

## Passo 1 — Criar a planilha e o backend

1. Crie uma planilha nova no Google Sheets (pode ficar em branco).
2. Menu **Extensões > Apps Script**.
3. Apague o conteúdo padrão do `Código.gs` e cole o conteúdo do arquivo **Code.gs**.
4. No topo do editor, escolha a função `setup` e clique em **Executar** (ícone ▶).
   - Na primeira vez, o Google vai pedir autorização — aceite (é a sua própria conta).
   - Isso cria automaticamente 3 abas na planilha: `Lancamentos`, `Paradas` e `Placas`.
5. Vá na aba **Placas** da planilha e cole a lista de veículos, no formato:

   | OP | FT | PLACA |
   |----|----|-------|
   | LS | 063 | LBW 8156 |
   | GR | 064 | LUV 9296 |
   | CV | 097 | LRX 4831 |

   A coluna OP aqui é só para agrupar/organizar a lista de placas (aparece como
   categoria no seletor) — **não** filtra pela operação do dia. Operação (LS/TL/HT/CV)
   e Placa são escolhidas de forma independente pelo motorista.

   Se preferir não digitar linha por linha: no Apps Script, depois de rodar `setup`,
   rode também a função `seedPlacas` — ela já vem com a lista do seu cadastro atual
   pronta no código.

6. Volte no editor do Apps Script: **Implantar > Nova implantação**.
   - Tipo: **App da Web**.
   - Executar como: **Eu** (sua conta).
   - Quem pode acessar: **Qualquer pessoa**.
   - Clique em **Implantar** e copie a **URL do app da Web** gerada.

---

## Passo 2 — Ligar o app do motorista e o painel ao backend

Abra **index.html** e **painel.html** e, no topo de cada um, troque:

```js
const CONFIG = {
  API_URL: 'COLE_AQUI_A_URL_DO_APPS_SCRIPT_WEB_APP'
};
```

pela URL que você copiou no passo anterior. Salve os dois arquivos.

---

## Passo 3 — Publicar (mesmo padrão do Frota Field)

Sugestão: subir os dois arquivos num repositório do GitHub Pages, do mesmo jeito
que o `frota-field` já está publicado. Por exemplo:

```
juanilgeral.github.io/frota-jornada/           → index.html (motorista)
juanilgeral.github.io/frota-jornada/painel.html → painel (acompanhamento)
```

Assim o motorista acessa pelo celular, e o auxiliar/encarregado/supervisão acessa
o painel tanto pelo celular quanto pelo PC — tudo lendo a mesma planilha em tempo real.

---

## Como funciona o app do motorista

O motorista preenche em etapas, e cada etapa já grava direto na planilha
(não precisa preencher tudo de uma vez):

1. **Início da jornada** — data, hora do ponto, hora programada de saída,
   operação (LS/TL/HT/CV) ou **OUTRO**, placa (lista do cadastro de frota),
   destino e km de saída. Ao escolher **OUTRO**, o motorista informa o nome da
   operação ou descreve o serviço; esse texto é salvo no campo `Operacao` do
   lançamento.
2. **Lojas** — para cada loja: chegada, abertura do baú, fechamento do baú e saída.
   O total de tempo na loja é calculado sozinho. Se a carga for compartilhada,
   basta tocar em **"+ Adicionar próxima loja"** para lançar a 2ª, 3ª loja etc.,
   sempre com os mesmos campos.
3. **Retorno ao CD** — hora de chegada no CD, data de chegada do motorista e
   km de chegada. Ao salvar, o sistema calcula sozinho:
   - Total de Km rodado (km chegada − km saída)
   - Total de horas do motorista (do início do ponto até a chegada no CD)
   - Total de horas nas lojas (soma do tempo parado em cada loja)

O aparelho do motorista guarda o lançamento em andamento — se ele fechar o
app e abrir de novo, continua de onde parou. Ao final, dá para tocar em
"Encerrar e iniciar novo lançamento" para começar o próximo dia.

## Como funciona o painel

O painel mostra todos os lançamentos do dia (com filtro por data, status e
operação), com um resumo geral (quantos em andamento, quantos concluídos,
km total) e um cartão por veículo mostrando a situação atual — ex: "Na loja 2",
"Retornando ao CD". Tocando no cartão, abre o detalhe completo com todos os
horários e os totais calculados. A tela se atualiza sozinha a cada 15 segundos,
sem precisar recarregar a página.

## Observações

- Todos os dados ficam gravados diretamente na planilha (abas Lancamentos e Paradas) — dá pra abrir a planilha a qualquer momento
  para conferir, exportar ou criar relatórios extras.
- Não foi criado login neste primeiro modelo — se quiser um controle de acesso
  por motorista/usuário como o Frota Field já tem, dá pra integrar depois.
- Os cálculos de horário tratam virada de dia (ex: chegar 00:20 depois de sair
  às 22:00).
