# conspectus
Разные материалы


## Паттерны проектирования систем

### Заметки:
Причины использовать контейнеры при разработке распределенных систем:
 * установить ограничение на определенный ресурс (например, приложению нужно два процессорных ядра и 8 Гбайт оперативной памяти).
 * способствует разделению зон ответственности между командами. 
 
В отличие от реплицированных сервисов каждая копия шардированного сервиса (шард) может обслужить только часть запросов. Узел балансировки нагрузки (корневой узел) отвечает за изучение каждого запроса и перенаправление его соответствующему узлу (или узлам) для обработки.
![image](https://user-images.githubusercontent.com/38044240/86128083-517da300-bafa-11ea-818c-0fef261af112.png)


## Одноузловые паттерны
Паттерны зависящие от совместно используемых локальных ресурсов: дискового пространства, сетевых интерфейсов, а также от межпроцессного взаимодействия.
### Sidecar
Sidecar — это одноузловой паттерн, состоящий из двух контейнеров. Первый из них — контейнер приложения. Он содержит основную логику программы. Вдобавок к контейнеру приложения предусмотрен еще «прицепной» (sidecar) контейнер. Роль прицепа — дополнить и улучшить контейнер приложения, часто таким образом, чтобы приложение не знало о его существовании. 

Примеры использования:
 * Поставить перед основным контейнером сервер с NGinx для обеспечения httpS трафика (когда невозможно применение шифрования в старом контейнере).
 * Поставить перед основным контейнером, чтобы прицеп считывал настройки и перезагружал контейнер когда это требуется.
 * Добавляет любой несложный функционал.

### Ambassador
Контейнер-посол выступает посредником во взаимодействии контейнера приложения с внешним миром. Как и в случае с остальными одноузловыми паттернами проектирования, два контейнера составляют симбиотический союз и исполняются совместно на одном компьютере. ~~Грубо говоря, прокси-сервер~~

Один из вариантов — включить всю логику шардирования в сам шардированный сервис. При таком подходе шардированный сервис должен также иметь балансировщик нагрузки с независимой обработкой транзакций, адресующий трафик нужному шарду. Этот балансировщик нагрузки будет, по сути, распределенной реализацией паттерна Ambassador в виде сервиса.
Другой вариант — интегрировать одноузловую реализацию паттерна Ambassador на стороне клиента, чтобы она перенаправляла трафик нужному шарду.

Примеры использования:
 * Проксирование запросов к шардированным сервисам.
 * Сервис посредни для обследования окружения. Обязанность контейнера-посла сервиса-посредника заключается в обследовании окружения и опосредовании подключения к конкретному экземпляру целевого сервиса.
 * Разделение запросов для таких целей как: проверка тестового API новой версии на боевой нагрузке, сбор статистики, эксперементы с нагрузкой и т.д.
 
 ### Adapter
 В его рамках контейнер-адаптер модифицирует программный интерфейс контейнера приложения таким образом, чтобы он соответствовал некоему заранее определенному интерфейсу, реализация которого ожидается от всех контейнеров приложений. ~~Грубо говоря, переходник.~~
 
 Примеры использования: 
  * предоставления интерфейса для сервисов логирования, мониторинга и т.д.
---  
## Многоузловые паттерны
### Реплицированный сервис с распределением нагрузки
Простейший распределенный паттерн, известныймногим. В рамках такого сервиса все серверы идентичны друг другу и поддерживают входящий трафик. Паттерн состоит из масштабируемого набора серверов, находящихся за балансировщиком нагрузки. Балансировщик обычно распределяет нагрузку либо по карусельному (round-robin) принципу, либо с применением некоторой разновидности закрепления сессий.
Требуют продуманной системы детектирования готовности рабочего состояния для экземпляров сервисов.

Примеры:
 * Балансировщик нагрузки перед 2 и более экземплярами сервисов, для достижения 99.9% и отказоусточивости.
 * Кеширующая прослойка, для повышения скорости работы.
 * Различные надстройки перед подключением пользователя к сервисам (DoS-атаки, SSL-мосты, авторизация и тд.)
 
   
 
