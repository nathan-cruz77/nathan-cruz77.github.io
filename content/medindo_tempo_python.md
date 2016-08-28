Title: Medindo tempo de execução
Date: 21-07-2016 01:39
Category: Programação
Tags: Python, Otimização, Shell, Profiling, Bash, Linux

No último [post]({filename}/fibonacci_com_cache.md) falei um pouco sobre como
usar cache para melhorar o desempenho do fibonacci recursivo. Os gráficos que
gerei foram feitos usando [Gnuplot](http://www.gnuplot.info/).

Como queria uma quantidade significativa de dados para estes gráficos, não tinha
como gerá-los manualmente. Precisava automatizar.

Já havia usado a ferramenta `time`, uma utilidade oferecida pelo
[`bash`](https://en.wikipedia.org/wiki/Bash_%28Unix_shell%29), mas tive
dificuldade em formatar a saída, que tem a seguinte forma:
```shell
bla@notebook:~$ time python3 fibonacci.py 25
real	0m0.064s
user	0m0.056s
sys 	0m0.008s
```
Depois de várias buscas na internet descobri que existe um programa externo ao
`bash` também chamado `time` que também mede o tempo de execução e aceita
operadores de formato. O programa está instalado em `/usr/bin/` (pelo menos no
meu notebook com [Debian](https://www.debian.org/)) e, embora este diretório
faça parte da variável de ambiente `PATH`, é ofuscado pelo `time` do `bash`.
Para usá-lo então tive que invocar usando o caminho para o arquivo. Por preferência
pessoal usei o caminho absoluto (`/usr/bin/time`).

A saída do `/usr/bin/time` é no formato:
```shell
bla@notebook:~$ /usr/bin/time python3 fibonacci.py 25
0.06user 0.00system 0:00.06elapsed 95%CPU (0avgtext+0avgdata 7724maxresident)k
0inputs+0outputs (0major+865minor)pagefaults 0swaps
```
Como estava interessado apenas no tempo total usado (_elapsed_ na saída acima),
pude usar uma string de formato bem simples para pegar apenas este campo.
```shell
bla@notebook:~$ /usr/bin/time -f "%E" python3 fibonacci.py 25
0:00.06
```
Para remover a parte dos minutos (antes do `:`) e ter a saída como um número
facilmente interpretável usei o programa [`cut`](https://pt.wikipedia.org/wiki/Cut_%28Unix%29).
No caso em questão bastou determinar o delimitador de campos como `:` e o campo
de interesse como 2.
```shell
bla@notebook:~$ /usr/bin/time -f "%E" python3 fibonacci.py 25 | cut -d: -f2
0:00.07
```
Isso não foi exatamente como desejado. O que está acontecendo aqui é que o `|`
está redirecionando o [`stdout`](https://pt.wikipedia.org/wiki/Fluxos_padr%C3%A3o#Sa.C3.ADda_padr.C3.A3o_.28stdout.29)
do comando anterior a ele para o
[`stdin`](https://pt.wikipedia.org/wiki/Fluxos_padr%C3%A3o#Entrada_padr.C3.A3o_.28stdin.29)
do `cut`, mas o `/usr/bin/time` não imprime no `stdout` e sim no
[`stderr`](https://pt.wikipedia.org/wiki/Fluxos_padr%C3%A3o#Erro_padr.C3.A3o_.28stderr.29)!
Uma forma de contornar este problema é redirecionar o `stderr` para o `stdout`.
Felizmente a sintaxe de script do `bash` nos permite fazer isto facilmente.
A saída padrão (`stdout`) é representada como `1` e a saída de err (`stderr`) é
representada como `2`. Para redirecionar o `stderr` para o `stdout` podemos usar
`2>&1`. É possível fazer o contrário também (redirecionar `stdout` para `stderr`)
usando `1>&2`. Então, usando a primeira forma, atingimos o resultado esperado.
```shell
bla@notebook:~$ /usr/bin/time -f "%E" python3 fibonacci.py 25 2>&1 | cut -d: -f2
00.05
```
Para testar o tempo de execução `fibonacci.py` para vários números bastava colocar
tudo isto num laço e jogar a saída em um arquivo.
```shell
bla@notebook:~$ for n in {1..5}
> do
> /usr/bin/time -f "%E" python3 fibonacci.py $n 2>&1 | cut -d: -f2 > tempos.txt
> done
00.02
00.02
00.02
00.02
00.02
```

Foi assim que gerei os dados dos gráficos do post anterior. Desde então tenho me
perguntado se esta é a solução mais pythônica. Acredito que não, é excessivamente
complexo. Então comecei a mexer no código para que não precisasse fazer todo esse
malabarismo com o shell.

## Medindo tempo com python
A tarefa de medir o tempo de execução de pedaços de código é geralmente parte do
processo de _profiling_, que visa detectar gargalos de desempenho. Como este
processo é bastante usado, também são as subtarefas que o compõe. Medir o tempo
de execução não é exceção.

Como toda (ou pelo menos quase toda) linguagem de programação, python oferece um
módulo para trabalhar com medidas de tempo na biblioteca padrão. A função
[`time`](https://docs.python.org/3/library/time.html?highlight=time.time#time.time)
(pertencente ao módulo de mesmo nome) retorna a quantidade de segundos passados
desde [_epoch_](https://en.wikipedia.org/wiki/Unix_time) (1 de janeiro de 1970).
Esta representação de tempo popular é também conhecida como _Unix time_.

Com esta função é possível medir o tempo de execução de trechos de código facilmente.
Por exemplo:
```python
import time

def slow_func():
    # Função de interesse
    time.sleep(10)

tempo_inicial = time.time()
slow_func()
tempo_final = time.time()

tempo_total = tempo_final - tempo_inicial

print('{:.3f}s'.format(tempo_total))
```
Este código já pode ser usado dentro de um laço para gerar todos os tempos no
formato desejado de uma forma mais limpa que apresentada anteriormente mas, será
que pode ficar melhor?

Sim! Usando um decorator podemos tornar este código mais reutilizável e pythônico!
```python
import time
import functools


def _fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)


def temporizador(func):

    functools.wraps(func)
    def wrapper(*args, **kwargs):
        tempo_inicio = time.time()
        res = func(*args, **kwargs)
        tempo_fim = time.time()

        tempo_total = tempo_fim - tempo_inicio

        print('{} executou em: {:.3f}s'.format(func.__name__, fim - inicio))
        return res

    return wrapper


@temporizador
def fibonacci(n):
    return _fibonacci(n)
```
Assumindo que o código acima está no arquivo `fibonacci_com_decorator.py` podemos
usá-lo no prompt interativo da seguinte forma:
```python
>>> from fibonacci_com_decorator import *
>>> fibonacci(3)
fibonacci executou em: 0.000s
2
>>> fibonacci(25)
fibonacci executou em: 0.053s
75025
```
Foi necessário adicionar um _wrapper_ para a função fibonacci devido a sua natureza
recursiva. Se o decorator fosse aplicado à função recursiva os tempos de execução
de todas as chamadas recursivas seriam mostrados.

O ponto mais interessante é que este método permite medir o desempenho de diferentes
partes do código executando-o apenas uma vez. Além disto, os dados de performance
podem ser usados em logs, permitindo que a aplicação seja analisada sem ser
interrompida.
