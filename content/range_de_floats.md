Slug: range-of-floats
Title: Construindo um range de floats
Date: 17-07-2016 00:28
Modified: 19-07-2016 22:37
Category: Python
Lang: pt

Há algumas semanas tive que implementar alguns algoritmos de inteligência
artificial para um trabalho da faculdade. Acabei fazendo alguns códigos
que talvez sejam úteis para outras pessoas, então decidi compartilhá-los.

Obs: Todos os códigos deste artigo foram testados em Python 3.4.

## Um `range` de `float`'s
Uma das (sub)tarefas era gerar todos os números num dado intervalo
com um dado passo. Isto é:
 * [0, 5) com passo 1 deveria resultar em: [0, 1, 2, 3, 4]
 * [0, 0.5) com passo 0.1 deveria resultar em: [0, 0.1, 0.3, 0.4]

A maioria das pessoas com um pouco de familiaridade com python diria
que tal problema pode ser resolvido com a função  [`range`](https://docs.python.org/3.4/library/functions.html?highlight=range#func-range). Realmente,
esta função extremamente conveniente resolve este tipo de problema.
Então tentei resolver o problema usando `range`.
```python
>>> list(range(0, 5, 1))
[0, 1, 2, 3, 4]
>>>list(range(0, 0.5, 0.1))
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'float' object cannot be interpreted as an integer
```

`range` claramente resolve o primeiro caso do problema mas, por que não
resolve o segundo? Achei que estava fazendo alguma coisa errada, como
colocar os argumentos em ordem errada, então fui procurar a
[documentação](https://docs.python.org/3.4/library/stdtypes.html?highlight=range#range).
Fiquei um pouco decepcionado ao descobrir que os argumentos não podem
ser do tipo `float`. Como precisava realmente disto decidi fazer meu
próprio `float_range`.

Minha primeira tentativa foi:
```python
def float_range(start, stop=None, step=1):
    if stop is None:
        stop, start = start, 0

    while(start < stop):
        yield start
        start += step
```
Me senti bastante esperto com esta solução até encontrar um problema
(que acredito ter sido o motivo para não usarem `float` em `range`).
```python
>>> list(float_range(0, 5, 1))
[0, 1, 2, 3, 4]
>>> list(float_range(5))
[0, 1, 2, 3, 4]
>>> list(float_range(0, 0.5, 0.1))
[0, 0.1, 0.2, 0.30000000000000004, 0.4]
```
Pois é, estava indo tudo bem demais para ser verdade. Este problema de
aritmética de ponto flutuante é bastante conhecido e está relacionado
a dificuldade de representar números "quebrados" em potências de base
2 (binário). Para mais informação veja [este](http://stackoverflow.com/a/588014)
texto.

Felizmente existe uma função (_built-in_) no python que me ajudou a
resolver este problema. A função [`round`](https://docs.python.org/3/library/functions.html?highlight=round#round), como sugere o nome, arredonda
o número para o inteiro mais próximo. Apesar desta funcionalidade ser
bastante mágica, não é o mais incrível desta função. Existe um segundo
argumento que determina quantos dígitos devem ser considerados ao arredondar.

Com esta nova descoberta fiz uma segunda tentativa:
```python
def float_range(start, stop=None, step=1):
    if stop is None:
        stop, start = start, 0

    while(start < stop):
        yield round(start, 1)
        start += step
```
Mas esta solução falhava caso a precisão necessária fosse maior que 1.
```python
>>> list(float_range(0, 0.5, 0.1))
[0, 0.1, 0.2, 0.3, 0.4]
>>> list(float_range(0, 0.05, 0.01))
[0, 0.0, 0.0, 0.0, 0.0]
```
Este problema claramente se repetiria caso mantivesse a precisão constante.
Decidi que a melhor solução seria determinar a precisão do passo e
usar aquela precisão ao arredondar. Fiz então uma pequena função auxiliar
que determina a precisão.
```python
def precision(number):
    try:
        res = len(str(number).split('.')[1])
    except IndexError:
        res = 1

    return res
```
Para isto transformamos o número numa string, a "quebramos" no caracter
"." e a precisão será o tamanho da segunda string resultante da quebra.
Se a quebra resultar apenas em uma string, trata-se de um `int` e a precisão é 1.

Com esta função podemos melhorar a solução anterior.
```python
def float_range(start, stop=None, step=1):
    if stop is None:
        stop, start = start, 0

    while(start < stop):
        yield round(start, precision(step))
        start += step
```
Esta, finalmente, funciona como o necessário.
```python
>>> list(float_range(0, 5, 1))
[0, 1, 2, 3, 4]
>>> list(float_range(0, 0.5, 0.1))
[0, 0.1, 0.2, 0.3, 0.4]
>>> list(float_range(0, 0.5, 0.01))
[0, 0.01, 0.02, 0.03, 0.04, 0.05, 0.06, 0.07, 0.08, 0.09, 0.1, 0.11, 0.12, 0.13, 0.14, 0.15, 0.16, 0.17, 0.18, 0.19, 0.2, 0.21, 0.22, 0.23, 0.24, 0.25, 0.26, 0.27, 0.28, 0.29, 0.3, 0.31, 0.32, 0.33, 0.34, 0.35, 0.36, 0.37, 0.38, 0.39, 0.4, 0.41, 0.42, 0.43, 0.44, 0.45, 0.46, 0.47, 0.48, 0.49]
```
