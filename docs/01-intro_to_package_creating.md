# Обзор рабочего процесса разработки пакета
В этом уроке мы поверхностно разберём весь процесс разработки пакета.

## Видео
<iframe width="560" height="315" src="https://www.youtube.com/embed/3t2lbIQNQf8" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

### Тайм коды

00:00 Вступление<Br>
00:43 Как создать проект пакета<Br>
01:50 Структура проекта пакета<Br>
02:47 Добавляем проекту контроль версий<Br>
03:14 Как добавить функцию в свой пакет<Br>
04:17 Как загрузить текущий пакет (`load_all()`)<Br>
05:05 Как запустить проверку пакета (`check()`)<Br>
05:56 Файл DESCRIPTION<Br>
06:52 Добавляем лицензию пакету<Br>
07:08 Добавляем документацию к функциям пакета<Br>
09:47 Файл NAMESPACE<Br>
10:10 Добавляем юнит тесты для функций пакета<Br>
13:34 Как использовать в своём пакете функции из других пакетов<Br>
16:46 Обзор всего рабочего процесса<Br>
19:03 Заключение<Br>

## Презентация
<iframe src="https://www.slideshare.net/slideshow/embed_code/key/jRKxyjUM5qUrYe?hostedIn=slideshare&page=upload" width="476" height="400" frameborder="0" marginwidth="0" marginheight="0" scrolling="no"></iframe>

## Краткий конспект
### Вспомогательные пакеты

1. `devtools` - инструменты разработчика пакета
2. `usethis` - автоматизация настройки разрабатываемого пакета
3. `testthat` - разработка юнит тестов к функциям пакета
4. `roxygen2` - упрощённое написание документации к функциям пакета

### Процесс разработки пакета

1. Создаём проект пакета с помощью команды `create_package()`
2. Включаем контроль версий с помощью функции `use_git()`
3. Добавляем лицензию командой `use_mit_licence()`
4. Добавляем в пакет функции с помощью команды `use_r()`
5. Документируем созданные функции добавляя специализированиы комментарии сочетанием клавиш **Ctrl+Alt+Shift+R**
6. генерируем файлы документации функций командой `document()`
7. Для тестирования добавляем в файл DESCRIPTION пакет `testthat` командой `use_testthat()`
8. Добавляем для каждой функции юнит тесты командой `use_test()`
9. Запускаем тестирование функций командой `test()`
10. Для использования функций из стороних пакетов добавляем их в блок Imports файлв DESCRIPTION командой `use_package()`, в коде используем импортированные функции с помощью `package_name::function_name()`
11. Проверяем пакет командой `check()`

## Тест
<iframe id="otp_wgt_3q2sbgk6kmico" src="https://onlinetestpad.com/3q2sbgk6kmico" frameborder="0" style="width:100%;" onload="var f = document.getElementById('otp_wgt_3q2sbgk6kmico'); var h = 0; var listener = function (event) { if (event.origin.indexOf('onlinetestpad') == -1) { return; }; h = parseInt(event.data); if (!isNaN(h)) f.style.height = h + 'px'; }; function addEvent(elem, evnt, func) { if (elem.addEventListener) { elem.addEventListener(evnt, func, false); } else if (elem.attachEvent) { elem.attachEvent('on' + evnt, func); } else { elem['on' + evnt] = func; } }; addEvent(window, 'message', listener);" scrolling="no">
</iframe>
