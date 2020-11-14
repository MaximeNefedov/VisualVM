# Задача "Понимание JVM"
## Код для исследования
```java

public class JvmComprehension {

    public static void main(String[] args) {
        int i = 1;                      // 1
        Object o = new Object();        // 2
        Integer ii = 2;                 // 3
        printAll(o, i, ii);             // 4
        System.out.println("finished"); // 7
    }

    private static void printAll(Object o, int i, Integer ii) {
        Integer uselessVar = 700;                   // 5
        System.out.println(o.toString() + i + ii);  // 6
    }
}

```
# Описание

Происходит обращение к подсистеме загрузчиков классов (Application Classloader, Platform Classloader, Bootstrap Classloader) подгрузить классы Object, Integer, System, JvmComprehension и т.д.  Процесс осуществляется путем использования связки команд find / load к каждому Classloader`у. В случае невыполнения подгрузки какого-то из классов возникнет ClassNotFoundException. Далее происходит связывание (проверка валидности кода, подготовка примитивов и статических полей, а также связывание ссылок на другие классы) и инициализация статических полей класса.
Классы подгружаются в оперативную память, а точнее происходит деление выделенной для JVM памяти на 3 блока: Stack Memory(цепочка фреймов), Heap(выделение памяти под созданные объекты), Metaspace(данные о классе и константы).

![image](https://raw.githubusercontent.com/MaximeNefedov/VisualVM/master/screenshoots/dio.jpg)

На схеме выше продемонстрирована работа с областями памяти вплоть до 6 - й строки включительно. 

Когда метод заканчивает выполнение, соответствующий кадр стека очищается, поток возвращается к вызывающему методу, и пространство становится доступным для следующего метода.

Следовательно, при выполнении сроки 7 произойдет следующее:

![image2](https://raw.githubusercontent.com/MaximeNefedov/VisualVM/master/screenshoots/dio2.jpg)

У объекта Integer uselessVar = 700 не останется активных ссылок и вскоре сборщик мусора уничтожит данный объект. То же самое произойдет и с объектом типа String, созданным для вывода в консоль сторокового предствления экземпляра класса Object. 
Объекты: 
```java
        int i = 1;                      // 1
        Object o = new Object();        // 2
        Integer ii = 2;                 // 3

```
продолжат свое существавание вплоть до конца выполнения метода main(), посколько ссылки на них до сих пор существуют.

Вызов метода:

```java
   System.out.println("finished"); // 7     

```
приведет к созданию нового фрейма, как и показано на второй схеме.