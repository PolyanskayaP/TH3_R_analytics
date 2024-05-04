# lab5
Полянская Полина Алексеевна

Анализ данных сетевого трафика с использованием аналитической СУБД
Clickhouse

## Цель

1.  Изучить возможности СУБД Clickhouse для обработки и анализ больших
    данных
2.  Получить навыки применения Clickhouse совместно с языком
    программирования R
3.  Получить навыки анализа метаинфомации о сетевом трафике
4.  Получить навыки применения облачных технологий хранения, подготовки
    и анализа данных: Managed Service for ClickHouse, Rstudio Server.

## Исходные данные

1.  Ноутбук с ОС Windows 10
2.  RStudio
3.  Yandex Cloud
4.  RStudio Server
5.  СУБД ClickHouse

## Задание

Используя язык программирования R, библиотеку ClickhouseHTTP и облачную
IDE Rstudio Server, развернутую в Yandex Cloud, выполнить задания и
составить отчет.

## Ход работы

### Шаг 0. Настройка окружения.

Смена прав на доступ к файлу ключа.

    chmod =600 ~/key_th_pr3/rstudio.key

Подключение через SSH.

    ssh -i ~/key_th_pr3/rstudio.key -L 8787:127.0.0.1:8787 user23@62.84.123.211

Смена пароля.

    passwd

Также нужно подключиться к своему гит-репозиторию. Ссылка для генерации
токена аутентификации:

    https://github.com/settings/tokens

Полезные команды:

    git add lab3
    git commit -m "Цыпа"
    git push -u origin main

    git pull --rebase origin main

И установить пакет ClickHouseHTTP через кнопки Packages и Install в
RStudio. Или так:

    install.packages("ClickHouseHTTP")

### Шаг 1. Подключение к БД и импорт данных

``` r
Host <- Sys.getenv("HOST")
Port <- Sys.getenv("PORT")
User <- Sys.getenv("USER")
Pass <- Sys.getenv("PASS")
```

``` r
library(DBI)
```

``` r
library(dplyr)
```


    Attaching package: 'dplyr'

    The following objects are masked from 'package:stats':

        filter, lag

    The following objects are masked from 'package:base':

        intersect, setdiff, setequal, union

``` r
library(tidyverse)
```

    ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ✔ forcats   1.0.0     ✔ readr     2.1.5
    ✔ ggplot2   3.4.4     ✔ stringr   1.5.1
    ✔ lubridate 1.9.3     ✔ tibble    3.2.1
    ✔ purrr     1.0.2     ✔ tidyr     1.3.1
    ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ✖ dplyr::filter() masks stats::filter()
    ✖ dplyr::lag()    masks stats::lag()
    ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
library(lubridate)
```

``` r
library(ClickHouseHTTP)

## HTTP connection
con <- dbConnect(
   ClickHouseHTTP::ClickHouseHTTP(), 
   host=Host,
                      port=8443,
                      user=User,
                      password = Pass,
                      db = "TMdata",
   https=TRUE, ssl_verifypeer=FALSE)
```

``` r
dataFromDB <- dbReadTable(con, "data")
df <- dbGetQuery(con, "SELECT * FROM data")
```

``` r
head(df,5)
```

         timestamp           src          dst port bytes
    1 1.578326e+12   13.43.52.51 18.70.112.62   40 57354
    2 1.578326e+12 16.79.101.100  12.48.65.39   92 11895
    3 1.578326e+12 18.43.118.103  14.51.30.86   27   898
    4 1.578326e+12 15.71.108.118 14.50.119.33   57  7496
    5 1.578326e+12  14.33.30.103  15.24.31.23  115 20979

### Шаг 2. Обработка данных

#### 1. Найдите утечку данных из Вашей сети

Важнейшие документы с результатами нашей исследовательской деятельности
в области создания вакцин скачиваются в виде больших заархивированных
дампов. Один из хостов в нашей сети используется для пересылки этой
информации – он пересылает гораздо больше информации на внешние ресурсы
в Интернете, чем остальные компьютеры нашей сети. Определите его
IP-адрес.

``` r
leak <- df  %>% select(src, dst, bytes) %>% filter(!str_detect(dst, '1[2-4].*')) %>% group_by(src) %>% summarise(bytes_amount = sum(bytes)) %>% arrange(desc(bytes_amount)) %>% collect()

leak %>% head(1)
```

    # A tibble: 1 × 2
      src          bytes_amount
      <chr>               <dbl>
    1 13.37.84.125   5765792351

#### 2. Найдите утечку данных 2

Другой атакующий установил автоматическую задачу в системном
планировщике cron для экспорта содержимого внутренней wiki системы. Эта
система генерирует большое количество трафика в нерабочие часы, больше
чем остальные хосты. Определите IP этой системы. Известно, что ее IP
адрес отличается от нарушителя из предыдущей задачи.

#### 3. Найдите утечку данных 3

Еще один нарушитель собирает содержимое электронной почты и отправляет в
Интернет используя порт, который обычно используется для другого типа
трафика. Атакующий пересылает большое количество информации используя
этот порт, которое нехарактерно для других хостов, использующих этот
номер порта. Определите IP этой системы. Известно, что ее IP адрес
отличается от нарушителей из предыдущих задач.

#### 4. Обнаружение канала управления

Зачастую в корпоротивных сетях находятся ранее зараженные системы,
компрометация которых осталась незамеченной. Такие системы генерируют
небольшое количество трафика для связи с панелью управления бот-сети, но
с одинаковыми параметрами – в данном случае с одинаковым номером порта.
Какой номер порта используется бот-панелью для управления ботами?

#### 5. Обнаружение P2P трафика

Иногда компрометация сети проявляется в нехарактерном трафике между
хостами в локальной сети, который свидетельствует о горизонтальном
перемещении (lateral movement). В нашей сети замечена система, которая
ретранслирует по локальной сети полученные от панели управления бот-сети
команды, создав таким образом внутреннюю пиринговую сеть. Какой
уникальный порт используется этой бот сетью для внутреннего общения
между собой?

#### 6. Чемпион малвари

Нашу сеть только что внесли в списки спам-ферм. Один из хостов сети
получает множество команд от панели C&C, ретранслируя их внутри сети. В
обычных условиях причин для такого активного взаимодействия внутри сети
у данного хоста нет. Определите IP такого хоста.

#### 7. Скрытая бот-сеть

В нашем трафике есть еще одна бот-сеть, которая использует очень большой
интервал подключения к панели управления. Хосты этой продвинутой
бот-сети не входят в уже обнаруженную нами бот-сеть. Какой порт
используется продвинутой бот-сетью для коммуникации?

#### 8. Внутренний сканнер

Одна из наших машин сканирует внутреннюю сеть. Что это за система?

## Оценка результатов

С использованием СУБД ClickHouse, RStudio Server и языка
программирования R были выполнены задания.

## Вывод

СУБД ClickHouse - хорошее средство для решения задач с большими данными.