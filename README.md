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

## RODADA 5: A fábrica que devolve null (01h30)

**Tipo:** rodada · **Voto:** A

**Justificativa:** A `ContaFactory` retornava `null` para tipos de conta não reconhecidos, fazendo o erro aparecer apenas na camada de persistência. Optamos pelo princípio Fail-fast, lançando uma exceção clara diretamente na Factory para identificar a causa do problema e evitar a propagação de valores nulos. Embora essa abordagem não implemente automaticamente o suporte à Conta Universitária, ela centraliza a validação e facilita a manutenção do sistema.

**Placar do grupo após esta decisão:** 🔥 8 · 💰 R$ 70 mil · 🧹 -1


## EVENTO RELÂMPAGO C: O cadastro com metade dos campos (02h20)

**Tipo:** relâmpago · **Voto:** B

**Justificativa:** O cadastro simplificado exigia criar objetos com menos campos, enquanto o construtor atual obrigava o preenchimento de todos os atributos. Optamos pelo padrão Builder, que centraliza valores padrão e validações no método `build()`, evitando o crescimento de construtores sobrecarregados. Embora exija uma estrutura inicial mais elaborada, essa abordagem melhora a manutenção e reduz a propagação de valores nulos.

**Placar do grupo após esta decisão:** 🔥 8 · 💰 R$ 70 mil · 🧹 -1

## RODADA 6: O teste vermelho às 03h45

**Tipo:** rodada · **Voto:** B

**Justificativa:** O teste JUnit identificou uma divergência entre a taxa TED implementada no serviço (R$ 50,00) e a regra de negócio estabelecida no contrato (R$ 5,00). Optamos por investigar a falha e preservar a validação com `assertEquals`, em vez de desabilitar ou modificar o teste para mascarar o erro. Embora essa decisão nos faça perder a janela de deploy e aumente o prejuízo imediato, ela permite identificar a causa real e evita comprometer a confiabilidade da suíte de testes.

**Placar do grupo após esta decisão:** 🔥 7 · 💰 R$ 100 mil · 🧹 -2

## RODADA 7: Go/No-Go da Black Friday (06h15)

**Tipo:** rodada · **Voto:** B

**Justificativa:** Os incidentes da madrugada evidenciaram a concentração excessiva de responsabilidades no `TransferenciaService`, indicando baixa coesão e possíveis violações do Princípio da Responsabilidade Única (SRP). Optamos por corrigir os problemas identificados e validar as alterações com a suíte de testes antes da abertura da campanha. Embora essa decisão atrase o lançamento em duas horas e aumente o prejuízo imediato, ela prioriza a estabilidade, a confiabilidade das operações bancárias e a redução da dívida técnica.

**Placar do grupo após esta decisão:** 🔥 8 · 💰 R$ 200 mil · 🧹 -3

---

## 🔎 O caminho do MEU grupo (preencher na 3ª aula, quando o mapa for revelado)

Uma linha por decisão registrada acima (usem os títulos ditados em aula):

| # | Decisão | Nossa letra | Consequência que ELA teria tido |
|---|---|---|---|
| 1 | R1 — O saldo que virou negativo | C | Corrigimos o tratamento da exceção e adicionamos um teste com `assertThrows`. A solução demorou mais, afetando oito clientes e gerando R$ 35 mil de prejuízo, mas reduziu a dívida técnica. |
| 2 | R2 — A injeção que derreteu o banco | D | Utilizamos a consulta derivada `findByTitular` do Spring Data JPA, eliminando a concatenação insegura de SQL e reduzindo a dívida técnica. |
| 3 | R3 — A noite das contas gêmeas | D | Delegamos a geração dos números de conta a uma `SEQUENCE` do Oracle, garantindo unicidade na geração. A migração levou aproximadamente uma hora e gerou R$ 10 mil de prejuízo. |
| 4 | R4 — O comprovante que aponta pra pessoa errada | B | Corrigimos a gravação e reprocessamos os extratos antigos, restaurando a integridade dos registros. O processo gerou R$ 25 mil de prejuízo e reduziu a dívida técnica. |
| 5 | R5 — A fábrica que devolve null | A | Aplicamos Fail-fast na Factory, substituindo o retorno `null` por uma exceção clara. A reputação caiu um ponto, mas o erro passou a ser identificado próximo à sua origem. |
| 6 | Relâmpago C — O cadastro com metade dos campos | B | Escolhemos o padrão Builder para centralizar valores padrão e validações, evitando construtores sobrecarregados e sem aumentar a dívida técnica. |
| 7 | R6 — O teste vermelho às 03h45 | B | Investigamos a divergência da taxa TED, preservando o teste correto. Perdemos a janela de deploy, gerando R$ 30 mil de prejuízo, mas identificamos a cobrança incorreta. |
| 8 | R7 — Go/No-Go da Black Friday | B | Decidimos corrigir os problemas e abrir a campanha duas horas depois, com a suíte saudável. A decisão gerou R$ 100 mil de prejuízo adicional, mas aumentou a reputação e reduziu a dívida técnica. |

### Placar final da equipe

- **Reputação:** 🔥 8
- **Prejuízo acumulado:** 💰 R$ 200 mil
- **Dívida técnica:** 🧹 -3
- **Estado da suíte:** Saudável
- **Perfil final:** A Noite dos Heróis

O grupo encerrou a madrugada com reputação elevada e dívida técnica reduzida. Embora as decisões tenham gerado prejuízos imediatos, priorizamos a integridade dos dados, a segurança das operações e a confiabilidade do sistema para a Black Friday.

---

# 📋 PÓS-MORTEM · Relatório de Incidente

**Equipe:** Plantão Java  
**Turma:** 2CCPH  
**Sistema:** FiapBank — Black Friday  
**Tecnologias envolvidas:** Java 17, Spring Boot, Spring Data JPA, Oracle e JUnit.

**Placar final da simulação:** 🔥 8 · 💰 R$ 200 mil · 🧹 -3  
**Estado da suíte:** Saudável  
**Perfil final:** A Noite dos Heróis

## 1. Linha do tempo da madrugada

Durante o plantão simulado, nossa equipe enfrentou incidentes relacionados ao tratamento de exceções, segurança das consultas, persistência, padrões de projeto e testes automatizados.

| Horário | Incidente | Decisão e consequência |
|---|---|---|
| 19h12 | R1 — O saldo que virou negativo | **C:** Corrigir a exceção ignorada e adicionar um teste com `assertThrows`. Oito clientes foram afetados, com R$ 35 mil de prejuízo, mas reduzimos o risco de regressão. |
| 21h02 | R2 — A injeção que derreteu o banco | **D:** Utilizar a consulta derivada do Spring Data JPA, eliminando a concatenação insegura de SQL e reduzindo a dívida técnica. |
| 22h50 | R3 — A noite das contas gêmeas | **D:** Delegar a geração dos números de conta a uma `SEQUENCE` do Oracle. A migração gerou R$ 10 mil de prejuízo. |
| 00h47 | R4 — O comprovante que aponta pra pessoa errada | **B:** Corrigir a gravação e reprocessar os extratos anteriores, restaurando a integridade dos registros ao custo de R$ 25 mil. |
| 01h30 | R5 — A fábrica que devolve null | **A:** Aplicar Fail-fast na Factory, substituindo o retorno nulo por uma exceção clara. A reputação caiu um ponto. |
| 02h20 | Relâmpago C — O cadastro com metade dos campos | **B:** Utilizar Builder para centralizar valores padrão e validações, sem aumentar a dívida técnica. |
| 03h45 | R6 — O teste vermelho | **B:** Investigar a divergência entre a taxa TED de R$ 5 prevista em contrato e os R$ 50 implementados. A investigação custou a janela de deploy e R$ 30 mil. |
| 06h15 | R7 — Go/No-Go da Black Friday | **B:** Corrigir os problemas identificados, validar a suíte e abrir a campanha às 10h, duas horas depois do previsto, com prejuízo adicional de R$ 100 mil. |

Os eventos relâmpago A, B, D e E não se aplicaram ao caminho escolhido pela equipe.

Ao final da madrugada, o grupo acumulou R$ 200 mil de prejuízo, reputação 8 e dívida técnica -3, de acordo com as consequências da simulação. O perfil alcançado foi **A Noite dos Heróis**.

## 2. Causa raiz de 2 incidentes (aula + mecanismo técnico)

### Incidente 1: R1 — O saldo que virou negativo

**Aula 11 — Tratamento de exceções**

A causa raiz foi a captura indevida da `SaldoInsuficienteException` dentro do próprio método de transferência. Embora a exceção fosse lançada quando o saldo era insuficiente, o bloco `catch` a ignorava, permitindo que o método continuasse executando o débito.

Esse comportamento violava a regra de negócio que proíbe saldos negativos.

A decisão foi corrigir o tratamento da exceção e adicionar um teste com `assertThrows`, verificando que transferências sem saldo suficiente devem ser recusadas.

A principal lição foi que lançar uma exceção não garante a proteção de uma regra de negócio quando seu tratamento permite que a execução continue incorretamente.

### Incidente 2: R6 — O teste vermelho às 03h45

**Aula 15 — Testes unitários e validação de regras de negócio**

A causa raiz foi a implementação incorreta da taxa TED no serviço de transferências. Enquanto o contrato estabelecia uma cobrança fixa de R$ 5,00, o código utilizava o valor de R$ 50,00, definido diretamente no serviço.

O teste JUnit identificou corretamente a divergência por meio de `assertEquals`, comparando o resultado esperado com o valor efetivamente calculado.

A equipe decidiu investigar a falha, mesmo perdendo a janela de deploy. Desabilitar o teste com `@Disabled` ou modificar seu valor esperado apenas esconderia o defeito.

Como prevenção, recomendamos substituir valores numéricos isolados por constantes de negócio e manter testes automatizados que validem as taxas previstas no contrato.

A principal lição foi que uma suíte de testes confiável deve verificar as regras de negócio, e não simplesmente confirmar o comportamento atual do código.

## 3. O que faríamos diferente (2 rodadas + por quê)

### Rodada 3 — A noite das contas gêmeas

**Decisão original:** D — Delegar a numeração ao banco de dados.

Embora a utilização de uma `SEQUENCE` no Oracle seja uma solução robusta para gerar identificadores únicos, nossa abordagem poderia ter considerado melhor a recuperação dos registros já afetados.

Em uma situação real, combinaríamos a migração com uma auditoria das contas duplicadas, identificando os clientes impactados e corrigindo eventuais inconsistências nos dados existentes.

Também avaliaríamos uma contenção imediata para impedir novas duplicações enquanto a migração definitiva estivesse em andamento.

Isso permitiria tratar tanto a causa do incidente quanto os danos produzidos antes da correção.

### Rodada 5 — A fábrica que devolve null

**Decisão original:** A — Aplicar Fail-fast na Factory.

A escolha melhorou o diagnóstico dos erros, pois substituiu o retorno `null` por uma exceção clara. Entretanto, ela não resolveu completamente a necessidade do negócio: permitir a abertura de contas universitárias.

Em uma implementação real, complementaríamos a correção adicionando suporte explícito ao tipo `UNIVERSITARIA` na Factory, juntamente com testes unitários e de integração para validar a criação e a persistência da nova conta.

Dessa forma, além de evitar erros silenciosos, garantiríamos que a funcionalidade prometida aos clientes estivesse disponível e funcionando corretamente.

A reflexão mostrou que corrigir o tratamento de um erro não significa necessariamente resolver a necessidade que originou o incidente.

## 4. A maior lição da equipe

A principal lição do Plantão Java foi que **a estabilidade de um sistema depende de decisões técnicas fundamentadas, e não apenas da velocidade das correções**.

Durante a madrugada, percebemos como problemas aparentemente pequenos, como uma exceção ignorada, uma consulta SQL concatenada ou um teste desabilitado, podem comprometer a segurança, a integridade dos dados e a confiança dos clientes.

Nossas escolhas priorizaram o tratamento das causas, a confiabilidade dos testes e a redução da dívida técnica, mesmo quando isso representou atrasos e prejuízos imediatos.

Também aprendemos que decisões tecnicamente adequadas podem ser incompletas se não considerarem os dados já afetados e as necessidades do negócio.

Encerramos a simulação com o perfil **A Noite dos Heróis**, reconhecendo que nossa principal responsabilidade como equipe de desenvolvimento é construir sistemas seguros, testáveis e confiáveis.

**Uma palavra que resume nossa madrugada: CONFIABILIDADE.
