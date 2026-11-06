# Meus Gastos (MVVM com Kotlin e Jetpack Compose)

Projeto de referência do tutorial "Meus Gastos". Pacote: `br.unasp.meusgastos`, minSdk 24, Lifecycle Compose 2.8.7.

## Como abrir
1. No Android Studio: **File > Open** e escolha a pasta `MeusGastos`.
2. Aguarde a sincronização do Gradle. 
3. **Build > Make Project** e depois **Run**.

Alternativa (como no tutorial, seção 10): crie um projeto *Empty Activity* com Kotlin DSL, package `br.unasp.meusgastos`, e copie apenas a pasta `app/src/main/java/br/unasp/meusgastos`, ajustando `libs.versions.toml` e `app/build.gradle.kts` conforme o passo 2 e 3 do tutorial.

## Conteúdo
- `app/src/main/java/.../model`, `viewmodel`, `ui`: código do apêndice (com previews nos arquivos de `ui/`)
- `ARQUITETURA.md`: respostas às 3 perguntas
- `ESTRAGOS.md`: registros dos 5 experimentos de depuração
