# System Design для социальной сети "Впути"
Разработка системного дизайна социальной сети Впути для путешественников и всех, кто
хочет узнать больше о красивых и интересных местах (локациях) нашей планеты. 
## Функциональные требования
- профиль пользователя (фамилия, имя, описание, фотография, емейл, юзернейм)
- публикация постов
- - пост = название + текст + фотографии + локация + оценка локации от (1 до 10)
- - локация это название, страна, провинция, город и геометка
- - во время создания поста должна быть возможность, как добавлять локации самому, так и искать локации по списку уже созданных
- реакции под постами
- - возможность поставить лайк
- - возможность оставить комментарий (только текст)
- - лента комментариев в обр. хрон. порядке
- подписка на путешественников
- - лента, где показываются посты путешественников, на которые ты подписан в обр. хрон. порядке
- - уведомления о новых постах у путешественников, на которых ты подписался
- поиск локаций по названию
- - должно работать автодополнение
- - возможность видеть посты, связанные с этой локацией, отсортированные либо в обратном хрон порядке, либо по популярности (количеству комментариев). лента с постами "бесконечная"
## Нефункциональные требования
- лимиты на данные
- - название поста - 100 символов макс
- - текст поста - 4000 символов макс
- - фотографии - 10 штук на пост макс. максимальный размер 4мб. средний размер 2мб. среднее количество фоток в посте 5. фотографии должны сжиматься для последующего показа. средний размер сжатой фотографии 200кб.
- - текст комментария - 1000 символов макс 
- - лимиты для локаций: 100 символов на каждое текстовое поле. итого 400 символов.
- данные храним всегда
- аудитория
- - dau 10 000 000
- - есть сезонность. летом трафик вырастает в 3 раза.
- - регион: СНГ
- лимиты на операции пользователя
- - 50 постов в день
- - лимитов на операции чтения нет
- - макс количество подписчиков 1 000 000
- - макс количество подписок 100 000
- - макс количество лайков в день 100
- - макс количество комментариев в день 100
- активность пользователей (в среднем)
- - 2 поста в неделю
- - - когда создает пост, то требуется 5 запросов (автодополнение), чтоб найти локацию
- - 2 раза в день проверяет ленту
- - - примерно по 10 постов читает за один заход (суммарно 20 постов в день)
- - - под каждым четвертым постом читает комментарии
- - - читает 10 комментариев под постом
- - - каждому второму ставит лайк
- - - каждому десятому пишет комментарий
- - 2 раза в день читает посты про определенной локации (переходит с ленты)
- - - 3 локации по 5 постов
- - - Здесь лайки и комментарии пользователь практически не ставит
- - 1 раз в неделю прицельно ищет локаци и читает про нее
- - - требуется 5 запросов (автодополнение), чтоб найти локацию
- - - Один запрос возвращает 10 результатов
- - - читает по 10 постов
- 5 отписок / подписок в неделю
## Оценка нагрузки
### Посты
#### RPS(read).

В ленте и поиске посты грузятся батчами по 5 шт.
За один просмотр ленты пользователь сходит 3 раза за постами (2 основных, чтоб 10 постов получить и еще один чтоб дальше ленту подгрузить). Также пользователь с ленты переходит на конкретные локации и читает про них. На просмотр одной локации будет тратиться 2 запроса. Пренебрегаем тем, что пользователь раз в неделю сам прицельно ищет информацию про какую-то локацию.

Итого, пользователь 2 раза в день сделает 3 запроса в ленте + 3 (локации) * 2 запроса. Это 18 запросов на батчи постов (по 5 штук) в день. Округлим до 20. 

_RPS (read) = 10 000 000 (пользователей) * 20 / 100000 = 2000_

Отдельно посчитаем картинки в посте.

_RPS (read картинки) = 20 000_

#### RPS (write)

2 поста в неделю, т.е примерно 0.3 поста в день.

_RPS (write) = 10 000 000 * 0.3 / 100000  = 30_

_RPS (write картинки) = 300_

#### Traffic (read)

Основной объем данных займет текст поста. Кодировка utf-8, и средний размер символа 3 байта. Размер текста 3 * 4000 = 12 000 байт. 

_traffic (read) = 2000 * 12 000 байт = 2.4 * 10 ^ 7 b/s = 24 mb/s_

Картинки в посте 0.2 * 5 = 1 mb.

_traffic (read картинки) = 2000 * 1 mb = 2 gb/s_

#### Traffic (write)

Трафик на запись оценю через соотношение RPS

_traffic (write) = 24 * (30 / 2000) = 0.4 mb/s_

Средний размер картинки при загрузке 2мб, а картинок 5

_traffic (write картинки) = 300 * 10 = 300 mb/s_

### Лайки

Количество лайков, это просто число в посте. Поэтому оценим лишь RPS (write). В сутки пользователь ставит 10 лайков.

#### RPS (write)

_rps (write) = 10 000 000 * 10 / 100 000 = 1000_

#### Traffic (write)

Пейлоада нет. Основной трафик - сервисная информация (токены, хэдеры). Допустим, эта информация будет около 500 байт.

_traffic (write) = 1000 * 500 = 500 kb/s_

### Комментарии

#### RPS (read)

Пользователь открывает ветки с комментариями 5 раз в день. Комментарии грузятся батчами по 20. Поскольку в среднем пользователь читает 10 комментариев, то будет достаточно 1го запроса, чтоб выгрузить нужные комменты под постом.

_rps (read) = 10 000 000 * 5 / 100 000 = 500_

#### RPS (write)

Пользователь пишет комментарий 2 раза в день.

_rps(write) = 10 000 000 * 2 / 100 000 = 200_

#### Traffic (read)

Один коммент 1000 символов или 3000 байт

_traffic (read) = 500 * 3000 * 20 = 3 * 10 ^ 7 b/s = 30 mb/s_

#### Traffic (write)

_traffic (write) = 200 * 3000 = 600 kb/s_

### Локации

#### RPS (read)

Два места, для поиска локации: создание поста и поиск по локации. Когда создаем посты, то делаем 10 запросов в неделю. И еще 5 запросов на поиск и чтение про локацию. Это 15 запросов в неделю или 2 запроса в день.

_rps (read) = 10 000 000 * 2 / 100 000 = 200_

#### Traffic (read)

Локация это 400 символов + геометка (пренебрегаем). То есть 1200 байт. Один запрос возвращает 10 результатов.

_traffic (read) = 200 * 1200 * 10 = 2.4 mb/s_

Что касается операций записи, то со временем ситуаций, когда надо будет добавлять локации, будет мало, т.к. "все" локации уже будут существовать, за исключением экзотических. Поэтому мы можем это не учитывать в нашей оценке.

## Оценка дисков
### Посты
#### Capacity
0.4 mb / s * (100000 * 400)s = 1.6 * 10 ^ 7 mb = 16 tb 
#### HDD
* iops: 2000 rps => 20 
* traffic: 24 mb / s => 1
* capacity: 16 tb => 1

20 дисков по 1 тб

#### SSD (sata)
* iops: ~2000 rps => 2
* traffic: 1
* capacity: 1

2 ssd (sata) 8 tb либо 1 (nVME)

Выбираем 20 HDD дисков по 1 тб, т.к. iops нагрузка.

### Лайки
### Capacity
Трафика на лайки нет. Оценим емкость иначе. В день пользователь ставит 10 лайков. Количество лайков в год.

10 000 000 * 10 * 365 (400) =  4 * 10 ^ 10.

Одна запись в таблице likes 32 байта.

32 * 4 * 10 ^ 10 = ~ 1тб

#### HDD
* iops: ~1000 rps => 10
* traffic: 1
* capacity: 1

10 дисков по 100 гб 

#### SSD (sata)
* iops: ~1000 rps => 1
* traffic: 1
* capacity: 1

1 ssd (sata) по 1 тб

Выбираем 10 HDD дисков, т.к. iops нагрузка + дешевле.

### Посты (картинки)
#### Capacity
Сожмем картинки в 2 раза.

0.5 * 300 mb / s * (100 000 * 400)s = 6 * 10 ^ 9 mb = 6 * 10 ^ 3 tb = 6 pb
#### HDD
* iops: 20 000 rps => 200
* traffic: 2 gb / s => 20
* capacity: 200

200 дисков по 32 тб

#### SSD (sata)
* iops: 20
* traffic: 4
* capacity: 60

60 ssd (sata) по 100 тб

Выбираем 60 ssd по 100 тб, т.к. храним много данных.

### Комментарии
#### Capacity
600 kb/s * (100 000 * 400)s = 24 tb
#### HDD
* iops: 700 rps => 7
* traffic: 30 mb /s => 1
* capacity: 1

10 дисков по 4 tb

#### SSD (sata)
* iops: 700 rps => 1
* traffic: 30 mb /s => 1
* capacity: 1

2 ssd (sata) по 16 tb

Выбираем 10 HDD дисков, т.к. характер нагрузки iops + дешевле.


## Хранение данных
### Посты

* async RF 3 master-slave (преобладает read)
* 2 disks by host => 10 shards * 3 => 30 hosts

### Лайки

* RF 1 (преобладает write)
* 2 disks by host => 5 shards => 5 hosts

### Посты (картинки)

* RF 1 (дорого будет реплики делать)
* 6 ssd by host => 10 shards => 10 hosts

### Комментарии

* async RF 3 master-slave (преобладает read)
* 2 disks by host => 5 shards * 3 => 15 hosts