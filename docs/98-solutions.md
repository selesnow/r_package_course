# Решение заданий {-}

## Урок 7: Тестирование {-}

После того как клонируете пакет, откройте его и запустите команду `usethis::use_testthat(3)`.

Далее запустите команду `usethis::use_test('str_plus')`, и в открывшийся файл `test-str_plus.R` добавьте следующие тесты:

Файл с тестами к функции `R/str_plus.R`:


```r
test_that("check concat length", {
  expect_length("my" %+% "little" %+% "string", 1)
})

test_that("check concat class", {
  expect_type("my" %+% "little" %+% "string", 'character')
})

test_that("check concat error", {
  expect_error("my" %+% "little" / 7, regexp = 'non-numeric argument to binary operator')
})

```

Далее запустите команду `usethis::use_test('str_split')`, и в открывшийся файл `test-str_split.R` добавьте следующие тесты:

Файл с тестами к фунции `R/str_split.R`


```r
test_that("check split value", {
  expect_equal('The-little-text' %/% "-", c('The', 'little', 'text'))
})

test_that("check split length", {
  expect_length('The-little-text' %/% "-", 3)
})

```

Запустите команду `devtools::test()` для выполнения тестов.
