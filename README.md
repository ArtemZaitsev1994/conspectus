# conspectus
Разные материалы


## Паттерны проектирования систем

### Заметки:
Причины использовать контейнеры при разработке распределенных систем:
 * установить ограничение на определенный ресурс (например, приложению нужно два процессорных ядра и 8 Гбайт оперативной памяти).
 * способствует разделению зон ответственности между командами. 

#### Шардированные\Реплицированные
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
 
---
 
## Транзакции баз данных
Цель транзакций - изоляция доступа к данным несколькими приложениями в конкурентной среде и обеспечение корректоного восстановления после сбоев. 
### ACID
**Atomicity, атомарность** - никакая транзакция не будет зафиксирована в системе частично. Будут либо выполнены все её подоперации, либо не выполнено ни одной. Поскольку на практике невозможно одновременно и атомарно выполнить всю последовательность операций внутри транзакции, вводится понятие «отката» (rollback): если транзакцию не удаётся полностью завершить, результаты всех её до сих пор произведённых действий будут отменены и система вернётся во «внешне исходное» состояние — со стороны будет казаться, что транзакции и не было. (Естественно, счётчики, индексы и другие внутренние структуры могут измениться, но, если СУБД запрограммирована без ошибок, это не повлияет на внешнее её поведение.)

**Consistency, согласованность** - 

**Isolaction, изоляция** - во время выполнения транзакции параллельные транзакции не должны оказывать влияния на её результат. Изолированность — требование дорогое, поэтому в реальных БД существуют режимы, не полностью изолирующие транзакцию (уровни изолированности Repeatable Read и ниже).

### Уровень изолированности транзакций
Условное значение, определяющее, в какой мере в результате выполнения логически параллельных транзакций в СУБД допускается получение несогласованных данных.

При параллельном выполнении транзакций возможны следующие проблемы:
 * потерянное обновление (англ. lost update) — при одновременном изменении одного блока данных разными транзакциями теряются все изменения, кроме последнего;
 * «грязное» чтение (англ. dirty read) — чтение данных, добавленных или изменённых транзакцией, которая впоследствии не подтвердится (откатится);
 * неповторяющееся чтение (англ. non-repeatable read) — при повторном чтении в рамках одной транзакции ранее прочитанные данные оказываются изменёнными;
 * фантомное чтение (англ. phantom reads) — одна транзакция в ходе своего выполнения несколько раз выбирает множество строк по одним и тем же критериям. Другая транзакция в интервалах между этими выборками добавляет строки или изменяет столбцы некоторых строк, используемых в критериях выборки первой транзакции, и успешно заканчивается. В результате получится, что одни и те же выборки в первой транзакции дают разные множества строк.

## Уровни изоляции
### Read uncommitted (чтение незафиксированных данных)
Низший (первый) уровень изоляции. Он гарантирует только отсутствие потерянных обновлений[1]. Если несколько параллельных транзакций пытаются изменять одну и ту же строку таблицы, то в окончательном варианте строка будет иметь значение, определенное всем набором успешно выполненных транзакций. При этом возможно считывание не только логически несогласованных данных, но и данных, изменения которых ещё не зафиксированы.

### Read committed (чтение фиксированных данных)
Большинство промышленных СУБД, в частности, Microsoft SQL Server, PostgreSQL и Oracle, по умолчанию используют именно этот уровень. На этом уровне обеспечивается защита от чернового, «грязного» чтения, тем не менее, в процессе работы одной транзакции другая может быть успешно завершена и сделанные ею изменения зафиксированы. В итоге первая транзакция будет работать с другим набором данных.

### Repeatable read (повторяемость чтения)
Уровень, при котором читающая транзакция «не видит» изменения данных, которые были ею ранее прочитаны. При этом никакая другая транзакция не может изменять данные, читаемые текущей транзакцией, пока та не окончена.


### Serializable (упорядочиваемость)
Самый высокий уровень изолированности; транзакции полностью изолируются друг от друга, каждая выполняется так, как будто параллельных транзакций не существует. Только на этом уровне параллельные транзакции не подвержены эффекту «фантомного чтения».

---

# Построение API
## REST

### Принципы
1. **Client - Server.** Четкое разделение слоев клиента и сервера. Клиент никак не связан с бизнес-логикой или хранением данных.

2. **Stateless**. Сервер не должен хранить состояние. Все запросы обрабатываются полным циклом, в запросе хранится вся информация для обработки.

3. **Cache.** Ответы должны помечаться можно их кэшировать или нет. 

4. **Uniform Interface.** Единый интерфейс между клиентом и сервером для унификации обработки запросов и ответов.
  * Идентификатором является URI.
  * Манипуляции над ресурсами через представления.
  * Само-документированные сообщения.
  * HATEOAS (hypermedia as the engine of application state). Статус ресурса передается через содержимое body, параметры строки запроса, заголовки запросов и запрашиваемый URI (имя ресурса).

5. **Layered System.** Каждый слой видит только последующий слой системы.

6. **Code on demand.** Возможность загрузки кода для выполнения на сторону клиента.

## GraphQL
Позволяет составлять запросы на клиенте, для получения данных от сервера строго указаных в запросе. Экономит количество запросов, количество передаваемой информации в запросе и повышает удобочитаемость. Но понижает скорость разработки.

---

# Безопасность
## Атаки

### ARP Spoofing
ARP - протокол для нахождения в сети нужных хостов - gateway для выхода в интернет, например. Транслирует IP в MAC. ARP не проверяет от кого пришел ответ и принимает response даже если не отсылал request. В IPv6 используется Neighbor Discovery Protocol (NDP). 

Разновидность MitM атаки. В локальной сети Ева отправляет поддельный response на все устройства, в котором говорится что MAC Евы - это MAC роутера и разных Алис и Бобов. Как итог - Алиса, Боб и остальные общаются с роутером через Еву.

Для Евы открываются такие возможности как:
 * Прослушивать трафик во всей локальной сети.
 * Воровать сессии.
 * Отправлять зловреды Алисе, Бобу...
 * Использовать ресурсы для DOS атак.

Меры противодействия:
 * Использовать VPN для безопасного\шифрованного подключения.
 * Использовать статический ARP.
 * Анализ трафика в сети, анализ подозрительных пакетов ARP.
