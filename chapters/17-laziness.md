# Лень

В языке Haskell по умолчанию используются ленивые вычисления. Ленивым называется такое вычисление, которое откладывается до тех пор, пока кому-нибудь не понадобится результат этого вычисления. Соответственно, если он не понадобился никому, то и вычисляться ничего не будет.

## Начнём с C++

Допустим, нам нужен список из некоторого числа одинаковых IP-адресов. Да, в реальной жизни нам такое едва ли понадобится, но этот пример хорошо покажет суть ленивых вычислений.

Функция, возвращающая список адресов, на C++ может выглядеть так:
```cpp
typedef std::vector<std::string> IPAddresses;

IPAddresses generate_addresses( size_t how_many,
                                const std::string& address ) {
    const IPAddresses addresses( how_many, address );
    return addresses;
}
```
Теперь нам понадобилась функция, получающая заданное количество адресов из этого списка и выводящая их на экран. Например:
```cpp
void take_and_print( size_t how_many,
                     const IPAddresses& addresses ) {
    for( size_t i = 0; i < how_many; ++i ) {
        std::cout << addresses[i] << std::endl;
    }
} 

int main() {
    take_and_print( 2, generate_addresses( 100, "127.0.0.1" ) );
}
```
Функция `take_and_print` получает список, возвращённый функцией `generate_addresses`, а потом печатает первые два адреса из этого списка. Вывод будет таким:
```bash
127.0.0.1
127.0.0.1
```
И всё бы хорошо, но из 100 созданных строк фактически потребовались лишь первые две.  Оставшиеся 98 строк были созданы абсолютно напрасно. Было затрачено время, была затрачена память - и всё впустую.

Это - следствие строгости вычислений, присущей языку C++. Функция `generate_addresses` прямолинейна и сразу рвётся в бой. Сказали ей создать 100 адресов - получите 100. Скажут создать миллион - пожалуйста, вот вам миллион. Скажут миллиард - ну что ж, потерпите чуток, но будет вам и миллиард.

Тем временем функция `take_and_print` столь же прямолинейна, и ей абсолютно наплевать на усилия трудолюбивой функции `generate_addresses`. Если ей сказали отобразить лишь первые два элемента полученного контейнера, именно это она и сделает. И ей без разницы, сколько там ещё осталось элементов, десять или полмиллиарда.

Результатом строгости вычислений является лишняя работа. Но функции в Haskell, в отличие от своих трудолюбивых коллег из C++, терпеть не могут лишней работы.

## Вот как в Haskell

Откроем файл `Main.hs` и перепишем его:
```haskell
main = print (take 2 (replicate 100 "127.0.0.1"))
```
Функция `replicate` создаёт список из 100 адресов вида `127.0.0.1`, а функция `take` берёт 2 первых адреса из этого списка (о чём свидетельствует число 2, переданное ей в качестве первого аргумента). Функция `print` выводит результат на экран. Не обращайте внимания на синтаксические непонятности этого кода. В последующих главах они будут подробно разъяснены.

Весь фокус в том, что функция `replicate` создаст список вовсе не из ста адресов, а всего из двух. Почему? Потому что именно столько понадобилось функции `take`.

Функция `replicate` - лентяйка. Несмотря на то, что мы попросили её создать список из 100 строк, она смотрит по сторонам и думает: "Так-с, кому тут нужны мои строки? Ага, функции `take` нужны. И сколько же ей нужно? А-а, всего две. Ну так а чего я, глупая что ли, создавать сто строк, когда требуется всего две?! Вот тебе две строки и будь счастлива!"

Да, трудолюбие - это хорошо, а лень - это плохо, однако в данном случае мне более симпатична функция-лентяйка. Она, как хороший рационализатор, делает не столько, сколько её попросили, а столько, сколько реально понадобилось. В этом и заключается суть ленивых вычислений в Haskell.

Разумеется, если аппетиты функции `take` возрастут и она попросит первые пятьдесят элементов вместо первых двух, то функция `replicate` создаст список уже из пятидесяти строк. Столько, сколько нужно, и ни капли больше.

Да, но откуда мы можем знать, что функция `replicate`  создаёт лишь столько IP-адресов, сколько потребовалось? А вдруг я вас обманываю? Давайте проверим.

Ленивость языка Haskell позволяет нам оперировать бесконечно большими списками. Нет, не просто очень большими, но именно бесконечными. Перепишем наш пример:
```haskell
main = print (take 2 (repeat "127.0.0.1"))
```
Функция `repeat` создаст бесконечно большой список IP-адресов, элементами которого будет переданный ей адрес `127.0.0.1`. Да-да, бесконечно много одинаковых адресов. И вот если бы наша трудолюбивая функция `generate_addresses` из C++ захотела стать похожей на свою ленивую коллегу из Haskell, ей пришлось бы стать примерно такой:
```cpp
IPAddresses generate_addresses( size_t how_many,
                                const std::string& address ) {
    IPAddresses addresses;
    for(;;) {
        addresses.push_back( address );
    }

    return addresses;
}
```
И всё бы хорошо, но это намертво зависнет. И причиной тому служит уже известное нам трудолюбие функции `generate_addresses`. Сказали ей создать бесконечно большой список - будет создавать до последнего вздоха, ведь цикл `for` в данном случае не имеет выхода.

Однако если мы соберём наш Haskell-проект и запустим его, то не будет никакого зависания, и на экран вновь выведутся уже знакомые нам два адреса. А всё потому, что функция `repeat` столь же ленива и рациональна, как и её коллега `replicate`. Да, мы попросили её создать бесконечно большой список, однако на деле она создаст список вовсе не бесконечный, а такой, который реально нужен. И если в данном случае нужен список только из двух строк - получите список из двух строк. Конечно, если бы потребовался список из миллиона строк - извольте, будет вам миллион.

## Аргументы

Ленивые вычисления распространяются и на работу с аргументами функции. И это очень полезное свойство Haskell.

Вернёмся к C++. Пусть у нас есть функция, принимающая два аргумента. И в качестве этих двух аргументов используются уже известные нам списки IP-адресов:

```cpp
void take_and_print_two_lists( const IPAddresses& first_list,
                               const IPAddresses& second_list ) {
    for( auto ip_address : first_list ) {
        std::cout << ip_address << std::endl;
    }

    for( auto ip_address : second_list ) {
        std::cout << ip_address << std::endl;
    }
} 

int main() {
    take_and_print_two_lists( generate_addresses( 1000, "127.0.0.1" ),
                              generate_addresses( 2000, "22.34.47.65" ) );
}
```

Итак, мы дважды вызываем функцию `generate_addresses`, в итоге в качестве первого аргумента функция `take_and_print_two_lists` принимает список из 1000 адресов, а в качестве второго - список из 2000 адресов. В итоге адреса из обоих списков будут выведены на экран.

А теперь представим себе, что печатать адреса из второго списка нам больше не нужно:

```cpp
void take_and_print_two_lists( const IPAddresses& first_list,
                               const IPAddresses& second_list ) {
    for( auto ip_address : first_list ) {
        std::cout << ip_address << std::endl;
    }
} 
```

Второй аргумент уже не используется. Вопрос: будет ли вызвана функция `generate_addresses`, создающая 2000 адресов? Ответ: конечно да! Ведь, как мы выяснили ранее, эта функция прямолинейна и трудолюбива. Сказали создать 2000 адресов - получите. Что же мы имеем в этом случае? Опять-таки лишнюю работу: целых 2000 адресов были созданы, а в итоге это оказалось никому не нужным. А если бы их было 10 миллионов?..

В языках, похожих на C++, все аргументы вычисляются *до* их передачи в тело функции, независимо от того, используются ли значения этих аргументов или нет. В Haskell же аргументы вычисляются только в том случае, если они реально нужны в теле функции. Поэтому в Haskell функция `generate_addresses`, создающая 2000 адресов, не будет вызвана, ведь результат её работы никому не понадобился. Да здравствуют рационализм и оптимизация!

А, кстати, вдруг я пошутил? Откуда мы знаем, что аргумент не будет вычислен в случае его неиспользуемости? А это очень легко проверить.

В Haskell, как и в любом языке программирования, нельзя делить на 0. Воспользуемся же этим опасным трюком. Откроем файл `Main.hs` и напишем в нём следующее:

```haskell
f x y = x + y

main = print (f (2 `div` 1) (3 `div` 0))
```

Функция `f` принимает два аргумента и возвращает их сумму. И вот мы передаём ей в качестве первого аргумента результат деления 2 на 1, а в качестве второго - результат деления 3 на 0. Эта программа скомпилируется, но если вы её запустите, получите ожидаемую ошибку выполнения:

```bash
$ ./dist/build/Real/Real
Real: divide by zero
```

Функция `f` использует оба свои аргумента, а значит, оба выражения, результат которых будет передан в качестве этих аргументов, будут вычислены, и мы поймаем ошибку деления на 0. А теперь давайте чуток перепишем функцию `f`:

```haskell
f x y = x
```

Теперь эта функция просто возвращает значение своего первого аргумента, абсолютно игнорируя второй. Эта программа тоже скомпилируется, но при запуске мы увидим:

```bash
$ ./dist/build/Real/Real
2
```

Всё верно, перед нами первый аргумент функции `f`. Но вы спросите, куда же делась ошибка деления на 0? А никакого деления на 0 не было. Ведь второй аргумент функции `f` не используется, а следовательно, результат выражения:

```haskell
(3 `div` 0)
```

никому не понадобился. Ну а раз он никому не понадобился, то и вычислено это выражение не будет.

## Строгость

Как было упомянуто выше, ленивость вычислений - это *умолчальное* поведение в Haskell. Однако иногда нам нужна строгость. Поэтому существуют способы избежать ленивого рационализма и действовать напролом. Но об этом будет рассказано в одной из следующих глав.
