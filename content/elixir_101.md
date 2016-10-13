Title: Elixir e a web
Date: 12-10-2016 23:50
Category: Programação
Tags: Elixir, Phoenix

Elixir é uma linguagem de programação funcional criada pelo brasileiro José
Valim, autmagicamente rápida e com vários açúcares sintáticos que roda em cima
da VM do Erlang. Tinha ouvido falar bastante de programação funcional mas não
sabia direito o que era. Fui procurar e achei esta
definição: `Programação funcional é um paradigma de programação que trata a
computação como uma avaliação de funções matemáticas e que evita estados ou
dados mutáveis`. Então, por esta definição, temos que todos os tipos de dados em
elixir são imutáveis. Isso pode parecer muito estranho no começo mas, na maioria
dos casos do dia-a-dia, não vai gerar muitas complicações.


## Code!!!
Sem mais burocracia, vamos ao que interessa. Tenho trabalhado com elixir faz
alguns meses e tem sido mais divertido do que eu esperava. Vou falar de algumas
coisas que me chocaram no início.

### Pattern Matching
Honestamente, isso ainda me parece bastante mágico. Não sei como funciona por
trás dos panos mas, pelo que pude notar, quando uma variável é usada do lado
esquerdo do operador `=` uma atribuição é feita à ela (a não ser que esteja
precedida por `^`). Caso seja usada do lado direito do operador o valor
vinculado a ela é usado.
``` elixir
# Valor 1 vinculado a variável x
x = 1

# Valor de x "dá match" no literal 1 (1 == x)
1 = x

# Valor de x igual ao valor a direita do =
^x = 1

# Explode
2 = x
# ** (MatchError) no match of right hand side value: 1

# Pegar apenas um valor de um mapa
%{name: x} = %{name: "Um nome qualquer", isso_nao_sera_pego: 12}

# Match baseado em um valor literal (várias funções usam esse padrão)
{:ok, x} = {:ok, "valor qualquer"}
```

### Funções e Módulos
Uma coisa que achei bastante estranho é não ter `return`. O último valor gerado
é retornado.
```elixir
# Funções fora de módulos só podem ser anônimas
func = fn (x) -> x + 10 end

# E tem que ser chamadas com essa sintaxe estranha
func.(12)

# Nomes de módulos tem que começar com letra maiúscula
defmodule ModuloX do
  # Funções com a mesma assinatura podem ser declaradas várias vezes
  # e a primeira que der match nos argumentos é invocada
  def func_2(x = 12) do
    IO.puts "executing first definition of func_2"
  end
  # _ dá match em qualquer coisa e não pode ser referenciado
  def func_2(_) do
    IO.puts "executing default func_2"
  end

  # Funções nomeadas podem ser privadas ao módulo com defp
  defp func_3(x) do
    x / 2
  end
end

# Parênteses para chamar funções são opcionais
ModuloX.func_2(12) == ModuloX.func_2 12 # true

ModuloX.func_2 12 # "executing first definition of func_2"
ModuloX.func_2 # "executing default func_2"
```

### Iteration no more
Não é muito comum fazer laços em elixir. Apesar de ser possível é preferível, e
mais rápido, usar recursão para percorrer e processar coleções. O módulo `Enum`
da biblioteca padrão oferece várias funções comuns para processamento de
coleções.
```elixir
# Uma lista de números (sim, os elementos podem ter tipos diferentes)
nums = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# Aplica uma função a todos os elementos de uma coleção
Enum.map(nums, fn (x) -> x + 1 end) # [2, 3, 4, 5, 6, 7, 8, 9, 10, 11]

# Filtra apenas os pares
Enum.filter(nums, fn (x) -> rem(x, 2) == 0 end) # [2, 4, 6, 8, 10]

# Soma todos os elementos
Enum.reduce(nums, fn (x, acc) -> acc + x end) # 55

# Joga todos os elementos para uma outra coleção
Enum.into(["1", "2", "3"], "") # "123"
```

## Phoenix
[Phoenix](http://www.phoenixframework.org) é o framework web _de facto_ para
construção de APIs e aplicações web com elixir. Vários dos seus aspectos foram
inspirados em frameworks web de sucesso, como `Rails` e `Django`. A forma de
instalação mais simples é usando o
[`Mix`](http://elixir-lang.org/docs/stable/mix/Mix.html)(uma ferramenta de
_build_ instalada juntamente com o elixir que traz _tasks_ embutidas para criar,
compilar e testar projetos elixir, além de gerenciar suas dependências).

### Instalação
```shell
# Instala o gerenciador de pacotes elixir "hex"
$ mix local.hex

# Instala o phoenix propriamente dito
$ mix archive.install https://github.com/phoenixframework/archives/raw/master/phoenix_new.ez
```
É necessário instalar também algumas outras ferramentas (`PostgreSQL` e
`inotify`) mas a instalação varia de acordo com o ambiente. Para mais
instruções [clique aqui](http://www.phoenixframework.org/docs/installation#section-postgresql).

### Criar o projeto
Criar um prjeto phoenix vazio é bastante simples, basta fazer:
```shell
$ mix phoenix.new diretorio_do_projeto
```
Considerando que as credenciais do banco de dados estão devidamente configuradas
no arquivo do configurações do projeto (`config/config.exs`) podemos criar um
banco de dados para a aplicação e iniciá-la localmente com:
```shell
# Cria banco de dados local
$ mix ecto.create

# Inicia a aplicação
$ mix phoenix.server
```
Uma aplicação modelo estará disponível em [localhost:4000](http://localhost:4000).

Feito isto é possível escrever models, views, controllers e módulos auxiliares
da forma mais adequada para o objetivo do projeto e ser feliz!

## Links úteis
Alguns links que podem ser muito úteis:

  * [Elixir docs](http://elixir-lang.org/docs/stable/elixir/Kernel.html)
<- Cheque a opção "modo noturno" no rodapé da página
  * [Phoenix docs](https://hexdocs.pm/phoenix/Phoenix.html)
  * [Ecto docs](https://hexdocs.pm/ecto/Ecto.html)
