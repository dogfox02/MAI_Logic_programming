# Отчет по лабораторной работе №3
## по курсу "Логическое программирование"

## Решение задач методом поиска в пространстве состояний

### студент: Сысоев М.А.

## Результат проверки

| Преподаватель     | Дата         |  Оценка       |
|-------------------|--------------|---------------|
| Сошников Д.В. |              |               |
| Левинская М.А.|              |               |

> *Комментарии проверяющих (обратите внимание, что более подробные комментарии возможны непосредственно в репозитории по тексту программы)*


## Введение

Метод поиска в пространстве состояний применяется для задач искусственного интелекта.

Почему Prolog оказывается удобным языком для решения таких задач?

Поиск в пространстве состояний является по сути задачей поиска в графе, генерируемом по правилам перехода между состояниями, алгоритмы обхода которого довольно просто на нем реализуются.

В основе процесса логического вывода лежит поиск в глубину, что уже наводит на мысли об использовании языка Prolog для решения таких типов задач.

## Задание (Вариант 2)

Три миссионера и три каннибала хотят переправиться с левого берега реки на правый. Как это сделать за минимальное число шагов, если в их распоряжении имеется трехместная лодка, и ни при каких обстоятельствах (в лодке или на берегу) миссионеры не должны оставаться в меньшинстве.

## Принцип решения

Задача была описана двумя состояниями - начальным и конечным.  
`state([LM, LK],[RM, RK], B)` будет обозначать состояние, при котором на левом берегу находится LM миссионеров и LK каннибалов, на правом берегу RM миссионеров и RK каннибалов. Лодка находится на берегу B (l - левый берег, r - правый).
Правила перехода между состояниями заданы при помощи предиката `crossing(S1, S2)`, где S1 - текущее состояние, S2 - состояние после перехода.
```prolog
% Переправа с правого на левый берег
crossing(state([LM, LK], [RM, RK], r), state([X, Y], [V, W], l)):- 
   	check(M),
    check(K),
    (M >= K; M = 0),
    SUM is M + K, 
    SUM =< 3, 
    SUM > 0,
    X is LM + M, 
    Y is LK + K,
    V is RM - M, 
    W is RK - K, 
   	is_possible_state(state([X, Y], [V, W], l)), 
    in_safe(state([X, Y], [V, W], l)).
```
Аналогично и для переправы с левого на правый.

Далее поиск в пространстве состояний происходит при помощи одного из алгоритмов (поиск в глубину, в ширину, с итеративным заглублением).

Приведу в пример реализацию, как будет выяснено ниже, самого оптимального алгоритма в данной задаче - поиск в ширину.
```prolog
b_path([[Cur|T]|_], Cur, [Cur|T]).  
b_path([[Cur|T]|TT], Goal, Result):-  
    setof(           % получаем список продолженных  путей                    
        [Next, Cur|T],
    	(crossing(Cur, Next), not(member(Next, [Cur|T]))),
    	New
    ), 
    append(TT, New, RR), !,    
    b_path(RR, Goal, Result);   
    b_path(TT, Goal, Result).  
                                
bfs(Start, Finish, Path, Time):-   
    b_path([[Start]], Finish, Reversed_path),   % находим путь (список состояний получается инвертированным)
    reverse(Reversed_path, Path),               % переворачиваем чтобы получить в прямом порядке
```
Все остальные реализации поиска представлены в файле `lab3.pl`.

На выходе любого из описанных алгоритмов поиска получаем цепочку состояний. Однако, необходимо получить последовательность действий для решения задачи. Для этого используем предикат `action(States, Actions)`, который переводит пару состояний, в действие, приведшее к переходу.

## Результаты

#### Поиск в глубину

Поиск в глубину выводит первое решение за 11 шагов.

Этот путь:
```prolog
carry([0,2],r)
carry([0,1],l)
carry([0,2],r)
carry([0,1],l)
carry([2,0],r)
carry([1,1],l)
carry([2,0],r)
carry([0,1],l)
carry([0,2],r)
carry([0,1],l)
carry([0,2],r)
```
> *carry([M, K], D) означает действие, в результате которого перевезено M миссионеров иCK каннибалов на берег D.*
И лишь 23-им по счету путем предлагает кратчайший путь к решению задачи в 5 шагов.

Кратчайшее решение:

```prolog
carry([0,2],r)
carry([0,1],l)
carry([3,0],r)
carry([0,1],l)
carry([0,3],r)
```

#### Поиск в ширину

Поиск в ширину сразу, как и ожидалось, нашёл кратчайший путь длины 5 для решения задачи.
```prolog
carry([1,1],r)
carry([1,0],l)
carry([3,0],r)
carry([0,1],l)
carry([0,3],r)
```

#### Поиск с итеративным заглублением

Поиск с итеративным заглублением, сочитая плюсы поиска в ширину и поиска в глубину, также сразу же нашёл самый короткий путь.
```prolog
carry([0,2],r)
carry([0,1],l)
carry([3,0],r)
carry([0,1],l)
carry([0,3],r)
```
> *Стоит отметить, что решение задачи не является единственным. Решений несколько.*

### Анализ алгоритмов поиска


| Алгоритм поиска | Длина найденного первым пути | Время работы |
|-----------------|------------------------------|--------------|
| В глубину       | 11                           | 0.00062      |
| В ширину        | 5                            | 0.00445      |
| IDS             | 5                            | 0.00582      |

Как можно заметить, поиск в глубину оказался самым быстрым. За ним идет алгоритм поиска в ширину. Самым медленным оказался поиск с итеративным заглублением.

Поиск в глубину уже заложен в системе логического вывода языка Пролог. Мы лишь запоминаем его шаги. Именно поэтому поиск в глубину расположился на первом месте.

Поиск в ширину оказался быстрее ids для решения данной конкретной задачи. Это можно объяснить тем, что поиск с итеративным заглублением после каждого заглубления не сохраняет уже полученные пути меньшей длины, и вынужден вычислять их каждый раз заново. Тогда как поиск в ширину хранит все пути-кандидаты и на каждом шагу лишь продляет их, если это возможно.

Хоть поиск в глубину и быстрее, но находит оптимальное решение не с первого раза, а лишь с 23-его.
Остальные же два алгоритма с первой попытки приходят пускай к разным, но оптимальным решениям.

Если говорить про пространственную сложность алгоритмов, то тут, очевидно, проигрывает поиск в ширину, так как он хранит очередь всех путей-кандидатов. Другие же два алгоритма работают с одним путем в один момент времени.

## Выводы

Алгоритм поиска в глубину лучше использовать, если не принципиально какой путь будет найден, а важно лишь его наличие/отсутствие.

Алгоритм поиска в ширину можно использовать, когда необходимо найти оптимальное решение, и при этом не критичен объем используемой памяти.

Алгоритм поиска с итерационным заглублением можно использовать, когда необходимо найти оптимальное решение и при этом важно количество занимаемой памяти.

Для данной задачи наилучшим вариантом является алгоритм поиска в ширину. Это объясняется тем, что кратчайшее решение лежит на довольно малой глубине (на глубине 6, если быть точным), что не позволяет случиться комбинаторному взрыву и использовать сколь угодно значимые объемы памяти. К тому же, поиск в ширину оказался быстрее всех, исключая поиск в глубину, так как он предлагает неоптимальное решение.

Итак, в процессе выполнения работы были изучены основные методы неинфоримрованного поиска в пространстве состояний. Указанные алгоритмы были проанализированы, оценены и сравнены. Были выявлены их достоинства и недостатки.



