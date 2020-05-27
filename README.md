# TheGreatCombinator

Android-приложение реализует интерфейс игры, похожей на ["Быки и коровы"](https://ru.wikipedia.org/wiki/Быки_и_коровы)

Присутствуют два режима игры, а также настройка уровня сложности.


## Правила:

Игра рассчитана на двух игроков. Игрок1 загадывает число, а Игрок2 делает попытки его отгадать. В ответ Игрок1 сообщает, сколько цифр угадано вплоть до позиции в загаданном числе (в игре - это "А") и сколько цифр угадано без совпадения с их позициями в загаданном числе (в игре - это "В").

_Пример_:

Задумано:&nbsp; **3216** <br>
Попытка:&nbsp;&nbsp; **4336**

**А = 1** (цифра 6 угадана вплоть до позиции) <br>
**В = 2** (цифра 3 дважды угадана, но стоит на неверной позиции)

В игре есть 2 режима:
- В первом компьютер загадывает число, а пользователь пытается отгадать.<br>
- Во втором наоборот - пользователь загадывает, а компьютер отгадывает.


## Скриншоты
<p>
  <img src="https://i.ibb.co/vVgNXML/mm.jpg" width="200"/>&nbsp;
  <img src="https://i.ibb.co/2WCRH6F/r46.jpg" width="200"/>&nbsp;
  <img src="https://i.ibb.co/NndSh2b/r69.jpg" width="200"/>&nbsp;
  <img src="https://i.ibb.co/4ZVm82W/s46.jpg" width="200"/>
</p>


## Подробности реализации

Проект написан на Kotlin в Android Studio.<br>
Текущий apk-файл доступен по [ссылке](https://yadi.sk/d/-QCrQiMHtNPLPQ)<br>
Приложение с изменённным дизайном [здесь](https://yadi.sk/d/uOWbDF8Hu-fEwg)

Тестирование логики - [JUnit4](https://junit.org/junit4/)<br>
Тестирование интерфейса - [Espresso](https://developer.android.com/training/testing/espresso?hl=ru)

Сборка: 
- `./gradlew build` - apk
- `./gradlew test` - тесты


## Алгоритм отгадывания

Основная идея состоит в том, чтобы после каждой сделанной попытки поддерживать множество возможных ответов `possibleAnswers` и выдавать в качестве следующей попытки один из его элементов.<br>
Однако для слишком больших чисел хранение такого множества в явном виде недопустимо (размер будет порядка 10^8), поэтому для стабильно быстрой работы приложения было реализовано три различных алгоритма, которые запускаются последовательно в зависимости от расчётного размера `possibleAnswers`.

Также важно отметить первую попытку, которую делает компьютер. Она построена по такому шаблону, чтобы в среднем отбросилось как можно больше вариантов ответа. (Для длины 4 - это `abcc`, где `a`, `b`, `c` - различные цифры).

Следующие несколько попыток выбираются случайно, но так, чтобы удовлетворять предыдущим запросам. Важно, что на данном этапе программа не хранит `possibleAnswers` явно.

В какой-то момент программа "понимает", что предполагаемый размер `possibleAnswers` достаточно мал, чтобы начать его поддерживать. На этом этапе запускается рекурсивный перебор с отсечением заранее неподходящих вариантов. А именно, на каждом шаге мы знаем префикс некоторого числа, и, если так получилось, что его никак нельзя дополнить до возможного ответа, эта ветка перебора обрывается. Иначе увеличиваем длину префикса на 1 и продолжаем. В конце, если число удовлетворяет всем условиям, оно добавляется во множество.<br>
_Например_, сделали попытку **1234** и получили ответ **A = 2, B = 0**. Тогда **123** не может быть префиксом ответа, так как в таком случае **A** хотя бы 3, однако известно, что **A = 2**.<br>
Или наоборот: **555** не может быть префиксом ответа, так как в нём недостаточно места, чтобы получить **A = 2**.<br>
С параметром **B** сильно сложнее, но, в целом, похожим образом.

Дальше начинает работу алгоритм, который пытается минимизировать некоторую эвристическую функцию от следующей попытки, перебирая все варианты в `possibleAnswers`.

Наконец, когда возможных вариантов ответа становится совсем мало, запускается самый медленный, но более оптимальный алгоритм, который для каждого возможного ответа вычисляет среднее квадратичное размеров `possibleAnswers`, если бы выбрали этот ответ в качестве следующей попытки, и возвращает элемент с минимальной такой величиной.


## Результаты

Сначала рассмотрим стандартный случай, то есть длина = 4, количество возможных цифр = 6.<br>
Для него точно известно, что любое число можно отгадать за максимум **5** попыток, а в среднем такой алгоритм будет угадывать за **3.7** попыток.

Наш алгоритм делает максимум **6** попыток, а в среднем угадывает за **3.79**, что не слишком хуже оптимального. К тому же работает сильно быстрее и легко обобщается на другие параметры загадываемого числа, что нельзя сказать об оптимальном.


**Все возможные варианты настроек.**

- по вертикали - длина числа
- по горизонтали - количество цифр

_Максимальное количество попыток:_
|      |   4  |   6  |   6  |   7  |   8  |   9  |
|:----:|:----:|:----:|:----:|:----:|:----:|:----:|
| **3**|   4  |   5  |   6  |   7  |   8  |   9  |
| **4**|   4  |   5  |   6  |   7  |   8  |   9  |
| **5**|   7  |   7  |   7  |   8  |   9  |   9  |
| **6**|   9  |   9  |   9  |   9  |  10  |  10  |
| **7**|   9  |  10  |  10  |  10  |  10  |  11  |
| **8**|   9  |  10  |  11  |  11  |  11  |  12  |

_Количество попыток в среднем:_
|      |   4  |   6  |   6  |   7  |   8  |   9  |
|:----:|:----:|:----:|:----:|:----:|:----:|:----:|
| **3**| 2.69 | 3.03 | 3.43 | 3.88 | 4.32 | 4.69 |
| **4**| 2.97 | 3.38 | 3.79 | 4.16 | 4.55 | 4.99 |
| **5**| 3.80 | 4.15 | 4.47 | 4.79 | 5.12 | 5.47 |
| **6**| 4.71 | 5.10 | 5.48 | 5.81 | 6.10 | 6.39 |
| **7**| 5.54 | 6.01 | 6.39 | 6.71 | 6.99 | 7.24 |
| **8**| 6.30 | 6.94 | 7.41 | 7.90 | 8.44 | 9.32 |

Видно, что результаты не сильно ухудшаются, несмотря на экспоненциальный рост количество вариантов.
<br><br>
<p align="right"> <i>Учебная практика, МКН СПбГУ, 1 курс, весна 2020</i> </p>
