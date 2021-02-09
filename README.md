# Домашняя работа Архитектура
### Представим, что у нас есть данные, которые мы очень часто читаем по сравнению с другими(например словарь стран). Как можно это оптимизировать?

`Данные можно оптимизировать для хранения в памяти, например, Memcached или настроить СУБД на работу с данной таблицей из памяти. 
Так же это может быть отдельный инстанс БД на чтение.
Если объем большой, можно разделить обращения к этим данным на несколько сервисов и распределять запросы на балансировщике, так же кешировать особо популярные запросы, что может снизить нагрузку.`


### Что можно сделать, если таблица вакансий стала слишком большой? Какие есть решения на уровне текущей базы данных? Можно ли ее чем то заменить?

`Можно настроить партицирование таблицы, с настройкой логики хранения данных, например, более старые вакансии или менее ранжированные выделять в отдельные партиции, возможно на других серверах или СХД.
Так же для ускорения доступа к данным настроить индексы, в соответствии с планами выполнения запросов (проанализировать запросы в профайлере и выполнить с просмотром плана выполнения, для принятия решения об оптимизации запроса и создании индексов)`


### Какие вы видите узкие места, возможно неправильно выбранные технологии в текущей схеме(можно рассмотреть как “нашу” схему, так и схему настоящего hh.ru)

`Я взял за основу эту схему`
![Архитектура HH](/arh.PNG)

`На мой взгляд, если есть запросы не нуждающиеся в проверке session_id(авторизации) можно их разделить на входе, т.к. авторизация, как правило, тяжелая(долгая) операция. 
Это возможно даже сделать на основе формальных признаков(cookies, session, header). Т.е. если запрос к API/XHH подразумевает авторизацию, проверять на наличие необходимых параметров, до обращения к API/XHH.`

`Везде, где можно, использовать бинарные протоколы, для обмена информации между внутренними сервисами. Это уменьшит трафик и увеличит скорость, т.к сериализация бинарных данных менее ресурсна(память, процессор) чем стоки (JSON). `

`Из схемы не понятно, как идет работа со статичными данными. Например, если выделить отдельную БД с данными на чтение, которые обновляются в заданный интервал или по тригеру, это позволяет работать без блокировок на чтение, во время записи в основную БД. Т.е например,мы имеем, БД "статистики" (посчитанные данные, справочники), "хранилище" (данные для хранения и анализа), БД "репликации" (быстрое хранилище для сбора данных от поставщиков и дальнейшего коммита в "Хранилище" и "Статистику")`

`Как пример хранения:   
БД Статистики: Вакансии, пользователи, Города и т.п хранится в статистике (последнее результирующее действие, например, дата обновления резюме/вакансии),   
БД Хранилища: События, действия и т.п, например, даты правок, отклики, даты опубликования, аутентификации...    
БД Репликации: Данные полученные от АПИ, промежуточные результаты выполнения сервисов и т.п.    `
