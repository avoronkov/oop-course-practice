Семинар 1 (04.09.2015)
======================

Задача
------

Рассмотрим следующую задачу. Деканат каждый день присылает отчёт по прошедшим занятиям в текстовом файле, имеющим следующий формат:
каждая строка содержит следующую информацию:
` <предмет>:<номер группы>:<фамилия>:<оценка>:<присутствие> `

Например, файл *Report\_2015\_09\_04.txt*:
```
OOP_seminar:12345:Ivanov:Good:attend
OOP_seminar:12345:Petrov:None:absent
OOP_seminar:54321:Smith:Exсellent:attend
```

Уже к середине семестра таких файлов накапливается достаточно много и анализировать их "вручную" становится затруднительно.
Допустим, некоторый студент хочет извлечь информацию только о себе. Как он может это сделать?

Программа grep
--------------

Напишем небольшую программу, которая называется grep. Программа делает следующее:

* в качестве единственного аргумента grep принимает pattern.

* в процессе работа grep считывает строки со стандартного потока ввода (stdin) и печатает на стандартный поток вывода те, которые содержат pattern.

( *Дополнительное задание 1* : написать решение на языке C. )

Ниже приведено решение на языке C++:
```C++
#include <iostream> // A
#include <string> // B

int main(int argc, char **argv) {
	if (argc < 2) {
		return 2; // C
	}
	std::string line; // D
	while (std::getline(std::cin, line)) { // E
		if (line.find(argv[1]) != std::string::npos) { // F
			std::cout << line << std::endl; // G
		}
	}
	return 0;
}
```

Рассмотрим подробнее:

* _Строка A_: подключение заголовочного файла (header'а) `iostream` из стандартной библиотеки C++.
  `iostream` содержит определения классов и функций для работы с вводом/выводом. Замечание: header'ы стандартной
  библиотеки С++ не имеют расширения ".h", в отличии от C.

* _Строка B_: подключение header'а `string` для работы со строками. Документацию по работе со строками можно найти
  [здесь](http://www.cplusplus.com/reference/string/string/).

* _Строка C_: ненулевой код возврата в случае неправильных входных данных (отсутствует аргумент pattern)

* _Строка D_: объявление переменной `line` типа `std::string`. `std` - название пространства имён (namespace). 
  Все классы и методы стандартной библиотеки C++ лежат внутри пространства имён `std`.

* _Строка E_: здесь много всего и сразу :)

    * `std::getline` - это просто функция, объявленная в пространстве имён `std`. Фактически, ничем не отличается
    от функций C, за исключением префикса `std::`. Замечание: не всё в C++ есть классы и объекты, иногда встречаются и обычные функции.
	Документация к функции [std::getline](http://www.cplusplus.com/reference/string/string/getline/).

    * `std::cin` и `line` - аргументы функции `std::getline`.

    * `std::cin` - стандартный поток ввода в C++ (аналог `stdin` из C).

    * Функция `std::getline` считывает строку, заканчивающуюся символом конца строки, из потока ввода,
	и сохраняет её в переменной, переданной вторым аргументом.
    _Важное замечание_: переменная `line` передаётся в функцию не по значению, а "по ссылке". Поэтому `std::getline` может модифицировать
	содержимое строки `line`.

    * Можно заметить, что `geline` возвращает не булевое значение `true` или `false`, а поток ввода, переданный первым аргументов.
    Каким же образом поток ввода может использоваться в качестве условия `while`? Если коротко, то у класса	`std::istream` перегружен
	оператор приведения к типу `bool`, и этот оператор возвращает `true` в случае, если из потока ещё можно что-то считать.
	Документация: [раз](http://www.cplusplus.com/reference/istream/istream/), [два](http://www.cplusplus.com/reference/ios/ios/operator_bool/).

* _Строка F_:

    * `line.find(...)` - вызов метода `find` объекта `line`. В целом, class'ы в С++ являются "продвинутыми" struct'ами из C.
    Вызов метода осуществляется аналогично доступу к полю структуры C: через `.` для объектов и через `->` для указателей на объекты.
	Методы, в свою очередь, весьма похожи на обычные функции, но, в дополнение, они имеют доступ к содержимому объекта, для которого они вызваны.
	В C аналогичная запись могла бы выглядеть так: `find(str, pattern)`.

    * В случае, если подстрока `argv[1]` не найдена внутри строки `line`, метод `find` вернёт специальное значение `std::string::npos`.
    Кстати, писать `line.find(argv[1]) >= 0` нельзя. Почему?

* _Строка G_:

    * `std::cout` - стандартный поток вывода (по аналогии с `std::cin`).

    * `<<` - оператор "записи в поток" (перегруженный для потоков вывода оператор побитового сдвига). Записывает значение справа в поток слева.

    * `std::endl` - символ конца строки ("\n").

    * Важно понимать (и уметь объяснять :) ), почему можно писать `std::cout << line << std::endl`. Подсказки:

        * Исходная запись аналогична следующей: `(std::cout << line) << std::endl`

	    * Оператор `<<` возвращает поток вывода

*Самостоятельно* : скомпилировать, проверить, что программа работает правильно.

*Дополнительное задание 2* : сравнить исходные коды программ на C и C++.

Программа cat
-------------

Программа выводит на экран содержимое файлов, указанных в аргументах командной строки. (К котам не имеет никакого отношения:
"cat" - это сокращение от "catenate".)
Решение:
```C++
#include <iostream>
#include <string>
#include <fstream> // A

void catFile(std::string filename) { // B
	std::ifstream f(filename); // C
	std::cout << f.rdbuf(); // D
}

int main(int argc, char **argv) {
	for (int i=1; i<argc; i++) {
		catFile(argv[i]); // E
	}
	return 0;
}
```

Новое в программе cat:

* _Строка A_: подключение header'а fstream для работы с файловыми потоками ввода/вывода.

* _Строка B_: определение функции `catFile`. В качестве аргумента функция принимает объект класса `std::string`.

* _Строка C_: объявление и "конструирование" объекта `f` класса `std::ifstream` (= input file stream - файловый поток ввода).
  Конструкторы - это специальные "методы" для инициализации объектов (не совсем "методы", так как имеются некоторые важные отличия).
  В классе может быть объявлено несколько конструкторов, различающихся набором аргументов. Документация по конструкторам 
  [std::ifstream](http://www.cplusplus.com/reference/fstream/ifstream/ifstream/).

* _Строка D_: запись содержимого входного потока `f` в выходной поток `std::cout`.
  (Рекомендую посмотреть документацию, чтобы лучше понять, что произошло).

* _Строка Е_: вызов функции `catFile`. Следует отметить, что аргумент типа `char*` неявно приводится к типу `std::string`.

Самостоятельно: скомпилировать, проверить, что программа работает правильно.

Консольные приложения и unix-way
--------------------------------

Поскольку мы создаём консольные приложения, хотелось бы показать один трюк, связанный с консолью.
Для простоты скопируйте файлы с исходным кодом и скомпилированные исполняемые файлы в одну директорию.
Откройте консоль и перейдите в эту директорию. (В Windows я бы воспользовался файловым менеджером Far - в нём удобнее перемещаться по директориям,
и можно запускать консольные программы с аргументами). Давайте посмотрим, как часто встречается слово "string" в исходных файлах наших программ.
Для этого выполним в консоли команду:

* для windows:
` .\cat.exe cat.cpp grep.cpp | .\grep.exe string `

* для unix:
` ./cat cat.cpp grep.cpp | ./grep string `

В результате на экран должно быть выведено приблизительно следующее:
```
#include <string>
void catFile(std::string filename) {
#include <string> // B
        std::string line; // D
                if (line.find(argv[1]) != std::string::npos) { // F
```

Что произошло? Конструкция `|` в консоли называется pipe ("труба"). Pipe позволяет соединять две программы так,
что stdout первой программы перенаправляется в stdin второй программы. Можно использовать несколько "труб" в одной команде:
```
command1 args1 | command2 args2 | command3 args3 | command4 args4
```

*Дополнительное задание 3* : написать программу count, которая считает количество строк, которое ей подаётся на стандартный поток ввода.
Показать, как с помощью этой программы, а также программ cat и grep можно посчитать количество пропусков некоторого студента
имея файлы с отчётами о студентах, описанные в начале документа.