# Checkpoint 5 — Bug Hunt PetFiap

> Copie este arquivo para a raiz do seu repositório com o nome **README.md**
> e preencha todas as seções.

## Identificação

**Grupo:** Equipe11

| Integrante | RM | Turma |
|---|---|---|
| Auro Vanetti | RM563761 | 2CCPH |
| Renan Mano Otero | RM554911 | 2CCPH |
| Marco Antonio Ferreira Fonseca | RM566434 | 2CCPH |
| Bruno Soares de Santanna | RM562235 | 2CCPH |
| Enzo Yokokura Araujo | RM564177 | 2CCPH |

| Campo | |
|---|---|
| **Total de bugs corrigidos** | 10 / 12 |
| **Total de ajustes de Clean Code** | 6 + 1 extra / 6 |
| **Total de testes novos escritos** | 6 / 6 |
| **Suíte final (Run As → JUnit Test)** | ___ testes, ___ falhas |

---

## Parte 1 — Bugs encontrados

> Uma linha por bug, na ordem em que você os encontrou. Use a numeração dos seus
> commits (`fix: bug01 ...`). Preencha TODAS as colunas — metade da nota está aqui.

| # | Sintoma observado (o que fiz/vi) | Causa raiz (arquivo e linha aproximada) | Correção aplicada | Conceito da disciplina |
|---|---|---|---|---|
| bug01 | Falta da indicação da variável petNome em AtendimentoBuilder.classe no método "comPet". | Linha 24 `AtendimentoBuilder.java`. | Adicionar `this.` a variável.| Revisão - Exercício ENADE: Questão 3 |
| bug02 | O Builder aceitava a criação de um atendimento sem nome ou sem porte. | `AtendimentoBuilder.java` no método `construir()`. Não havia validação do nome do pet nem mesmo do porte antes de chamar a Factory. | Adicionamos validações fail test e lançamento de IllegalArgumentException. | Exceções/Validação/Builder |
| bug03 | Deveria criar objeto do tipo `Tosa` ao invés do tipo `Banho`. | `AtendimentoFactory`. O case `Tosa` instanciava `Banho`. | Substituição para case `Tosa` -> new `Tosa(...)`. | Factory/Polimorfismo |
| bug04 | Criava consulta veterinária com dados vazios/não preenchidos.| Em `ConsultaVeterinaria.java` os parâmetros recebidos não eram utilizados pelo `super()`.| Modificado para super(protocolo, petNome, petPorte, tutorNome, dataHora). | Herança/Construtores |
| bug05 | `getInstancia()` retornava valores diferentes. | `GeradorProtocolo.java`, `método getInstancia()`. Singleton criava new `GeradorProtocolo()` sem guardar em instancia. | A instância agora passa a ser atribuída a instância antes de ser retornada. | Singleton |
| bug06 | Conflito de horário não era detectado. |`AgendaService.java`no método `agendar()` comparava `a.getPetNome()` e `a.getDataHora()` com `==`. | Trocar `==` por `.equals()` nas duas comparações. | Igualdade de objetos |
| bug07 | A Busca por um ID inexistente retornava `null`. | `AgendaService.java`no método `buscarPorId()` capturava qualquer exceção e a engolia. | Removido o `try/catch` genérico, `AtendimentoNaoEncontradoException` passa a propagar normalmente. | Exceções específicas |
| bug08 |`Banho.calcularPreco()` estava com os valores de PEQUENO e GRANDE invertidos. | `Banho.java` no método `calcularPreco()` retornava R$100 para o PEQUENO e R$60 no GRANDE. | Corrigidos para seus valores originais de acordo com a regra de negócio. | Polimorfismo / regra de negócio  |
| bug09 | Tosa retornava duração herdada de 30 min. | Método `getDuracaoMinutos(String)` era sobrecarga, não sobrescrita. | Alterado para `getDuracaoMinutos()` com `@Override`. | Override / overload |
| bug10 | Era possível agendar atendimento no passado. | `AgendaService.agendar` não validava data/hora antes do repositório. | Adicionada validação com `IllegalArgumentException` antes de qualquer consulta. | Fail Fast / regra de negócio |
| bug11 | Atendimento concluído ou cancelado podia ser cancelado novamente. | `Atendimento.cancelar` mudava o status sem validar o estado atual. | Agora só `AGENDADO` pode virar `CANCELADO`; demais lançam `StatusInvalidoException`. | Máquina de estados / exceções |
| bug12 | | | | |

## Parte 2 — Ajustes de Clean Code

| # | Onde estava | Qual princípio/boas práticas era violado | O que eu mudei |
|---|---|---|---|
| clean01 | `AtendimentoFactory`, no método `criar()` | O nome das variáveis dificultavam o seu entendimento. | Renomeamos as variáveis `p`, `t`, `n`, `po`, `tu` e `d` para `protocolo`, `tipo`, `petNome`, `petPorte`, `tutorNome` e `dataHora`. |
| clean02 | `AgendaService.agendar()` | `System.out.println` dentro da camada de serviço, misturando regra de negócio com saída, não agregando nada ao mesmo. | Removemos o `System.out.println` do "recibo" |
| clean03 | `GeradorProtocolo` | `System.out.println` de debug perdido dentro de código de produção.  | Removemos o `System.out.println("GeradorProtocolo criado!")` |
| clean04 | `AtendimentoController` | Field injection escondia dependência e dificultava testes. | Trocamos `@Autowired` em campo por injeção via construtor.  |
| clean05 | `AtendimentoController` | Código morto: método privado que nunca é chamado em lugar nenhum do sistema.| Removemos o método  do controller. |
| clean06 |`AgendaService`| Mesmo problema de field injection. | Trocamos o mesmo por injeção via construtor. |
| clean07 (extra) |GeradorProtocolo.getInstancia| No comentário da classe afirma que é "thread-safe", mas o método não tem sincronização. | Adicionamos synchronized no método getInstancia().|

## Parte 3 — Testes novos (regras que estavam sem cobertura)

> Uma linha por teste novo (`test: ...`). "Regra coberta" é o comportamento do
> contrato (seção 3 do enunciado) que o teste protege. Em "Resultado", diga se o
> teste ficou vermelho ao ser escrito (revelou bug — qual?) ou verde de cara
> (regra já estava correta).

| # | Teste escrito (classe.método) | Regra coberta | Resultado ao escrever (vermelho/verde) |
|---|---|---|---|
| teste01 | `BanhoTest.deveCalcularPrecoDoBanhoConformePorte` | Banho custa 60/80/100 para pequeno/médio/grande. | Verde — já haviamos corrigido esse problema no `bug08`, rodamos para confirmar se está correto. |
| teste02 | `TosaTest.deveDurar60Minutos` | Tosa dura 60 minutos. | Vermelho — nos revelou o `bug09`. |
| teste03 | `ConsultaVeterinariaTest.deveCustar150ReaisIndependentementeDoPorte` | Consulta custa R$150 para qualquer porte. | Verde — regra já estava correta. |
| teste04 | `AgendaServiceTest.deveRecusarAgendamentoNoPassadoSemConsultarBanco` | Data/hora no passado deve falhar antes de acessar o banco. | Vermelho — revelou bug10. |
| teste05 | `AgendaServiceTest.deveCancelarAtendimentoAgendado` | Atendimento AGENDADO pode ser cancelado. | Verde — regra já estava correta. |
| teste06 | `AgendaServiceTest.deveRecusarCancelamentoQuandoAtendimentoNaoEstaAgendado` | CONCLUIDO e CANCELADO não podem ser cancelados. | Vermelho — revelou bug11. |

---

## Parte 4 — Perguntas de reflexão

> Responda com suas palavras, 5 a 10 linhas cada, **usando o código real do
> projeto como exemplo**. Respostas genéricas de tutorial não pontuam.

### 1. A suíte como contrato (Aula 15)
O projeto chegou com 20 testes, 9 vermelhos. Descreva como você usou as
mensagens de falha (ex.: `expected: <Rex> but was: <null>`) para caçar os bugs.
O que a suíte de testes tem de melhor do que testar tudo na mão com curl?

### 2. Mock e injeção de dependência (Aulas 13 a 15)
No `AgendaServiceTest`, o `@Mock` cria um `AtendimentoRepository` falso e o
`@InjectMocks` o injeta no service. Explique a relação disso com o `@Autowired`
que o Spring faz em produção — quem "injeta" em cada mundo, e por que o teste
consegue rodar sem banco e sem subir o Spring?

### 3. `==` vs `.equals()` (Aula 7)
Um dos bugs fazia o agendamento duplicado passar pela verificação de conflito.
Explique por que `==` entre Strings e `LocalDateTime` falhou aqui, por que ele
"funciona por sorte" com literais como `"Rex"`, e o que a sua correção mudou.

### 4. Sobrescrita vs sobrecarga (Aula 7)
Um dos bugs compilava sem nenhum erro: um método parecia sobrescrever
`getDuracaoMinutos`, mas na verdade criava uma assinatura nova. Explique a
diferença entre override e overload nesse caso e por que a anotação `@Override`
teria impedido o bug.

### 5. Singleton manual vs bean do Spring (Aula 14)
O `GeradorProtocolo` é um Singleton escrito à mão e causou um dos bugs.
Explique o que ele garante, qual foi o bug, e por que o `AgendaService`
(`@Service`) não corre o mesmo risco no container do Spring.

### 6. Cobertura de testes: onde parar? (Aula 15)
Dos 6 testes novos que você escreveu, alguns ficaram vermelhos (revelaram
bugs) e outros verdes de cara (regras já corretas). Vale a pena manter os que
ficaram verdes? Em um projeto real com prazo, o que você priorizaria testar:
caminho feliz, caminhos de erro, ou 100% de cobertura? Justifique.

---

## Parte 5 — Espaço livre (opcional)

Alguma dificuldade, dúvida ou comentário sobre o checkpoint?

```

```
