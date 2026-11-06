# ESTRAGOS.md — Experimentos de depuração

> **Aviso de honestidade:** estes registros foram preparados sem executar o app em um emulador. "Minha previsão" e "O que explica o resultado" vêm da leitura do código. O campo **"O que observei"** está marcado como **[CONFIRMAR NO EMULADOR]**: rode cada experimento, compare com a previsão e ajuste o texto se o comportamento for diferente. Os experimentos 4 e 5 são principalmente de leitura de código.

---

## Experimento 1 — Construir o ViewModel na tela

**Experimento:** em `TelaGastos`, trocar `vm: GastosViewModel = viewModel()` por `vm: GastosViewModel = GastosViewModel()`.

**Minha previsão:** o app continua funcionando em uso normal: cadastrar e digitar parecem funcionar. A diferença aparece ao **girar a tela**: a Activity é recriada, o `ViewModelStore` não guarda essa instância, e os gastos cadastrados desaparecem. Não espero que "digitar sempre recrie" o ViewModel.

**O que observei:** **[CONFIRMAR NO EMULADOR]** (cadastrar itens, digitar, depois girar a tela, e comparar com a versão original usando `viewModel()`).

**O que explica o resultado:** `viewModel()` entrega uma instância associada ao `ViewModelStore` da Activity, que sobrevive a mudanças de configuração. Uma instância criada diretamente pertence à composição e morre junto com ela; se a composição for recriada, nasce um ViewModel novo, com repositório e lista vazios.

**Como restaurei o código:** voltei o parâmetro para `vm: GastosViewModel = viewModel()`.

---

## Experimento 2 — Lista não observável

**Experimento:** em `GastosViewModel`, trocar `mutableStateListOf<Gasto>()` por `mutableListOf<Gasto>()`.

**Minha previsão:** ao **remover**, a lista não redesenha, pois nenhum estado observável mudou. Ao **adicionar**, pode parecer funcionar por acaso: `adicionar()` também limpa `descricao` e `valorTexto` (que continuam observáveis), e essa mudança dispara uma recomposição que relê a lista. Total e quantidade ficam defasados quando a lista muda sem que outro estado mude junto.

**O que observei:** **[CONFIRMAR NO EMULADOR]**.

**O que explica o resultado:** Compose só recompõe quando lê um `State` que mudou. Uma `MutableList` comum muda sem avisar ninguém; qualquer atualização visível seria "carona" de outro estado. `mutableStateListOf` é o que torna `add`, `clear` e `remove` observáveis.

**Como restaurei o código:** voltei para `mutableStateListOf<Gasto>()`.

---

## Experimento 3 — Total manual

**Experimento:** guardar `var total` no ViewModel (em vez do `get()` calculado) e atualizá-lo somente em `adicionar()`.

**Minha previsão:** três caminhos deixam o número errado: (1) **remover** um item (o total não diminui); (2) **filtrar** por categoria (o total continua somando tudo, enquanto a lista mostra só uma parte); (3) voltar para "Todas" depois de remover. Só o fluxo "adicionar sem filtro" fica correto.

**O que observei:** **[CONFIRMAR NO EMULADOR]**.

**O que explica o resultado:** o valor guardado é uma cópia da informação que já existe em `gastos` e `filtro`. Cada ação que não o atualiza o deixa inconsistente. O valor derivado (`get() = gastosVisiveis.sumOf { ... }`) não tem como ficar desatualizado, pois é recalculado a partir da fonte.

**Como restaurei o código:** voltei para `val total: Double get() = gastosVisiveis.sumOf { it.valor }`.

---

## Experimento 4 — Acoplamento à classe

**Experimento:** no ViewModel, trocar o campo `private val repositorio: GastoRepository` por uma instância criada dentro da classe: `private val repositorio = GastoRepositoryMemoria()`, e remover o parâmetro do construtor.

**Minha previsão:** o app funciona exatamente igual. O custo aparece na hora de trocar para `GastoRepositoryDemo`: será preciso **editar o corpo do ViewModel**, e não apenas um valor de parâmetro; além disso, não será mais possível injetar um repositório falso em testes.

**O que observei:** **[CONFIRMAR NO EMULADOR]** (o comportamento visual não muda; confirme tentando fazer a troca para Demo).

**O que explica o resultado:** ao criar a classe concreta dentro do ViewModel, ele passa a depender da implementação, e não do contrato. O acoplamento é invisível na tela, mas aparece no custo de mudança e de teste.

**Como restaurei o código:** voltei ao construtor `class GastosViewModel(private val repositorio: GastoRepository = GastoRepositoryMemoria())`.

---

## Experimento 5 — Regra na tela

**Experimento:** no `FormularioGasto`/`TelaGastos`, validar a descrição dentro do `onClick` do botão, antes de chamar `aoAdicionar`.

**Minha previsão:** a validação funciona na tela, mas a regra passa a existir em dois lugares (ou só na View), ficando fora do alcance de testes sem interface. Se outra tela também cadastrar gastos, a regra teria de ser copiada, e as duas cópias podem divergir (por exemplo, uma aceitar descrição só com espaços).

**O que observei:** **[CONFIRMAR NO EMULADOR]**.

**O que explica o resultado:** a View deve desenhar estado e encaminhar eventos. Regras de negócio no ViewModel ficam em um só lugar, reutilizáveis por qualquer tela e testáveis com um teste unitário simples, sem Compose.

**Como restaurei o código:** removi a validação do `onClick` e mantive só `onClick = aoAdicionar`.
