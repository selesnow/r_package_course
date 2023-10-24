# (PART) Примеры пакетов {-}
# Разработка пакета обёртки над API (пакет httr2)

------

В этом видео мы разберёмся с тем, зачем покрывать код вашего пакета юнит-тестам, и как технически это реализовать.

------

::: {style="border: 2px solid #4682B4; background: #EEE8AA; padding: 15px; border-radius: 9px;"}
*Данный урок основан на документации к пакету httr2:*

* Статья ["httr2"](https://httr2.r-lib.org/articles/httr2.html)
* Статья ["Wrapping APIs
"](https://httr2.r-lib.org/articles/wrapping-apis.html)
:::

------

## Видео
<iframe width="560" height="315" src="https://www.youtube.com/embed/ktPGh7HY8Tg?si=XS5PXrZry2fzxOwa&enablejsapi=1" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

### Тайм коды
00:00 Вступление<Br>
00:45 Что такое API<Br>
01:49 Компоненты HTTP запросов и ответов<Br>
03:26 Введение в пакет httr2<Br>
08:04 Функции пакета httr2<Br>
09:36 Этапы работы с API<Br>
10:10 Простейший пример обёртки над Faker API<Br>
18:44 Управление конфиденциальными данными<Br>
29:33 Пример обёртки над NYTimes Books API<Br>
30:51 Обработка ошибок в HTTP ответах<Br>
34:06 Ограничение скорости отправки запросов<Br>
37:18 Как работать с API токена в пакетах-обёртках над API<Br>
39:32 Протокол OAuth<Br>
41:00 Пример обёртки над Facebook API<Br>
49:46 Обзор всего рабочего процесса<Br>
52:11 Заключение<Br>

## Презентация
<iframe src="https://www.slideshare.net/slideshow/embed_code/key/gRkzcDqIM23tw0?hostedIn=slideshare&page=upload" width="476" height="400" frameborder="0" marginwidth="0" marginheight="0" scrolling="no"></iframe>

## Конспект
### Построение HTTP запроса

Работа с `httr2` начинается с создания HTTP запроса. Это ключевое отличие от предшественника `httr`, в предыдущей версии вы одной командой выполняли сразу несколько действий: создавали запрос, отправляли его, и получали ответ. httr2 имеет явный объект запроса, что значительно упрощает процесс компоновки сложных запросов. Процесс построения запроса начинается с базового URL:


```r
req <- request("https://httpbin.org/get")
req
#> <httr2_request>
#> GET https://httpbin.org/get
#> Body: empty
```

Перед отправкой запроса на сервер мы можем посмотреть, что именно будет отправлено:


```r
req %>% req_dry_run()
#> GET /get HTTP/1.1
#> Host: httpbin.org
#> User-Agent: httr2/0.1.1 r-curl/4.3.2 libcurl/7.64.1
#> Accept: */*
#> Accept-Encoding: deflate, gzip
```

Первая строка содержит три важных составляющих запроса

* HTTP метод, т.е. глагол, который сообщает серверу, какое действие должен выполнить ваш запрос. По умолчанию подразумевается метод GET, самый распространенный метод, указывающий, что мы хотим получить ресурс от сервера. Другие так же есть и другие HTTP методы: POST, для создания ресурса, PUT, для изменения ресурса, и DELETE, для его удаления.
* Путь, URL адрес сервера, который состоит из: протокола (httpили https), хоста (httpbin.org), и порта (в нашем примере не использовался).
* Версия протокола HTTP. В данном случае эта информация нам не важна, т.к. обработка протокола идёт на более низком уровне.

Далее идут заголовки запроса. В заголовках зачастую передаётся некоторая служебная информация, представленная в виде пар ключ-значение, разделенных знаком :. Заголовки в нашем примере были автоматически добавлены `httr2`, но вы можете переопределить их или добавить свои с помощью `req_headers()`:


```r
req %>%
 req_headers(
 Name = "Hadley", 
 `Shoe-Size` = "11", 
 Accept = "application/json"
 ) %>% 
 req_dry_run()
#> GET /get HTTP/1.1
#> Host: httpbin.org
#> User-Agent: httr2/0.1.1 r-curl/4.3.2 libcurl/7.64.1
#> Accept-Encoding: deflate, gzip
#> Name: Hadley
#> Shoe-Size: 11
#> Accept: application/json
```

Имена заголовков не чувствительны к регистру, и сервера игнорируют неизвестные им заголовки.

Заголовки заканчиваются пустой строкой, за которой следует тело запроса. Приведённые выше запросы (как и все GET запросы) не имеют тела, поэтому давайте добавим его, чтобы посмотреть, что произойдет. функции семейства `req_body_*()` обеспечивают различные способы добавить данные к телу запроса. В качестве примера мы используем `req_body_json()` для добавления данных в виде JSON структуры:


```r
req %>%
 req_body_json(list(x = 1, y = "a")) %>% 
 req_dry_run()
#> POST /get HTTP/1.1
#> Host: httpbin.org
#> User-Agent: httr2/0.1.1 r-curl/4.3.2 libcurl/7.64.1
#> Accept: */*
#> Accept-Encoding: deflate, gzip
#> Content-Type: application/json
#> Content-Length: 15
#> 
#> {"x":1,"y":"a"}
```

Что изменилось?

* Метод запроса автоматически изменился с GET на POST. POST - это стандартный метод отправки данных на веб-сервер, который автоматически используется всякий раз, когда вы добавляете тело запроса. Вы можете использовать `req_method()` для переопределения метода.
* К запросу добавлены два новых заголовка: Content-Type и Content-Length. Они сообщают серверу, как интерпретировать тело - в нашем случае это JSON структура размером 15 байт.
* У запроса есть тело, состоящее из какого-то JSON.

Разные API могут требовать различных вариантов кодировки тела запроса, поэтому httr2 предоставляет семейство функций, для реализации наиболее часто встречающихся форматов. Например, `req_body_form()` преобразует тело запроса, в вид отправляемой браузером формы:


```r
req %>%
 req_body_form(list(x = "1", y = "a")) %>% 
 req_dry_run()
#> POST /get HTTP/1.1
#> Host: httpbin.org
#> User-Agent: httr2/0.1.1 r-curl/4.3.2 libcurl/7.64.1
#> Accept: */*
#> Accept-Encoding: deflate, gzip
#> Content-Type: application/x-www-form-urlencoded
#> Content-Length: 7
#> 
#> x=1&y=a
```

Для отправки данных большого объёма или бинарных файлов используйте `req_body_multipart()`:


```r
req %>%
 req_body_multipart(list(x = "1", y = "a")) %>% 
 req_dry_run()
#> POST /get HTTP/1.1
#> Host: httpbin.org
#> User-Agent: httr2/0.1.1 r-curl/4.3.2 libcurl/7.64.1
#> Accept: */*
#> Accept-Encoding: deflate, gzip
#> Content-Length: 228
#> Content-Type: multipart/form-data; boundary=------------------------cc86fca72508d8b0
#> 
#> --------------------------cc86fca72508d8b0
#> Content-Disposition: form-data; name="x"
#> 
#> 1
#> --------------------------cc86fca72508d8b0
#> Content-Disposition: form-data; name="y"
#> 
#> a
#> --------------------------cc86fca72508d8b0--
```

Если вам нужно отправить данные, закодированные в другой форме, вы можете использовать `req_body_raw()` для добавления данных в тело и передать тип отправляемых данных в заголовке Content-Type.

### Отправка запроса и обработка ответа

Чтобы фактически выполнить запрос и получить ответ от сервера, используйте функцию  req_perform():


```r
req <- request("https://httpbin.org/json")
resp <- req %>% req_perform()
resp
#> <httr2_response>
#> GET https://httpbin.org/json
#> Status: 200 OK
#> Content-Type: application/json
#> Body: In memory (429 bytes)
```

Посмотреть имитацию полученного ответа можно с помощью `resp_raw()`:


```r
resp %>% resp_raw()
#> HTTP/1.1 200 OK
#> date: Mon, 27 Sep 2021 20:40:32 GMT
#> content-type: application/json
#> content-length: 429
#> server: gunicorn/19.9.0
#> access-control-allow-origin: *
#> access-control-allow-credentials: true
#> 
#> {
#> "slideshow": {
#> "author": "Yours Truly", 
#> "date": "date of publication", 
#> "slides": [
#> {
#> "title": "Wake up to WonderWidgets!", 
#> "type": "all"
#> }, 
#> {
#> "items": [
#> "Why <em>WonderWidgets</em> are great", 
#> "Who <em>buys</em> WonderWidgets"
#> ], 
#> "title": "Overview", 
#> "type": "all"
#> }
#> ], 
#> "title": "Sample Slide Show"
#> }
#> }
```

Структура HTTP ответа очень похожа на структуру запроса. В первой строке указывается версия используемого HTTP и код состояния, за которым (необязательно) следует его краткое описание. Затем идут заголовки, за которыми следует пустая строка, за которой следует тело ответа. В отличие от запросов большинство ответов будет иметь тело.

Вы можете извлечь данные из ответа с помощью функций семейства `resp_()`:

* `resp_status()` возвращает код состояния и `resp_status_desc()` возвращает его описание:


```r
resp %>% resp_status()
#> [1] 200
resp %>% resp_status_desc()
#> [1] "OK"
```

* Вы можете извлечь все заголовки используя `resp_headers()` или получить значение конкретного заголовок с помощью `resp_header()`:


```r
resp %>% resp_headers()
#> <httr2_headers>
#> date: Mon, 27 Sep 2021 20:40:32 GMT
#> content-type: application/json
#> content-length: 429
#> server: gunicorn/19.9.0
#> access-control-allow-origin: *
#> access-control-allow-credentials: true
resp %>% resp_header("Content-Length")
#> [1] "429"
```

Заголовки нечувствительны к регистру:


```r
resp %>% resp_header("ConTEnT-LeNgTH")
#> [1] "429"
```

Тело ответа, так же как и тело запроса, в зависимости от устройства API может приходить в разных форматах. Для извлечения тела ответа используйте функции семейства  `resp_body_*()`. В нашем примере мы получили ответ в виде JSON структуры, поэтому для его извлечения необходимо использовать `resp_body_json()`:


```r
resp %>% resp_body_json() %>% str()
#> List of 1
#> $ slideshow:List of 4
#> ..$ author: chr "Yours Truly"
#> ..$ date : chr "date of publication"
#> ..$ slides:List of 2
#> .. ..$ :List of 2
#> .. .. ..$ title: chr "Wake up to WonderWidgets!"
#> .. .. ..$ type : chr "all"
#> .. ..$ :List of 3
#> .. .. ..$ items:List of 2
#> .. .. .. ..$ : chr "Why <em>WonderWidgets</em> are great"
#> .. .. .. ..$ : chr "Who <em>buys</em> WonderWidgets"
#> .. .. ..$ title: chr "Overview"
#> .. .. ..$ type : chr "all"
#> ..$ title : chr "Sample Slide Show"
```

Ответы с кодами состояния 4xx и 5xx являются ошибками HTTP. `httr2` автоматически преобразует их в ошибки R:


```r
request("https://httpbin.org/status/404") %>% req_perform()
#> Error: HTTP 404 Not Found.
request("https://httpbin.org/status/500") %>% req_perform()
#> Error: HTTP 500 Internal Server Error.
```

Это еще одно важное отличие от httr, который требовал явного вызова `httr::stop_for_status()` для преобразования ошибок HTTP в ошибки R. Вы можете вернуться к поведению `httr` с помощью `req_error(req, is_error = ~ FALSE)`.

### Оборачиваем API с помощью httr2
#### Faker API

Мы начнем с очень простого API, [faker API](https://fakerapi.it/en) , который предоставляет набор методов для генерации случайных выборок данных. Перед тем как приступить к разработке функции, которые вы могли бы поместить в пакет, мы выполним пробный запрос, что бы разобраться с устройством этого API:


```r
# We start by creating a request that uses the base API url
req <- request("https://fakerapi.it/api/v1")
resp <- req %>% 
  # Then we add on the images path
  req_url_path_append("images") %>% 
  # Add query parameters _width and _quantity
  req_url_query(`_width` = 380, `_quantity` = 1) %>% 
  req_perform()

# The result comes back as JSON
resp %>% resp_body_json() %>% str()

#> List of 4
#>  $ status: chr "OK"
#>  $ code  : int 200
#>  $ total : int 1
#>  $ data  :List of 1
#>   ..$ :List of 3
#>   .. ..$ title      : chr "Nisi totam nobis non."
#>   .. ..$ description: chr "Repellendus natus dolore eius in similique est est. Magnam maiores labore est expedita occaecati tenetur excepturi."
#>   .. ..$ url        : chr "http://placeimg.com/380/480/any"
```

##### Основная функция генерации запроса

Сделав несколько успешных запросов к изучаемому API стоит обратить внимание, есть ли какие-нибудь общие паттерны запросов к различным конечным точкам. Делается это с целью разработки основной функции пакета, генерирующей основу HTTP запроса для всех остальных функций.

Немного изучив документацию Faker API я отметил некоторые общие паттерны:

* Каждый URL-адрес имеет форму https://fakerapi.it/api/v1/{resource}, и данные передаются ресурсу с параметрами запроса. Все параметры начинаются с `_`.
* Каждый ресурс имеет три общих параметра запроса: `_locale`, `_quantity` и `_seed`.
* Все конечные точки возвращают данные в виде JSON структуры.

Это привело меня к созданию следующей функции:

```r
faker <- function(resource, ..., quantity = 1, locale = "en_US", seed = NULL) {
  params <- list(
    ...,
    quantity = quantity,
    locale = locale,
    seed = seed
  )
  names(params) <- paste0("_", names(params))
  
  request("https://fakerapi.it/api/v1") %>% 
    req_url_path_append(resource) %>% 
    req_url_query(!!!params) %>% 
    req_user_agent("my_package_name (http://my.package.web.site)") %>% 
    req_perform() %>% 
    resp_body_json()
}

str(faker("images", width = 300))
#> List of 4
#>  $ status: chr "OK"
#>  $ code  : int 200
#>  $ total : int 1
#>  $ data  :List of 1
#>   ..$ :List of 3
#>   .. ..$ title      : chr "Nihil beatae tenetur minus."
#>   .. ..$ description: chr "Provident pariatur iste consequatur enim id neque. Odio blanditiis libero aut. Accusantium ipsam et ex est."
#>   .. ..$ url        : chr "http://placeimg.com/300/480/any"
```

Тут я сделал несколько важных решений:

* Я решил указать значения по умолчанию для параметров `quantity` и `locale.` Это упрощает демонстрацию моей функции в этой статье.
* Я использовал значение по умолчанию `NULL` для аргумента seed . `req_url_query()` автоматически отбрасывает аргументы со значением `NULL`, это означает, что в API не отправляется значение по умолчанию, но когда вы смотрите определение функции, вы видите, что значение seed установлено.
* Я автоматически добавляю ко всем параметрам запроса префикс, `_` т.к. имена параметров в API начинаются с `_`.
* Моя функция генерирует запрос, выполняет его и извлекает тело ответа. Такой подход будет работать в общих случаях с простыми API, для более сложных API возможно вам будет удобнее вернуть объект запроса, который можно изменить перед выполнением.

Я использовал один приём: `req_url_query()` использует динамические точки, поэтому можно использовать `!!!` для их преобразования, например `req_url_query(req, !!!list(`_quantity` = 1, `_locale` = "en_US"))` конвертируется в `req_url_query(req, `_quantity` = 1, `_locale` = "en_US")`.

##### Обёртывание конечных точек

`faker()` является довольно обобщённой функцией — это хороший инструмент для разработчика пакета, т.к. вы можете прочитать документацию Faker API и перевести ее в вызов функции. Но это не очень удобно для пользователя пакета, который может ничего не знать о веб-API, и тем более об особенностях устройства Faker API и параметров вызовов отдельных его методов. Поэтому следующим шагом в процессе разработки пакета - обёртки к API является обертывание отдельных конечных точек их собственными функциями.

Например, возьмем конечную точку persons с тремя дополнительными параметрами: gender (мужчина или женщина),  birthday_start и birthday_end. Простейшая обёртка этой конечной точки будет выглядеть примерно следующим образом:


```r
faker_person <- function(gender = NULL, birthday_start = NULL, birthday_end = NULL, quantity = 1, locale = "en_US", seed = NULL) {
  faker(
    "persons",
    gender = gender,
    birthday_start = birthday_start,
    birthday_end = birthday_end,
    quantity = quantity,
    locale = locale,
    seed = seed
  )  
}
str(faker_person("male"))
#> List of 4
#>  $ status: chr "OK"
#>  $ code  : int 200
#>  $ total : int 1
#>  $ data  :List of 1
#>   ..$ :List of 10
#>   .. ..$ id       : int 1
#>   .. ..$ firstname: chr "Terence"
#>   .. ..$ lastname : chr "Reinger"
#>   .. ..$ email    : chr "brennan.effertz@barton.com"
#>   .. ..$ phone    : chr "+8608217930964"
#>   .. ..$ birthday : chr "2021-06-01"
#>   .. ..$ gender   : chr "male"
#>   .. ..$ address  :List of 10
#>   .. .. ..$ id            : int 0
#>   .. .. ..$ street        : chr "950 Barrows Plains Suite 474"
#>   .. .. ..$ streetName    : chr "Barrows Extensions"
#>   .. .. ..$ buildingNumber: chr "864"
#>   .. .. ..$ city          : chr "North Cicero"
#>   .. .. ..$ zipcode       : chr "39030"
#>   .. .. ..$ country       : chr "Tokelau"
#>   .. .. ..$ county_code   : chr "TD"
#>   .. .. ..$ latitude      : num -57.3
#>   .. .. ..$ longitude     : num -40.4
#>   .. ..$ website  : chr "http://mills.com"
#>   .. ..$ image    : chr "http://placeimg.com/640/480/people"
```

Можно сделать эту функцию ещё более удобной для пользователя, проверив типы ввода и преобразовав полученный результат в таблицу. Я по-быстрому накидал небольшой вариант преобразования полученного ответа в таблицу с использованием функционала пакета purrr; в зависимости от ваших потребностей и предпочтений вы можете использовать для той же операции базовый R или `tidyr::hoist()`.


```r
library(purrr)

faker_person <- function(gender = NULL, birthday_start = NULL, birthday_end = NULL, quantity = 1, locale = "en_US", seed = NULL) {
  if (!is.null(gender)) {
    gender <- match.arg(gender, c("male", "female"))
  }
  if (!is.null(birthday_start)) {
    if (!inherits(birthday_start, "Date")) {
      stop("`birthday_start` must be a date")
    }
    birthday_start <- format(birthday_start, "%Y-%m-%d")
  }
  if (!is.null(birthday_end)) {
    if (!inherits(birthday_end, "Date")) {
      stop("`birthday_end` must be a date")
    }
    birthday_end <- format(birthday_end, "%Y-%m-%d")
  }
  
  json <- faker(
    "persons",
    gender = gender,
    birthday_start = birthday_start,
    birthday_end = birthday_end,
    quantity = quantity,
    locale = locale,
    seed = seed
  )  
  
  tibble::tibble(
    firstname = map_chr(json$data, "firstname"),
    lastname = map_chr(json$data, "lastname"),
    email = map_chr(json$data, "email"),
    gender = map_chr(json$data, "gender")
  )
}
faker_person("male", quantity = 5)
#> # A tibble: 5 × 4
#>   firstname lastname   email                          gender
#>   <chr>     <chr>      <chr>                          <chr> 
#> 1 Trey      Kassulke   haufderhar@konopelski.net      male  
#> 2 Weldon    Stiedemann elta.wolf@yahoo.com            male  
#> 3 Leonard   Runolfsson francisco.jacobson@hotmail.com male  
#> 4 Rashawn   Hegmann    fstroman@hotmail.com           male  
#> 5 Derick    Crooks     nikolaus.russel@gmail.com      male
```

Следующими шагами разработки пакета будет экспорт и документирование этой функции.

#### Управление секретными данными

Немного отвлечёмся от работы непосредственно с вызовами API и поговорим об управлении секретными данными. Секретные данные важны т.к. практически каждый API с которым вы будете работать, за исключением очень простых вроде Faker API, будут требовать от вас некой идентификации, зачастую идентификация пользователей в API реализована через ключи API или токены.

Описанный в этом разделе подход может быть для вас избыточным. Например, если у вас всего один токен, который вы используете в нескольких скриптах пакета, то достаточно будет поместить его в файл `.Renviron` и обращаться к нему с помощью `Sys.getenv()`. Но со временем количество хранимых API ключей и токенов будет расти, и вам потребуется разобраться с более эффективными способами хранения и распространения секретных данных, которые вам предоставляет пакет `httr2.`

##### Основы

httr2 предоставляет вам функции  `secret_encrypt()` и `secret_decrypt()` позволяющие шифровать секретные данные, и использовать их в своём коде не беспокоясь о том, что они попадут в третьи руки. Процесс шифрования состоит из трёх основных шагов:

1. С помощью функции `secret_make_key()` создаётся ключ шифрования, который используется для шифрования и дешифрования секретов с использованием симметричной криптографии:


```r
key <- secret_make_key()
key
#> [1] "-6cGNKmH2WTfH5pVUll-sg"
```

(Обратите внимание, что в `secret_make_key()` используется криптографически безопасный генератор случайных чисел, предоставляемый OpenSSL; на него не влияют настройки RNG R, и нет никакого способа сделать его воспроизводимым.)

2. Далее шифруете секретные данные с помощью `secret_encrypt()` и сохраняете полученный текст непосредственно в исходном коде вашего пакета:


```r
secret_scrambled <- secret_encrypt("secret I need to work with an API", key)
secret_scrambled
#> [1] "ohd9iBHJ66k5j8trIPVeENIPmINN2YWs4ceD1l6tz3B8GjotwFhI4f92lHDCSW_p6A"
```

3. При необходимости вы дешифруете ваши данные, используя `secret_decrypt()`:


```r
secret_decrypt(secret_scrambled, key)
#> [1] "secret I need to work with an API"
```

##### Пакетные ключи и секретные данные

Вы можете создать любое количество ключей шифрования, но я настоятельно рекомендую создавать один ключ для каждого пакета, который я буду называть ключом пакета. В этом разделе я покажу, как сохранить этот ключ, так чтобы к нему имели доступ только вы и написанные вами автоматические тесты.

В httr2 заложена идея, что ключ должен хранится в переменной окружения. Итак, первый шаг — сделать созданный вами ключ пакета доступным на вашем локальном компьютере, добавив строку с переменной на уровене пользователя в файл `.Renviron` (который вы можете открыть или при необходимости создать с помощью `usethis::edit_r_environ()`):

```
YOURPACKAGE_KEY=key_you_generated_with_secret_make_key
```

Теперь (после перезапуска R) вы сможете воспользоваться специальной возможностью `secret_encrypt()` и `secret_decrypt()`: аргументом `key` может быть имя переменной среды, а не сам ключ шифрования. На самом деле, это наиболее эффективное использование данного аргумента.


```r
secret_scrambled <- secret_encrypt("secret I need to work with an API", "YOURPACKAGE_KEY")
secret_scrambled
#> [1] "aoErRT9hj9M5N_zFZ4ehQIdKTKplbwaCovmYwrtpLkYt1HKa4aiKBWxriMjtpV2KBA"
secret_decrypt(secret_scrambled, "YOURPACKAGE_KEY")
#> [1] "secret I need to work with an API"
```

Вам также нужно будет сделать ключ доступным в GitHub Actions вашего репозитория (как check, так и pkgdown), чтобы к ключю имели доступ ваши автоматические тесты. Для этого требуется два шага:

1. Добавьте ключ в раздел [repository secrets](https://docs.github.com/en/actions/reference/encrypted-secrets).
2. Расшарьте ключ на рабочие процессы, которым он нужен, добавив строку в соответствующий рабочий процесс:

```
    env:
      YOURPACKAGE_KEY: ${{ secrets.YOURPACKAGE_KEY }}
```

Другие платформы непрерывной интеграции предлагают аналогичные способы сделать ключ доступным в качестве безопасной переменной среды.

##### Когда ключ пакета недоступен

Есть несколько важных случаев, когда ваш код не будет иметь доступа к ключу вашего пакета: в CRAN, на личных машинах внешних разработчиков и при прогонке автоматических тестов. Поэтому, если вы хотите поделиться своим пакетом на CRAN или облегчить другим пользователям возможность внести свой вклад в его развитие, вам нужно убедиться, что ваши примеры, виньетки и тесты работают без ошибок:

* В виньетках вы можете запустить `knitr::opts_chunk(eval = secret_has_key("YOURPACKAGE_KEY"))`, чтобы код внутри чанков выполнялся только в том случае, если ваш ключ доступен.
* В примерах вы можете окружить блоки кода, для которых требуется ключ, с помощью `if (httr2::secret_has_key("YOURPACKAGE_KEY")) {}` .
* Тесты не требуют от вас дополнительных действий, т.к. когда `secret_decrypt()` запускается в `testthat`, он автоматически запускает `skip()` для пропуска теста, если ключ недоступен.

#### NYTimes Books API

Далее мы рассмотрим NYTimes Books API. Данный API требует от вас простую авторизацию через ключи API, которыми необходимо подписывать каждый отправляемый запрос. Разрабатывая пакет для работы с API требующий указания API ключи в каждом запросе, вы столкнётесь с двумя проблемами:

Как организовать авто тесты не раскрывая свой ключ API;

Как упростить пользователям пакета передачу API ключа в каждый запрос, не дублируя его в каждую отдельную функцию.

Итак, на данном этапе вам уже понятно, как работает приведённый ниже код для получения моего ключа API NYTimes Book:


```r
my_key <- secret_decrypt("4Nx84VPa83dMt3X6bv0fNBlLbv3U4D1kHM76YisKEfpCarBm1UHJHARwJHCFXQSV", "HTTR2_KEY")
```

Я начну с решения первой проблемы, ко второму мы вернёмся в самом конце этого раздела, потому что с ним проще разобраться, когда у нас есть готовая функция.

###### Базовый запрос

Теперь давайте выполним тестовый запрос и посмотрим на ответ:


```r
resp <- request("https://api.nytimes.com/svc/books/v3") %>% 
  req_url_path_append("/reviews.json") %>% 
  req_url_query(`api-key` = my_key, isbn = 9780307476463) %>% 
  req_perform()
resp
```

Как и большинство современных API, NYTimes Books API возвращает результат в JSON формате:


```r
resp %>% 
  resp_body_json() %>% 
  str()
```

Прежде чем привести этот код в вид функции немного поэксперементируем с ошибочными запросами.

#### Обработка ошибок

Что произойдет, в случае ошибки? Например, если мы преднамеренно предоставим неверный ключ:


```r
resp <- request("https://api.nytimes.com/svc/books/v3") %>% 
  req_url_path_append("/reviews.json") %>% 
  req_url_query(`api-key` = "invalid", isbn = 9780307476463) %>% 
  req_perform()
```

Посмотреть, есть ли в ответе какая-либо дополнительная полезная информация, можно с помощью `last_response()`:


```r
resp <- last_response()
resp
resp %>% resp_body_json()
```

Полезную дополнительную информация об ошибке можно найти в `faultstring`:


```r
resp %>% resp_body_json() %>% .$fault %>% .$faultstring
```

Для того, что бы наш пакет выводил эту дополнительную информацию об ошибках полученных в ходе работы с API необходимо использовать функцию `req_error()` и её аргумент `body.` В body необходимо передать функцию, принимающую в качестве аргумента объект ответа от сервера, и возвращающую строку с дополнительной информацией о причине ошибке. Давайте попробуем доработать наш запрос:


```r
nytimes_error_body <- function(resp) {
  resp %>% resp_body_json() %>% .$fault %>% .$faultstring
}

resp <- request("https://api.nytimes.com/svc/books/v3") %>% 
  req_url_path_append("/reviews.json") %>% 
  req_url_query(`api-key` = "invalid", isbn = 9780307476463) %>% 
  req_error(body = nytimes_error_body) %>% 
  req_perform()
```

#### Ограничения скорости

Другим распространенным источником ошибок является ограничение скорости — этот лимит используется многими серверами, для избежание черезмерного потребления ресурсов одним пользователем. На странице часто задаваемых вопросов описаны ограничения скорости для API NYT:

> Существует два ограничения скорости: 4000 запросов в день и 10 запросов в минуту. Вы должны выдержать паузу в 6 секунд между запросами, чтобы избежать превышения предельного лимита количества отправленных запросов в минуту. Если вам нужен более высокий предел скорости, свяжитесь с нами по дресу code@nytimes.com.

Не редко API в ответе возвращают допонительную информацию, о том, какую паузу необходимо выждать для успешной отправки следующего запроса, если вы превысили какой то из описанных выще лимитов. Часто эта информация хранится в заголовке `Retry-After`.

Я намеренно нарушил лимит скорости, быстро сделав 11 запросов; к сожалению, хотя код статуса ответа был стандартным 429 (Too many requests), он не содержал ни в теле ответа, ни в заголовках никакой информации о том, какую паузу необходимо выдержать перед отправкой следующего запроса. Это означает, что мы не можем использовать `req_retry()`, которая ожидает информацию о времени таймаута в ответе сервера. Вместо этого мы будем использовать `req_throttle()`, которая позволяет ограничить количество отправляемых запросов, в данном случае мы будем уверены, что отправляем не более 10 запросов каждые 60 секунд:


```r
req <- request("https://api.nytimes.com/svc/books/v3") %>% 
  req_url_path_append("/reviews.json") %>% 
  req_url_query(`api-key` = "invalid", isbn = 9780307476463) %>% 
  req_throttle(10 / 60)
```

По умолчанию `req_throttle()` разделяет ограничение на все запросы к указанному хосту (т.е.  api.nytimes.com). Поскольку документы предполагают, что ограничение скорости применяется к отдельным конечным точкам API, вы можете использовать аргумент realm, чтобы более точно определить конечную точку, на которую действует указанное вами ограничение скорости отправки запроса:


```r
req <- request("https://api.nytimes.com/svc/books/v3") %>% 
  req_url_path_append("/reviews.json") %>% 
  req_url_query(`api-key` = "invalid", isbn = 9780307476463) %>% 
  req_throttle(10 / 60, realm = "https://api.nytimes.com/svc/books")
```

#### Оборачиваем функцию

Объединение всех вышеперечисленных примеров дает примерно такую ​​функцию:


```r
nytimes_books <- function(api_key, path, ...) {
  request("https://api.nytimes.com/svc/books/v3") %>% 
    req_url_path_append("/reviews.json") %>% 
    req_url_query(..., `api-key` = api_key) %>% 
    req_error(body = nytimes_error_body) %>% 
    req_throttle(10 / 60, realm = "https://api.nytimes.com/svc/books") %>% 
    req_perform() %>% 
    resp_body_json()
}

drunk <- nytimes_books(my_key, "/reviews.json", isbn = "0316453382")
drunk$results[[1]]$summary
```

Чтобы доработать этот код, до уровня пакета, надо:

1. Добавить явные аргументы и убедиться, что они имеют правильный тип.
2. Задокументировать и экспортировать функцию.
3. Преобразовать полученный список в более удобную для пользователя структуру данных (возможно, в фрейм данных с одной строкой на обзор).
4. Также лучше предоставить пользователю удобный способ использовать свой собственный ключ API.

#### Пользовательский ключ

Хорошим местом для хранения API ключа являются переменные среды, т.к. их легко установить, не вводя ничего в консоли (которая может быть случайно передана через ваш файл .Rhistory), и их легко установить в автоматизированных процессах. Затем вы должны написать функцию для получения ключа API, возвращающую сообщение, если он не найден:


```r
get_api_key <- function() {
  key <- Sys.getenv("NYTIMES_KEY")
  if (identical(key, "")) {
    stop("No API key found, please supply with `api_key` argument or with NYTIMES_KEY env var")
  }
  key
}
```

Теперь можно доработать `nytimes_books()`, и использовать `get_api_key()` как значение по умолчанию для аргумента `api_key.` Поскольку аргумент теперь является необязательным, мы можем переместить его в конец списка аргументов, так как он понадобится только в исключительных случаях.


```r
nytimes_books <- function(path, ..., api_key = get_api_key()) {
  ...
}
```

Вы можете сделать этот подход более удобным для пользователя, предоставив вспомогательную функцию, которая устанавливает переменную среды:


```r
set_api_key <- function(key = NULL) {
  if (is.null(key)) {
    key <- askpass::askpass("Please enter your API key")
  }
  Sys.setenv("NYTIMES_KEY" = key)
}
```

Использование `askpass()` (или её аналогов) является хорошей практикой, поскольку это даёт возможность скрыть вводимый пользователем ключь, в отличае от использования для этого консоли.

Рекомендуется доработать `get_api_key()` добавив автоматическое использование зашифрованного ключа, чтобы упростить написание авто тестов:


```r
get_api_key <- function() {
  key <- Sys.getenv("NYTIMES_KEY")
  if (!identical(key, "")) {
    return(key)
  }
  
  if (is_testing()) {
    return(testing_key())
  } else {
    stop("No API key found, please supply with `api_key` argument or with NYTIMES_KEY env var") 
  }
}

is_testing <- function() {
  identical(Sys.getenv("TESTTHAT"), "true")
}

testing_key <- function() {
  secret_decrypt("4Nx84VPa83dMt3X6bv0fNBlLbv3U4D1kHM76YisKEfpCarBm1UHJHARwJHCFXQSV", "HTTR2_KEY")
}
```

### OAuth протокол

Протокол OAuth был придуман с целью безопасности, для того, что бы вам в HTTP запросе не надо было предоставлять свои учётные данные, т.е. логин и пароль. Но, сам протокол является более сложным, чем вариант, когда ваш токен находится в настройках аккаунта. 

Весь процесс прохождения авторизации по протоколу OAuth отличается в разных API, но ниже я напишу общий процесс, который применим в большинстве случаев:

1. Создаёте приложение, для получения его ID и Secret
2. Далее из R запускаете браузер для генерации токена, либо кода для обмена на токен
3. Если на предыдущем шаге вы получили код, следующим шагом надо его обменять на токен
4. Далее кешируете полученный токен, можно в локальный файл
5. Многие API выдают токены с ограниченным сроком работы, такие токены по истечению этого срока необходимо обновлять, зачастую отдельным запросом

Далее мы в качестве примера возьмём Facebook API. В [справке по авторизации](https://developers.facebook.com/docs/facebook-login/manually-build-a-login-flow?locale=ru_RU) говорится о том, что для авторизации вам необходимо перейти по следующиему URL - https://www.facebook.com/v18.0/dialog/oauth, и указать некоторые дополнительные параметры:

* `client_id`. ID приложения, который можно найти в его панели.
* `redirect_uri`. URL, на который будет перенаправлен входящий пользователь. Этот URL получает ответ из диалога входа. Если вы используете веб-просмотр в приложении для ПК, для этого URL должно быть задано значение https://www.facebook.com/connect/login_success.html. Проверить, установлен ли этот URL для вашего приложения, можно в Панели приложений. В меню навигации в левой части Панели приложений выберите раздел Продукты, нажмите Вход через Facebook, а затем выберите Настройки. Проверьте Действительные URI для перенаправления OAuth в разделе Клиентские настройки OAuth.
* `state`. Строковое значение, создаваемое приложением для сохранения статуса между запросом и обратным вызовом. Этот параметр предназначен для защиты от подделки межсайтовых запросов и передается обратно без изменений в URI перенаправления.
* `response_type`. Указывает, куда будут добавлены данные ответа при перенаправлении обратно в приложение: в параметры или во фрагменты URL. Сведения о том, какой тип приложения выбрать, см. в этом разделе. Имеются следующие варианты:
* `code`. Данные ответа добавляются в параметры URL и содержат параметр code (зашифрованную строку, уникальную для каждого запроса входа). Если этот параметр не указан, по умолчанию функция работает именно так. Этот вариант подходит лучше всего, если маркер обрабатывается сервером.
* `token`. Данные ответа добавляются в виде фрагмента URL и содержат маркер доступа. Это значение response_type необходимо использовать в приложениях для ПК. Этот вариант подходит лучше всего, если маркер обрабатывается клиентом.
* `code%20token`. Данные ответа добавляются в виде фрагмента URL и содержат как маркер доступа, так и параметр code.
* `granted_scopes`. Возвращает разделенный запятыми список всех разрешений, предоставленных приложению пользователем на этапе входа. Может комбинироваться с другими значениями response_type. При использовании с параметром token данные ответа добавляются в виде фрагмента URL, в противном случае — в виде параметра URL.
* `scope`. Разделенный запятыми или пробелами список разрешений, которые нужно запросить у пользователя приложения.

Далеко не все из этих параметров являются обязательными, на самом деле обязательным являются только `client_id` и `redirect_uri`, остальные параметры опциональны. Так же в справке есть пример URL с параметрами, на который вы должны перейти для прохождения авторизации.

```
https://www.facebook.com/v18.0/dialog/oauth?
  client_id={app-id}
  &redirect_uri={"https://www.domain.com/login"}
  &state={"{st=state123abc,ds=123456789}"}
```

Для прохождения процесса авторизации нам потребуется `client_id`, а для того, что бы заменить краткосрочный токен на долгосрочный понадобится `secret`. Это параметры вашего OAuth клиента, который также в контексте протокола OAuth могут назвать приложением. Т.е. перед тем, как начать процесс авторизации вам необходимо перейти в раздел ["Мои Приложения"](https://developers.facebook.com/apps), создать приложения, и далее в разделе его настроект скопироваьт его id и secret. 

После того, как вы это сделаете, в R с помощью следующего кода можно инициировать процесс авторизации через браузер, получить краткосрочный токен, передать его в переменную, и обменять на долгосрочнй.


```r
library(urltools)
library(magrittr)
library(tidyr)
library(httr2)

app_id <- "ID вашего приложения"
secret <- "Секрет вашего приложения"

# разрешения
browseURL('https://developers.facebook.com/docs/permissions/reference')
scopes <- c("ads_read", "pages_manage_ads", "ads_management", "public_profile")
scopes <- paste(scopes, collapse = ",")

# авторизация
"https://www.facebook.com/v18.0/dialog/oauth" %>% 
  param_set(key = "client_id",     value = app_id) %>% 
  param_set(key = "display",       value = "popup") %>% 
  param_set(key = "redirect_uri",  value = "https://selesnow.github.io/rfacebookstat/getToken/get_token.html") %>% 
  param_set(key = "response_type", value = "token") %>% 
  param_set(key = "scope",         value = scopes) %>% 
  browseURL()

shorttime_token <- "ПОЛУЧЕНЫЙ ВАМИ КРАТКОСРОЧНЫЙ ТОКЕН"

# обмен на долгосрочный токен
browseURL('https://developers.facebook.com/docs/facebook-login/guides/access-tokens/get-long-lived')
lt_token <- request("https://graph.facebook.com/oauth/access_token") %>% 
  req_url_query(
    grant_type = "fb_exchange_token",
    client_id = app_id,
    client_secret = "40ed3b067df92249372c7501d512c198",
    fb_exchange_token = shorttime_token
  ) %>% 
  req_perform() %>% 
  resp_body_json()
```

`scopes` - это набор разрешений, т.е. с помощью этого параметра вы можете дать определённые права вашему токену, примерно также, как вы расшариваете доступ к какому то сервису на других пользователей, указав их роль, тем самым регулируя их возможности на просмотр или редактивание данных. Посмотреть список разрешений в Facebook API можно [тут](https://developers.facebook.com/docs/permissions/reference).

Т.к. мы проверили наш код авторизации, теперь мы можем упаковать его в готовую функцию:


```r
fb_auth <- function(app_id, client_secret, scopes) {
  
  scopes <- paste(scopes, collapse = ",")
  
  "https://www.facebook.com/v18.0/dialog/oauth" %>% 
    param_set(key = "client_id",     value = app_id) %>% 
    param_set(key = "display",       value = "popup") %>% 
    param_set(key = "redirect_uri",  value = "https://selesnow.github.io/rfacebookstat/getToken/get_token.html") %>% 
    param_set(key = "response_type", value = "token") %>% 
    param_set(key = "scope",         value = scopes) %>% 
    browseURL()
  
  shorttime_token <- askpass::askpass("Please enter your API key")
  
  lt_token <- request("https://graph.facebook.com/oauth/access_token") %>% 
    req_url_query(
      grant_type = "fb_exchange_token",
      client_id = app_id,
      client_secret = client_secret,
      fb_exchange_token = shorttime_token
    ) %>% 
    req_perform() %>% 
    resp_body_json()
  
  Sys.setenv("FB_APIKEY"=lt_token$access_token)
  lt_token
  
}

# тесируем
fb_token <- fb_auth(
  app_id = "ID вашего приложения", 
  client_secret = "Секрет вашего приложения", 
  scopes = c("ads_read", "pages_manage_ads", "ads_management", "public_profile")
)
```

Теперь у нас есть токен, которым мы должны подписывать все запросы к API, в справке Facebook API говорится о том, что токен необходимо передавать с каждым запросом через GET параметр `access_token`.

Далее мы можем попробовать запросить какие нибудь данные, например данные узла [/me](https://developers.facebook.com/docs/graph-api/overview#me), в справке указан следующий пример:

```
curl -i -X GET \
  "https://graph.facebook.com/me?access_token=ACCESS-TOKEN"
```

В R код это можно перевести следующим образом:


```r
resp <- request('https://graph.facebook.com/') %>% 
  req_url_path('me') %>% 
  req_url_query(access_token = Sys.getenv("FB_APIKEY")) %>%
  req_perform() %>% 
  resp_body_json()
```

Так же указав дополнительный параметр `metadata` можно запросить метаданные узла, в справке приведён следующий пример:

```
curl -i -X GET \
  "https://graph.facebook.com/USER-ID?
    metadata=1&access_token=ACCESS-TOKEN"
```

В R это выглядит так:


```r
metadata <- request('https://graph.facebook.com/') %>% 
  req_url_path('me') %>% 
  req_url_query(access_token = Sys.getenv("FB_APIKEY"), metadata = 1) %>%
  req_perform() %>% 
  resp_body_json()
```

В результате мы получим список метаданных в виде списка:

```
{
  "name": "Jane Smith",
  "metadata": {
    "fields": [
      {
        "name": "id",
        "description": "The app user's App-Scoped User ID. This ID is unique to the app and cannot be used by other apps.",
        "type": "numeric string"
      },
      {
        "name": "age_range",
        "description": "The age segment for this person expressed as a minimum and maximum age. For example, more than 18, less than 21.",
        "type": "agerange"
      },
      {
        "name": "birthday",
        "description": "The person's birthday.  This is a fixed format string, like `MM/DD/YYYY`.  However, people can control who can see the year they were born separately from the month and day so this string can be only the year (YYYY) or the month + day (MM/DD)",
        "type": "string"
      },
...
```

Имеет смысл преобразовать полученый список в таблицу с тремя полями: name, description, type:


```r
metadata_res <- tibble(metadata = metadata$metadata$fields) %>% 
  unnest_wider(metadata)
```

Теперь мы можем обернуть эндпоинт для получения метаданных узла `/me` в функцию:


```r
# заворачиваем в функцию
fb_get_metadata <- function() {
  
  metadata <- request('https://graph.facebook.com/') %>% 
    req_url_path('me') %>% 
    req_url_query(access_token = Sys.getenv("FB_APIKEY"), metadata = 1) %>%
    req_perform() %>% 
    resp_body_json()
  
  # разворачиваем ответ
  metadata_res <- tibble(metadata = metadata$metadata$fields) %>% 
    unnest_wider(metadata)
  
  metadata_res
  
}

meta <- fb_get_metadata()
```

## Тест
<iframe id="otp_wgt_lehi63pzmq24e" src="https://onlinetestpad.com/lehi63pzmq24e" frameborder="0" style="width:100%;" onload="var f = document.getElementById('otp_wgt_lehi63pzmq24e'); var h = 0; var listener = function (event) { if (event.origin.indexOf('onlinetestpad') == -1) { return; }; h = parseInt(event.data); if (!isNaN(h)) f.style.height = h + 'px'; }; function addEvent(elem, evnt, func) { if (elem.addEventListener) { elem.addEventListener(evnt, func, false); } else if (elem.attachEvent) { elem.attachEvent('on' + evnt, func); } else { elem['on' + evnt] = func; } }; addEvent(window, 'message', listener);" scrolling="no">
</iframe>

------


```{=html}
<style>.social-btn-left{padding:6px 5px 5px 30px;border-bottom-right-radius:20px;border-top-right-radius:20px;margin-bottom:5px;left:-110px}.r2social-social-inline{display:flex}.social-btn-right{padding:6px 30px 5px 5px;border-bottom-left-radius:20px;border-top-left-radius:20px;margin-bottom:5px;right:110px}.social-btn-inline{padding:0;border-radius:50%;margin:5px}.r2social-social-inline p{display:none!important}.social-btn-left,.social-btn-right{display:flex;width:150px;align-items:center;justify-content:space-between;position:relative;border:1px;transition:left 1s;color:#fff}.social-btn-left:hover{left:-20px;transition:left 1s}.social-btn-right:hover{right:190px;transition:right 1s}.r2social-icons-inline:hover{background-color:#ff0}.r2social-social-left{position:fixed;top:50px;left:15px}.r2social-social-inline a,.r2social-social-left a,.r2social-social-right a{text-decoration:none!important}.r2social-social-right{position:fixed;top:50px;right:-215px;z-index:999!important}.color-telegram{background-color:#0084c6}.color-instagram{background-color:#f62782}.color-whatsapp{background-color:#24cc63}.google-font{font-family:Lato,sans-serif;font-size:1.25rem}.social-btn-left img,.social-btn-right img{width:40px}.social-btn-left p,.social-btn-right p{color:#fff;margin-top:0;margin-bottom:0}.r2social-icons-left{order:2}.r2social-icons-inline,.r2social-icons-left,.r2social-icons-right{display:inline-block;width:38px;height:38px;background:url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAASwAAAEsCAYAAAB5fY51AACFMElEQVR4Xu2dB7gV1dX3B0uQS7GBCEaKqBTFRhErgsYGGImiImpiiYqJmryJJZ8lb8xr1KiJib4SjS0SW9RILESNNCtSXgMoRUCKCiolSLkXROX7/8bZN3PmzpyZOXfmnIvZ8zzzcDmz2/rvtddee+2113Yc+1gELAIWAYuARcAiYBGwCFgELAIWAYuARcAiYBGwCFgELAIWAYuARcAiYBGwCFgELAIWAYuARcAiYBGwCFgELAIWAYuARcAiYBGwCFgELAIWAYuARcAiYBGwCFgELAIWAYuARcAiYBGwCFgELAIWAYuARcAiYBGwCFgELAIWAYuARcAiYBGwCFgELAIWAYuARcAiYBGwCFgELAIWAYvAfwACNdVrG/0HkBlJYp70U3ba8uPyxJVn8sel+0/uc0u7RcAiYBHIDIHNagZ9/u9/P+zpp58+aeasWXuDQMuWLZcddtihEwafeOLj7dp3WBGFyssvT+j1zDPPnDjznZl7r6uuburlG3/MMUc/16VLtw+Kobl40cKWL40Ze/T4ceOOXPz++x2UtlG3rl1n9Ovf7x9DhpzybDCv2tj3nnvvvei8c88dcexxx40vVrbSHq60Pzj++OP/ds455zxcLO3UKZO73XDjTdd6aR+pDwfQxocffvjs875/3v8efnjfyWFlLfvk42a/vvnmq1q02HbVpZdcfHOLbbf70qRb/emqLX/3+9svm/vuu131G79v8pXRqPXOrZdcftll17faqXW1+d3U6aWtTb/HnnvO7tv38LH77bvvW6rji7C20H+/V32nnnrKSGH+TAymLv5JMC1WzuzZM9v96vobftGjZ49Jl176oxFhaYVDoxdefHHguLHjvmV4st2uuy48ol+/l446sv+L4snlYfngqadGjTr5lVde7bt8+fKdSANPnXDCCX8Vz7wcUdcWwvynS5cs2eUXv/jvq4TtWn869VeV+utq9denwf4y6XzjoLs3Dj7Zd999/2/IkJMeiRsH9eG3/8i8v/zlL3+6zTbb1MDw3/zmNz/Q+775f69evd7QgGbw1HmU7/IddthhFfn0778C+SaqE3tEASqm3XXA8cczQBhgtfXyt+recNHw4beJUZr481922U+v57u+/a5YRylf0yOOOGIMabt06TyTAVIs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) -21px -246px no-repeat #fff;border-radius:50%}.r2s-ico-vk{background-position:-187px -246px}.r2s-ico-blogger{background-position:-244px -247px}.r2s-ico-link-out{background-position:-245px -152px}.r2s-ico-visit-us{background-position:-243px -197px}.r2s-ico-weibo{background-position:-131px -246px}.r2s-ico-facebook{background-position:-19px -50px}.r2s-ico-facebook-messenger{background-position:-75px -50px}.r2s-ico-instagram{background-position:-131px -50px}.r2s-ico-whatsapp{background-position:-187px -50px}.r2s-ico-twitter{background-position:-243px -50px}.r2s-ico-pinterest{background-position:-19px -99px}.r2s-ico-tiktok{background-position:-76px -99px}.r2s-ico-youtube{background-position:-131px -99px}.r2s-ico-linkedin{background-position:-187px -99px}.r2s-ico-snapchat{background-position:-131px -150px}.r2s-ico-tumblr{background-position:-187px -148px}.r2s-ico-reddit{background-position:-187px -197px}.r2s-ico-telegram{background-position:-75px -246px}</style><script>/*! r2social | (c) Written by Obi Obianom, www.obianom.com | all rights reserved */$(function(){$(".r2social-link-container a").click(function(t){t.preventDefault(),console.log($(this).attr("href")),window.open($(this).attr("href"),"_r2socialxlink","height=500,width=400,resizable=yes,scrollbars=yes,toolbar=yes,menubar=no,location=no,directories=no, status=yes")})});</script>
```

```{=html}
<div class="r2social-link-container r2social-social-inline">
<a href="https://www.facebook.com/sharer/sharer.php?u=https://selesnow.github.io/r_package_course/" target="_r2socialxlink">
<div class="social-btn-inline" style="background-color:#1877f2">
<div class="r2social-icons-inline r2s-ico-facebook"></div>
<p class="order-1 google-font margin-telgram" style="color:black">facebook</p>
</div>
</a>
<a href="https://www.linkedin.com/shareArticle?mini=true&amp;url=https://selesnow.github.io/r_package_course/" target="_r2socialxlink">
<div class="social-btn-inline" style="background-color:#0A66C2">
<div class="r2social-icons-inline r2s-ico-linkedin"></div>
<p class="order-1 google-font margin-telgram" style="color:black">linkedin</p>
</div>
</a>
<a href="https://twitter.com/intent/tweet?url=https://selesnow.github.io/r_package_course/&amp;text=" target="_r2socialxlink">
<div class="social-btn-inline" style="background-color:#1DA1F2">
<div class="r2social-icons-inline r2s-ico-twitter"></div>
<p class="order-1 google-font margin-telgram" style="color:black">twitter</p>
</div>
</a>
<a href="https://web.whatsapp.com/send?text= https://selesnow.github.io/r_package_course/" target="_r2socialxlink">
<div class="social-btn-inline" style="background-color:#24cc63">
<div class="r2social-icons-inline r2s-ico-whatsapp"></div>
<p class="order-1 google-font margin-telgram" style="color:black">whatsapp</p>
</div>
</a>
<a href="https://reddit.com/submit?url=https://selesnow.github.io/r_package_course/&amp;title=" target="_r2socialxlink">
<div class="social-btn-inline" style="background-color:#FF5700">
<div class="r2social-icons-inline r2s-ico-reddit"></div>
<p class="order-1 google-font margin-telgram" style="color:black">reddit</p>
</div>
</a>
<a href="https://service.weibo.com/share/share.php?url=https://selesnow.github.io/r_package_course/&amp;title=" target="_r2socialxlink">
<div class="social-btn-inline" style="background-color:#ce1126">
<div class="r2social-icons-inline r2s-ico-weibo"></div>
<p class="order-1 google-font margin-telgram" style="color:black">weibo</p>
</div>
</a>
<a href="https://www.xing.com/app/user?op=share&amp;url=https://selesnow.github.io/r_package_course/" target="_r2socialxlink">
<div class="social-btn-inline" style="background-color:#C32AA3">
<div class="r2social-icons-inline r2s-ico-instagram"></div>
<p class="order-1 google-font margin-telgram" style="color:black">instagram</p>
</div>
</a>
<a href="https://telegram.me/share/url?url=https://selesnow.github.io/r_package_course/&amp;text=" target="_r2socialxlink">
<div class="social-btn-inline" style="background-color:#0088cc">
<div class="r2social-icons-inline r2s-ico-telegram"></div>
<p class="order-1 google-font margin-telgram" style="color:black">telegram</p>
</div>
</a>
</div>
```

```{=html}
<div style="display:inline-block">
<div class="r2social-link-container r2social-social-left">
<a href="https://t.me/R4marketing" target="_r2socialxlink">
<div class="social-btn-left" style="background-color:#0088cc">
<div class="r2social-icons-left r2s-ico-telegram"></div>
<p class="order-1 google-font margin-telgram" style="color:white">telegram</p>
</div>
</a>
</div>
</div>
```

```{=html}
<div style="display:inline-block">
<div class="r2social-link-container r2social-social-right">
<a href="https://www.youtube.com/R4marketing/?sub_confirmation=1" target="_r2socialxlink">
<div class="social-btn-right" style="background-color:#ff0000">
<div class="r2social-icons-right r2s-ico-youtube"></div>
<p class="order-1 google-font margin-telgram" style="color:white">youtube</p>
</div>
</a>
</div>
</div>
```
