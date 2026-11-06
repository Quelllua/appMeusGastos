# ARQUITETURA.md — Meus Gastos (MVVM)

## 1. Se a fonte de dados mudar de memória para outra implementação, que partes podem permanecer iguais? Quais ainda podem precisar de adaptação?

**Permanecem iguais:** toda a pasta `ui/` (TelaGastos, FormularioGasto, LinhaGasto, ResumoTotal, Etiqueta), as regras de validação e o estado derivado do `GastosViewModel`, e as classes `Gasto` e `Categoria`. Isso acontece porque o ViewModel depende da interface `GastoRepository` (o contrato: `listar`, `adicionar`, `remover`) e não de uma classe concreta. Trocar `GastoRepositoryMemoria()` por `GastoRepositoryDemo()` exige mudar apenas o valor padrão do parâmetro do construtor.

**Podem precisar de adaptação**, caso a nova fonte seja de outro tipo (por exemplo, Room):
- A criação do repositório: Room precisa de `Context` e de um banco, então o valor padrão no construtor deixa de servir e seria necessária uma fábrica ou injeção de dependência.
- O contrato: acesso a banco costuma ser assíncrono (`suspend` ou `Flow`). Se `listar()` passar a ser `suspend`, o ViewModel precisaria de `viewModelScope`/corrotinas.
- A função `sincronizar()`, que deixaria de ser necessária se o repositório expusesse um `Flow` observável.

Ou seja, a troca entre duas implementações em memória é gratuita, mas uma troca para persistência real só é indolor se o contrato já tiver sido pensado para isso.

## 2. Por que total e quantidade são calculados a partir da lista visível? Um erro concreto se fossem atualizados manualmente.

`total` e `quantidade` são *estado derivado*: dependem apenas de `gastos` e `filtro`, então são recalculados a cada leitura e nunca ficam desatualizados. Como saem de `gastosVisiveis`, acompanham o filtro automaticamente.

Se `total` fosse uma variável guardada e atualizada só em `adicionar()`:
1. Cadastro "Cinema" (R$ 32,00) e "Conta de luz" (R$ 142,00): total mostra R$ 174,00.
2. Removo "Cinema": `remover()` não atualizou o total, e a tela continua em R$ 174,00 com apenas um item de R$ 142,00 na lista.
3. Filtro por "Lazer": a lista fica vazia, mas o total continua mostrando R$ 174,00.

Para corrigir, cada ação (adicionar, remover, filtrar, e qualquer outra que viesse a existir) teria de lembrar de atualizar o número; basta esquecer uma para a tela mostrar informação inconsistente.

## 3. O que os componentes sem estado conseguem fazer nos previews que seria menos simples com dependência direta de um ViewModel?

Componentes sem estado recebem dados e callbacks por parâmetro, então o `@Preview` pode passar valores fabricados e lambdas vazias (`{}`). Com isso:
- é possível desenhar o `FormularioGasto` já com a mensagem "O valor precisa ser um número" (`FormularioComErroPreview`) sem digitar nada nem acionar a validação;
- é possível ver `ResumoTotal` vazio e preenchido, `Etiqueta` ativa e inativa, `LinhaGasto` com qualquer texto, tudo lado a lado e sem executar o app;
- não é preciso instanciar repositório, ViewModel nem `ViewModelStore`.

Com dependência direta de um ViewModel real, o preview teria de construir o ViewModel (e seu repositório) e, para chegar ao estado de erro, executar a ação que o provoca. Os valores ficariam amarrados à lógica, em vez de serem escolhidos livremente para testar o visual.

## Resposta rápida às perguntas de autoavaliação

- **Model:** representa e acessa dados, em Kotlin puro. **ViewModel:** guarda estado observável e executa regras e ações. **View:** desenha o estado e encaminha eventos.
- **Quem valida:** o `GastosViewModel.adicionar()`. **Quem só mostra a mensagem:** o `FormularioGasto`, que recebe `erro` pronto.
