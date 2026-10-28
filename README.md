# FICHA DE DECISÕES · War Room FiapBank (CP6 · 3 aulas)

> Este arquivo é o **README.md do repositório do grupo** (`cp6-warroom-<nome-do-grupo>`).
> Vale **5,0 pontos** (rodadas 0,5 · relâmpagos 0,3), e a nota é pela
> **justificativa**, não pela letra. Preencham após cada aula e commitem até
> **23h59 do mesmo dia** (regras completas na seção 5 do enunciado).
>
> **Os incidentes da madrugada são revelados só em aula.** O título de cada registro
> será **ditado pelo professor na hora**; ninguém se antecipe.

**Grupo (nome da equipe plantonista):** Plantão Java

**Turma:** 2CCPH | **Repo:** `cp6-warroom-plantao-java`

**Integrantes (nome + RM):**

| Nome | RM |
|---|---|
| Amom Ianaguivara Brito|565718 |
| Fernando Antônio|565363 |
|Victor Chen |562549 |

## 0. Setup do repositório (antes da 1ª aula; podem apagar esta seção depois)

1. Um integrante cria o repo **público** no GitHub: `cp6-warroom-<nome-do-grupo>`
   (ex.: `cp6-warroom-debugadores`), com um README qualquer
2. Substituam o conteúdo do `README.md` por este template (no navegador, pelo próprio
   GitHub, ou clonando):
   ```bash
   git clone https://github.com/<conta>/cp6-warroom-<nome-do-grupo>.git
   cd cp6-warroom-<nome-do-grupo>
   # substitua o conteúdo do README.md por este template e:
   git add .
   git commit -m "chore: ficha em branco do grupo"
   git push
   ```
3. Postem o **link no Teams** (o mesmo link para todo o grupo)

**Convenção de commits** (1 commit por rodada; relâmpagos podem ir junto com a
rodada seguinte):

```
decisao: R1 - opcao C (<resumo da justificativa em uma frase>)
decisao: R2 - opcao A (<resumo em uma frase>)
pos-mortem: relatorio de incidente da madrugada
```

---

## 📁 Dossiê técnico do FiapBank (MVP em produção)

**Stack:** Java 17 + Spring Boot + Spring Data JPA + Oracle. API com endpoints em
`/api/contas` e `/api/transferencias` (cenário visto desde a Aula 13).

**Contrato e regras de negócio que o banco prometeu aos clientes e aos reguladores:**

| Regra | Como deve ser |
|---|---|
| Transferência **PIX** | taxa **R$ 0,00** |
| Transferência **TED** | taxa fixa **R$ 5,00** |
| Saldo | **nunca fica negativo**: transferência/saque sem saldo é recusado com erro claro |
| Número de conta | **sequencial e único** (1001, 1002, 1003...), gerado pelo sistema |
| Extrato de transferência | grava **quem pagou** e **quem recebeu**, na ordem certa |
| CPF | **dado sensível**: nunca aparece nas respostas da API |
| Consultas ao banco | sempre parametrizadas, e cada operação **usa e libera** a conexão |
| Suíte de testes | roda antes de todo deploy; **verde** é pré-requisito pra subir |

**Como escrever a justificativa:** nomeie o **mecanismo técnico** em jogo (o
conceito das Aulas 11 a 15 que explica o incidente) e o **trade-off** (velocidade ×
segurança × faturamento × dívida). "Porque é mais seguro" não é justificativa.

---

# 📝 REGISTRO DE DECISÕES

> A cada incidente, o professor dita o título (ex.: "Rodada 1"). **Copiem o modelo
> abaixo, colem no fim desta seção** e preencham com o rascunho feito em aula, junto
> com o **placar do grupo** após a consequência. Commitem 1 commit por rodada até
> 23h59 do dia.
>
> **Eventos relâmpago:** registrem apenas se o grupo for **afetado** (o professor
> chama quem for; não ser chamado é bom sinal).

**Modelo (copiar para cada decisão):**

```
## <título ditado pelo professor>

**Tipo:** ( rodada / relâmpago ) · **Voto:** ( A / B / C / D )

**Justificativa:**

<2 a 3 linhas: mecanismo técnico + trade-off>

**Placar do grupo após esta decisão:** 🔥 __ · 💰 R$ __ mil · 🧹 __
```

*(as decisões entram aqui, na ordem em que a madrugada as trouxer; placar inicial:
🔥 7 · 💰 0 · 🧹 2)*

## RODADA 1: O saldo que virou negativo (19h12)

**Tipo:** rodada · **Voto:** C

**Justificativa:** A exceção `SaldoInsuficienteException` estava sendo capturada e ignorada pelo bloco `catch`, permitindo que a transferência fosse realizada mesmo sem saldo suficiente. Decidimos corrigir o tratamento da exceção e adicionar um teste com `assertThrows` para impedir regressões. Embora essa abordagem aumente o tempo de correção e o prejuízo imediato, ela protege a regra de negócio e reduz a dívida técnica.

**Placar do grupo após esta decisão:** 🔥 8 · 💰 R$ 35 mil · 🧹 1


## RODADA 2: A injeção que derreteu o banco (21h02)

**Tipo:** rodada · **Voto:** D

**Justificativa:** A consulta JDBC concatenava diretamente a entrada do usuário ao SQL, permitindo ataques de SQL Injection. Optamos por utilizar o método `findByTitular` do Spring Data JPA, que realiza consultas com parâmetros vinculados e elimina a concatenação insegura. Embora essa solução leve cerca de 50 minutos, sendo mais demorada que o `PreparedStatement`, ela aproveita o repositório existente e reduz a dívida técnica.

**Placar do grupo após esta decisão:** 🔥 8 · 💰 R$ 35 mil · 🧹 0

## RODADA 3: A noite das contas gêmeas (22h50)

**Tipo:** rodada · **Voto:** D

**Justificativa:** O método `getInstancia()` criava uma nova instância do gerador a cada chamada, violando o padrão Singleton e permitindo números de conta duplicados. Optamos por delegar a geração dos identificadores a uma `SEQUENCE` do banco Oracle, garantindo unicidade mesmo com múltiplas instâncias da aplicação. Embora a migração exija aproximadamente 1 hora, essa abordagem oferece maior robustez e preserva a regra de negócio.

**Placar do grupo após esta decisão:** 🔥 8 · 💰 R$ 45 mil · 🧹 0

## RODADA 4: O comprovante que aponta pra pessoa errada (00h47)

**Tipo:** rodada · **Voto:** B

**Justificativa:** A entidade `Transferencia` estava sendo instanciada com os números de origem e destino invertidos, comprometendo a integridade dos registros persistidos. Optamos por corrigir a gravação e reprocessar os extratos antigos para restaurar a consistência dos dados e permitir auditorias confiáveis. Embora o reprocessamento leve aproximadamente duas horas e aumente o prejuízo imediato, ele evita manter informações incorretas no histórico e reduz a dívida técnica.

**Placar do grupo após esta decisão:** 🔥 9 · 💰 R$ 70 mil · 🧹 -1

---

## 🔎 O caminho do MEU grupo (preencher na 3ª aula, quando o mapa for revelado)

Uma linha por decisão registrada acima (usem os títulos ditados em aula):

| # | Decisão | Nossa letra | Consequência que ELA teria tido |
|---|---|---|---|
| | | | |
| | | | |
| | | | |

*(adicionem linhas conforme as decisões da madrugada)*

---

# 📋 PÓS-MORTEM · relatório de incidente (montar em sala na 3ª aula)

> Rascunho em aula e commit final `pos-mortem:` até 23h59 do dia da 3ª aula.

## 1. Linha do tempo da madrugada

_______________________________________________________________________________________

_______________________________________________________________________________________

_______________________________________________________________________________________

## 2. Causa raiz de 2 incidentes (aula + mecanismo técnico)

**Incidente 1:** _______________________________________________

Aula/mecanismo:

_______________________________________________________________________________________

**Incidente 2:** _______________________________________________

Aula/mecanismo:

_______________________________________________________________________________________

## 3. O que faríamos diferente (2 rodadas + por quê)

_______________________________________________________________________________________

_______________________________________________________________________________________

## 4. A maior lição da equipe

_______________________________________________________________________________________
