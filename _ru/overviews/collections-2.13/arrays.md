---
layout: multipage-overview
title: Массивы

discourse: true

partof: collections-213
overview-name: Collections

num: 10
previous-page: concrete-mutable-collection-classes
next-page: strings
language: ru

---

[Массивы](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/Array.html) особый вид коллекций в Scala. 
С одной стороны, Scala массивы соответствуют массивам из Java. Например, Scala массив `Array[Int]`  реализован в виде Java `int[]`, а `Array[Double]` как Java `double[]` и `Array[String]` как Java `String[]`
 С другой стороны, Scala массивы дают намного больше чем их Java аналоги. Во-первых Scala массивы могут быть обобщены (_generic_). То есть вы можете описать массив как `Array[T]`, где `T` дополнительный `параметр-тип` массива или же абстрактный тип. 
 Во-вторых, Scala массивы совместимы со списками (`Seq`) Scala - вы можете передавать `Array[T]` на вход туда, где требуется `Seq[T]`. Ну и наконец, Scala массивы также поддерживают все операции, которые есть у списков. Вот пример:

    scala> val a1 = Array(1, 2, 3)
    a1: Array[Int] = Array(1, 2, 3)
    scala> val a2 = a1 map (_ * 3)
    a2: Array[Int] = Array(3, 6, 9)
    scala> val a3 = a2 filter (_ % 2 != 0)
    a3: Array[Int] = Array(3, 9)
    scala> a3.reverse
    res0: Array[Int] = Array(9, 3)

Учитывая то что Scala массивы соответствуют массивам из Java, каким же образом реализованы остальные дополнительные возможности массивов в Scala? 
Реализация массивов в Scala постоянно использует неявные преобразования. В Scala массив не пытается _притворяться_ последовательностью. Он и не может, потому что тип данных лежащий в основе массива не является подтипом `Seq`. Вместо этого, используя "упаковывание", происходит неявное преобразование между массивами и экземплярами класса `scala.collection.mutable.ArraySeq`, который является подклассом `Seq`. Вот как это работает:

    scala> val seq: collection.Seq[Int] = a1
    seq: scala.collection.Seq[Int] = ArraySeq(1, 2, 3)
    scala> val a4: Array[Int] = seq.toArray
    a4: Array[Int] = Array(1, 2, 3)
    scala> a1 eq a4
    res1: Boolean = false

Пример выше показывает, что массивы совместимы с последовательностями, потому как происходит неявное преобразование из массивов в `ArraySeq`ы. Чтобы перейти обратно от `ArraySeq` к `Array`, можно использовать метод `toArray`, описанный в `Iterable`. Последняя строка в консоле показывает, что упаковка и затем распаковка с помощью `toArray` создает копию исходного массива.

Существует еще одно неявное преобразование, которое применяется к массивам. Такое преобразование просто "добавляет" все методы последовательностей (`Seq`) к массивам, но не превращает сам массив в последовательность. "Добавление" означает, что массив обернут в другой объект типа `ArrayOps`, который поддерживает все методы последовательности. Объект `ArrayOps` недолговечный, обычно он недоступен после обращения к методу последовательности и он может быть удален. Современные виртуальные машины могут избегать создания такого промежуточного объекта.

Разница между двумя неявными преобразованиями на массивах показана в следующем примере:

    scala> val seq: collection.Seq[Int] = a1
    seq: scala.collection.Seq[Int] = ArraySeq(1, 2, 3)
    scala> seq.reverse
    res2: scala.collection.Seq[Int] = ArraySeq(3, 2, 1)
    scala> val ops: collection.ArrayOps[Int] = a1
    ops: scala.collection.ArrayOps[Int] = scala.collection.ArrayOps@2d7df55
    scala> ops.reverse
    res3: Array[Int] = Array(3, 2, 1)

Вы видите, что вызов `reverse` на `seq`, который является `ArraySeq`, даст снова `ArraySeq`. Это логично, потому что массивы - это `Seqs`, и вызов `reverse` на любом `Seq` даст снова `Seq`. С другой стороны, вызов `reverse` на экземпляре класса `ArrayOps` даст значение `Array`, а не `Seq`.

Пример `ArrayOps`, приведенный выше искусственный и используется лишь, чтоб показать разницу с `ArraySeq`. Обычно, вы никогда не создаете экземпляры класса `ArrayOps`. Вы просто вызываете методы `Seq` на массиве:

    scala> a1.reverse
    res4: Array[Int] = Array(3, 2, 1)

Объект `ArrayOps` автоматически вставляется через неявное преобразование. Так что строка выше эквивалентна

    scala> intArrayOps(a1).reverse
    res5: Array[Int] = Array(3, 2, 1)

где `intArrayOps` - неявное преобразование, которое было вставлено ранее. В связи с этим возникает вопрос, как компилятор выбрал `intArrayOps` вместо другого неявного преобразования в `ArraySeq` в строке выше. В конце концов, оба преобразования преобразуют массив в тип, поддерживающий метод reverse. Ответ на этот вопрос заключается в том, что два неявных преобразования имеют приоритет. Преобразование `ArrayOps` имеет больший приоритет, чем преобразование `ArraySeq`. Первый определяется в объекте `Predef`, а второй - в классе `scala.LowPriorityImplicits`, который `Predef` наследует. Неявные преобразования в дочерних классах и дочерних объектах имеют более высокий приоритет над преобразованиями в базовых классах. Таким образом, если оба преобразования применимы, выбирается вариант в `Predef`. Очень похожая схема используется для строк.

Итак, теперь вы знаете, как массивы могут быть совместимы с последовательностями и как они могут поддерживать все операции последовательностей. А как же обобщения? В Java нельзя написать `T[]`, где `T` является параметром типа. Как же представлен Scala `Array[T]`? На самом деле обобщенный массив типа `Array[T]` может быть любым из восьми примитивных типов массивов Java `byte[]`, `short[]`, `char[]`, `int[] `, `long[] `, `float[]`, `double ` или может быть массивом объектов. Единственным общим типом, включающим все эти типы, является `AnyRef` (или, равнозначно `java.lang.Object`), так что это тот тип в который компилятор Scala отобразит `Array[T]`. Во время исполнения, при обращении к элементу массива типа `Array[T]`, происходит последовательность проверок типов, которые определяют тип массива, за которыми следует подходящая операция на Java-массиве. Эти проверки типов замедляют работу массивов. Можно ожидать падения скорости доступа к обобщенным массивам в три-четыре раза, по сравнению с обычными массивами или массивами объектов. Это означает, что если вам нужна максимальная производительность, вам следует выбирать конкретные массивы, вместо обобщенных. Отображать обобщенный массив еще пол беды, нам нужен еще способ создания обобщенных массивов. Это куда более сложная задача, которая требует, от вас, небольшой помощи. Чтобы проиллюстрировать проблему, рассмотрим следующую попытку написания обобщенного метода, который создает массив.

    // это неправильно!
    def evenElems[T](xs: Vector[T]): Array[T] = {
      val arr = new Array[T]((xs.length + 1) / 2)
      for (i <- 0 until xs.length by 2)
        arr(i / 2) = xs(i)
      arr
    }

Метод `evenElems` возвращает новый массив, состоящий из всех элементов аргумента вектора `xs`, находящихся в четных позициях вектора. В первой строке тела `evenElems` создается результирующий массив, который имеет тот же тип элемента, что и аргумент. Так что в зависимости от фактического типа параметра для `T`, это может быть `Array[Int]`, или `Array[Boolean]`, или массив некоторых других примитивных типов Java, или массив какого-нибудь ссылочного типа. Но эти типы имеют разные представления при исполнении программы, и как же Scala подберет правильное представление? В действительности, Scala не может сделать этого, основываясь на предоставленной информации, так как при выполнении стирается фактический тип, соответствующий параметру типа `T`. Поэтому при компиляции показанного выше кода, появится следующее сообщение об ошибке:

    error: cannot find class manifest for element type T
      val arr = new Array[T]((arr.length + 1) / 2)
                ^

Тут нужно немного помочь компилятору, указав какой в действительности тип параметра `evenElems`. Это указание во время исполнения принимает форму манифеста класса типа `scala.view.ClassTag`. _Манифест класса_ - это объект дескриптор типа, который описывает, какой тип у класса верхнего уровня. В качестве альтернативы манифестам классов существуют также _полные манифесты_ типа `scala.Refect.Manifest`, которые описывают все аспекты типа. Впрочем для создания массива требуются только _манифесты класса_.

Компилятор Scala автоматически создаст манифесты классов, если вы проинструктируете его на это. "Инструктирование" означает, что вы требуете манифест класса в качестве неявного параметра, как в примере:

    def evenElems[T](xs: Vector[T])(implicit m: ClassTag[T]): Array[T] = ...

Используя альтернативный и более короткий синтаксис, вы также можете потребовать, чтобы тип приходил с манифестом класса, используя _контекстное связывание_ (`context bound`). Это означает установить связь с типом `ClassTag` идущим после двоеточия в описании типа, как в примере:

    import scala.reflect.ClassTag
    // так будет работать
    def evenElems[T: ClassTag](xs: Vector[T]): Array[T] = {
      val arr = new Array[T]((xs.length + 1) / 2)
      for (i <- 0 until xs.length by 2)
        arr(i / 2) = xs(i)
      arr
    }

Обе показанные версии `evenElems` означают одно и то же. Что бы не случилось, когда построен `Array[T]`, компилятор будет искать манифест класса для параметра типа `T`, то есть искать неявное значение (implicit value) типа `ClassTag[T]`. Если такое значение найдено, то этот манифест будет использоваться для построения требуемого типа массива. В противном случае вы увидите сообщение об ошибке, такое же как мы показывали выше.

Вот некоторые примеры из консоли, использующие метод `evenElems`.

    scala> evenElems(Vector(1, 2, 3, 4, 5))
    res6: Array[Int] = Array(1, 3, 5)
    scala> evenElems(Vector("this", "is", "a", "test", "run"))
    res7: Array[java.lang.String] = Array(this, a, run)

В обоих случаях компилятор Scala автоматически построил манифест класса для типа элемента (сначала `Int`, затем `String`) и передал его в качестве неявного параметра метода `evenElems`. Компилятор может сделать это для всех конкретных типов, но не тогда, когда аргумент сам параметризован типом, который не содержит манифест класса. Например, следующий пример не скомпилируется:

    scala> def wrap[U](xs: Vector[U]) = evenElems(xs)
    <console>:6: error: No ClassTag available for U.
         def wrap[U](xs: Vector[U]) = evenElems(xs)
                                               ^
В данном случае `evenElems` требует наличия класса манифеста для параметра типа `U`, однако ни одного не найдено. Чтоб решить такую проблему, конечно, необходимо запросить манифест от неявного класса `U`. Поэтому следующее будет работать:

    scala> def wrap[U: ClassTag](xs: Vector[U]) = evenElems(xs)
    wrap: [U](xs: Vector[U])(implicit evidence$1: scala.reflect.ClassTag[U])Array[U]

Этот пример также показывает, что контекстное связывание с `U`, является лишь сокращением для неявного параметра, названного здесь `evidence$1` типа `ClassTag[U]`.

Подводя итог, можно сказать, что для создания обобщенных массивов требуются манифесты классов. Поэтому при создании массива параметризированного типом `T`, вам также необходимо предоставить неявный класс манифест для `T`. Самый простой способ сделать это - объявить параметр типа `ClassTag` с контекстной привязкой, как `[T: ClassTag]`.