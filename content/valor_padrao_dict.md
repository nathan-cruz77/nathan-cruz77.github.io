Title: Valores padrão em dicionários
Date: 27-12-2017
Category: Programação
Tags: Python, Dict


Recentemente observei que um problema recorrente em python, principalmente para
iniciantes na linguagem, é mapear valores de uma lista para outros valores
específicos. Um exemplo deste tipo de problema é: dado uma lista de objetos,
separá-los de acordo com seu tipo.

Para resolver este problema podemos criar um dicionário onde as chaves são os
tipos e os valores são listas de objetos daquele tipo. Uma solução é uma função
de tenha o seguinte comportamento:
```python
>>> l = [1, 2.6, 5.3, 'asdasd', 8, 'bola', None]
>>> func(l)
{'int': [1, 8], 'float': [2.6, 5.3], 'str': ['asdasd', 'bola'], 'NoneType': [None]}
```

Uma forma de construir este dicionário é descobrindo todas as possíveis chaves e
populando-as com uma lista vazia antes de chamar `append`:
```python
def func(l):
   d = {}

   keys = {type(i).__name__ for i in l}
   for k in keys:
       d[k] = []

   for v in l:
       key = type(v).__name__
       d[key].append(v)
   return d

# >>> l = [1, 2.6, 5.3, 'asdasd', 8, 'bola', None]
# >>> func(l)
# {'int': [1, 8], 'str': ['asdasd', 'bola'], 'NoneType': [None], 'float': [2.6, 5.3]}
```
Mas isto é simplesmente cansativo.

Inicializar dicionários é uma tarefa comum então o método
[`setdefault`](https://docs.python.org/3.6/library/stdtypes.html?highlight=dict#dict.setdefault)
foi adicionado à classe `dict` para dar um valor padrão à uma chave caso ela
ainda não esteja inserida:
```python
>>> d = {}
>>> d.setdefault('a', []).append('valor1')
>>> d.setdefault('a', []).append('valor2')
>>> d.setdefault('b', []).append('valor3')
>>> d.setdefault('b', []).append('valor4')
>>> d
{'b': ['valor3', 'valor4'], 'a': ['valor1', 'valor2']}
```

Podemos usar este método para reescrever a função:
```python
def func(l):
    d = {}
    for v in l:
        key = type(v).__name__
        d.setdefault(key, []).append(v)
    return d

# >>> l = [1, 2.6, 5.3, 'asdasd', 8, 'bola', None]
# >>> func(l)
# {'float': [2.6, 5.3], 'str': ['asdasd', 'bola'], 'int': [1, 8], 'NoneType': [None]}
```
Bem melhor!

O problema aqui é que teríamos que lembrar de chamar `setdefault` toda vez. Para
resolver este tipo de problema a biblioteca padrão oferece
[`collections.defaultdict`](https://docs.python.org/3.6/library/collections.html#collections.defaultdict):
```python
>>> from collections import defaultdict
>>> d = defaultdict(list)
>>> d['a'].append('value1')
>>> d['a'].append('value2')
>>> d['b'].append('value3')
>>> d['b'].append('value4')
>>> d['a']
['value1', 'value2']
>>> d['b']
['value3', 'value4']
```

Usando `collections.defaultdict` para reescrever a função ficamos com:
```python
def func(l):
    d = defaultdict(list)
    for v in l:
        k = type(v).__name__
        d[k].append(v)
    return dict(d)

# >>> l = [1, 2.6, 5.3, 'asdasd', 8, 'bola', None]
# >>> func(l)
# {'float': [2.6, 5.3], 'str': ['asdasd', 'bola'], 'int': [1, 8], 'NoneType': [None]}
```
Note que passamos uma função para o construtor do `defaultdict`. Está função é
chamada para construir o objeto padrão quando a chave ainda não tiver sido
inserida. Poderíamos usar outras estruturas _built-in_, da biblioteca padrão ou
mesmo funções próprias para construir o valor padrão.
