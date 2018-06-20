# Light RSA: Write-up

У нас есть публичный ключ (*e*, *n*) = (257, 356 233), а чтобы расшифровать сообщение, нужен приватный.

Читаем внимательно алгоритм и думаем. 

У умного читателя должен был возникнуть вопрос: а какая операция создает сложность взлома?

Для сравнения почитаем RSA — там мы ищем *d* такое, что *ed* == 1 (mod (*p* – 1) × (*q* – 1))! То есть сложность заключается именно в факторизации числа *n* и нахождении его делителей *p* и *q*.

Тут же, казалось бы, делители вообще не нужны. Но не очень понятно, как из *e* получить *d*.

На самом деле, эта операция называется *взятием обратного по модулю*. Есть куча интернет-утилит, которые позволят это сделать, но мы-то хотим узнать, как это работает изнутри.

Напомню, что нам нужно найти *d* такое, что *ed* == 1 (mod *n*). 

## Теорема Эйлера

Есть [функция Эйлера](https://ru.wikipedia.org/wiki/%D0%A4%D1%83%D0%BD%D0%BA%D1%86%D0%B8%D1%8F_%D0%AD%D0%B9%D0%BB%D0%B5%D1%80%D0%B0) — *phi*(*n*) — число натуральных чисел, меньших *n* и взаимно простых с ним.

Можно узнать (из того же RSA), что в нашем случае (произведение двух простых чисел) функция Эйлера будет равна (*p* – 1) × (*q* – 1).

Так вот, есть ещё и [теорема Эйлера](https://ru.wikipedia.org/wiki/%D0%A2%D0%B5%D0%BE%D1%80%D0%B5%D0%BC%D0%B0_%D0%AD%D0%B9%D0%BB%D0%B5%D1%80%D0%B0_(%D1%82%D0%B5%D0%BE%D1%80%D0%B8%D1%8F_%D1%87%D0%B8%D1%81%D0%B5%D0%BB)) — она о том, что *a* в степени *phi*(*n*) дает остаток 1 по модулю *n*.

Но тогда *a*, умноженное на *a* в степени (*phi*(*n*) – 1) также дает остаток 1 по модулю *n*.

Значит, искомое число равно остатку от деления на *n* числа *e* в степени ((*p* – 1) × (*q* – 1) – 1).

Факторизовать число можно с помощью несложной программы или Интернета, возвести в степень и взять остаток — с помощью любого калькулятора (или не любого, степень-то большая).

## Расширенный алгоритм Евклида

Давайте решим в **целых** числах *d*, *f* вспомогательное уравнение:

> *e* × *d* + *n* × *f* = 1

Такие уравнения называются [диофантовыми уравнениями второго порядка](https://ru.wikipedia.org/wiki/%D0%94%D0%B8%D0%BE%D1%84%D0%B0%D0%BD%D1%82%D0%BE%D0%B2%D0%BE_%D1%83%D1%80%D0%B0%D0%B2%D0%BD%D0%B5%D0%BD%D0%B8%D0%B5) и решаются [расширенным алгоритмом Евклида](http://www.e-maxx.ru/algo/extended_euclid_algorithm).

Если взять остаток от деления на *n* обеих частей, получим искомое:

> *e* × *d* = 1 (mod *n*)


## Но...

Зачем думать, если есть [вольфрам](http://wolframalpha.com/)?

Нам нужно найти такое *d*, что *ed* == 1 (mod *n*). Так и введем:

```
257 * d = 1 (mod 356233)
```

Меньше, чем через секунду видим ответ — `d = 80395`.

Считаем: (67410 × 80395) mod 356233 = 54321. Красиво. Похоже на флаг.

*Мораль*: это второй таск вида «Я придумал свой алгоритм, посмотрите, как я хорош», а заканчивается он тем же самым — успешным нахождением флага.

Так что вот совет на будущее: **никогда** не придумывайте свои криптоалгоритмы :)

Если говорить более серьёзно, то сегодня очень много операций можно выполнять достаточно быстро. Для ассиметричного шифрования же, наоборот, нужно придумать такую операцию, которую никто быстро считать не умеет.

Более того, RSA с модулем 356233 не сильно надежнее нашего Light RSA, ведь разложить его на простые множители не составляет особого труда. Чтобы зашифровать реально надежно, нужно брать большие числа.

На всякий случай, вдруг кто не понял, *спойлер*: если мы возьмем большой-большой модуль *N* (например, 2048-битный — порядка 2 в 2048 степени), то RSA взломать будет сложно, а Light RSA — просто, так как операция поиска обратного по модулю, являющаяся «самой трудоёмкой» в нашем алгоритме, работает намного быстрее разложения на простые множители.

Флаг: **54321**