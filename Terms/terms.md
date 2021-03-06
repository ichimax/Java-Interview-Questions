## Цель документа
_**Описать простыми словами термины, встречающиеся при разработке web-приложений:**_

1. [Схема взаимодействия между архитектурными слоями в web-приложении](#1)
2. [POJO](#2)
3. [JavaBean](#3)
4. [Entity](#4)
5. [ValueObject](#5)
6. [DTO](#6)
7. [DAO](#7)
8. [Repository](#8)
9. [DAO vs Repository](#9)
10. [REST](#10)


### 1. <a name="1">Схема взаимодействия между архитектурными слоями в web-приложении</a>

**`из браузера`**

- передаем `HTTP-запрос`, который попадает в

**`web (controller, view, ui) layer`**

- если - это `REST-controller` у которого есть внешнее `API` для сторонних приложений (внешних сервисов), то он принимает информацию снаружи через `REST-запросы` (по протоколу `HTTP`). Переданный в слой запрос (с параметрами) обрабатывается, и результатом его работы являются какие-то сущности (`entities`). Далее, этот слой обращается к

**`service layer`**

- в котором находится и выполняется над сущностями некая бизнес-логика. Если по ходу выполнения бизнес-логики Сервисному слою нужны данные из базы данных (или иного места хранения), то для этого он задействует

**`dao (repository) layer`**

- который в свою очередь обращается в базу (или другие источники хранения данных). Данный слой работает с сущностями предметной области, т.е. сохраняет или достает из источника хранения данных `entities`, которые преобразуются в `DTO` (т.е. объекты, которые приближены по своей структуре к объектам бизнес-логики) и передаются вверх по цепочку слоев, попадая в итоге наружу (клиенту, отправившему запрос)

**`domain (model, entity) layer`**

- слой, описывающий сущности предметной области без какой-либо логики (только данные), т.е. классы, структура которых максимально приближена к формату таблиц, хранящихся в базе данных (или иного источника)


### <a name="2">2. POJO</a>

**P**lain **O**ld **J**ava **O**bject, старый добрый Java-объект — это объект, состоящий чаще всего из набора полей, их геттеров/сеттеров и без дополнительной нагрузки в виде:

- Extend prespecified classes, as in
  ```Java
  public class MealServlet extends HttpServlet { ...
  ```
- Implement prespecified interfaces, as in
  ```Java
  public class Entity implements EntityBean { ...
  ```

- Contain prespecified annotations, as in (хотя [статья](https://spring.io/understanding/POJO) на сайте Спринга допускает наличие аннотаций для `POJO-объектов`)
  ```Java
  @javax.persistence.Entity
  public class Entity { ...
  ```

Термин `POJO` возник в качестве ответной реакции на появление платформы `J2EE` и ее широко распространившееся внедрение в приложениях, из-за чего, в частности, усложнился весь процесс их разработки. Мартин Фаулер с коллегами придумали данный термин для описания класса, свободного от «немого» кода, который требовался лишь для корректной работы среды выполнения

`POJO` был представлен в качестве альтернативы для `Enterprise JavaBeans (EJB)` и других «тяжелых» enterprise-конструкций, которые были популярны в 2000-х годах (об этом можно почитать в [статье](https://dou.ua/lenta/articles/javaee-vs-spring/) Сергея Немчинского)

`POJO` — это класс, который не использует специальные возможности различных фреймворков (т.е. он не прибит гвоздями к архитектуре какой-либо библиотеки, а также не привязан к фреймворку, который его использует [вспомним Spring и поулыбаемся]), таких, как `Spring`, `EJB` и пр. Данные фреймворки появились позже и поэтому в названии присутствует слово «старый»

**Пример:**

это `POJO`
```Java
class Meal {

    private int calories;

    public int getCalories() {
        return calories;
    }

    public void setCalories(int calories) {
        this.calories = calories;
    }
}
```

а это, нет 
```Java
class Calories {
    ...
    public int calculateCalories() {
        return 75 / (1,8 * 1,8);
    }
}
```
Основной целью `POJO` было показать, что сущности могут быть успешно смоделированы без использования `JavaBeans`. Более того, `JavaBeans` вообще не должны быть использованы для этой цели

Таким образом, основной посыл `POJO` — упрощение классов-сущностей насколько, насколько это возможно для моделирования предметной области

>Резюмируя все вышесказанное, можно сказать, что `POJO` — этот класс, который ничего не делает и имеет только состояние

### <a name="3">3. JavaBean</a>
>не путать с `Enterprise JavaBeans`
	
это класс в языке Java, написанный по определённым правилам:
- должен быть сериализуемыми (реализовывать интерфейс `java.io.Serializable`)
- должен иметь конструктор без аргументов
- все поля `JavaBean` должны быть закрытыми (private)
- доступа к полям осуществляется через методы доступа `getters` (аксессоры) и `setters` (мутаторы)
- методы `equals()`, `hashCode()` и `toString()` должны быть переопределены 
	
**Пример:**
```Java
public class Resume implements Serializable {

    private static final long serialVersionUID = 1L;

    private String uuid;

    private String fullName;

    public Resume() {
    }

    public String getUuid() {
        return uuid;
    }
    
    public void setUuid(String uuid) {
        this.uuid = uuid;
    }

    public String getFullName() {
        return fullName;
    }

    public void setFullName(String fullName) {
        this.fullName = fullName;
    }

    @Override
    public boolean equals(Object o) {
        if(this == o) return true;
        if(o == null || getClass() != o.getClass()) return false;
        Resume resume = (Resume) o;
        return Objects.equals(uuid, resume.uuid) &&
            Objects.equals(fullName, resume.fullName);
    }

    @Override
    public int hashCode() {
        return Objects.hash(uuid, fullName);
    }

    @Override
    public String toString() {
        return uuid + " - " + fullName;
    }
}
```

### <a name="4">4. Entity</a>
«Сущности» — это классы, моделирующие объекты предметной области
	
По своей структуре они приближены к таблицам базы данных (типы и названия столбцов таблицы == типам и названиям полей в классе, взаимодействующему с конкретной таблицей)

Хранятся в `domain` слое

Сущности обладают неотъемлемой идентичностью, основанной на эквивалентности их `id`. Это значит, что  если данные в двух сущностях полностью одинаковы (за исключением `id` поля), они не являются одной и той же сущностью

Сущности почти всегда изменяемы (мутабельны)


### <a name="5">5. Value object</a>
«Объект-значение» ⎼ термин из среды [`DDD`](https://ru.wikipedia.org/wiki/Проблемно-ориентированное_проектирование) ⎼ это объект без специальных методов, имеющий набор свойств (полей) примитивных типов данных или тоже `Value object`

Объекты данного рода проверяются на равенство исходя не из физической одинаковости (одинаковости ссылок на них), а из значений свойств

Примером `VO` является любой класс, который реализует равенство через равенство содержащихся в нем данных

	Два объекта Дат считаются равными, если равны их значения дня, месяца и года

В отличие от `Entity` не обладает идентичностью. На практике это означает, что объекты-значения не имеют поля-идентификатора: если два экземпляра одного объекта-значения обладают одинаковым набором атрибутов, то они равны

Объекты-значения должны быть неизменяемы (immutable). Это значит, что если требуется изменить такой объект, то для этого придется создать его новый экземпляр, вместо того чтобы изменять существующий

Объект-значение всегда должен принадлежать одной или нескольким сущностям, он не может жить собственной жизнью

Чтобы распознать объект-значение, мысленно замените его на `Integer`

Объекты-значения не должны иметь собственной таблицы в базе данных

**Пример**: `Integer`, `Money`, `Даты`


### <a name="6">6. DTO</a>
**D**ata **T**ransfer **O**bject, объект переноса данных ⎼ это паттерн, который предполагает использование отдельных классов для передачи данных (объектов без поведения) между слоями, c целью уменьшения количества запросов к базе данных

Данный класс, если его необходимо передавать по сети, должен быть сериализуемым (в `XML`, `JSON` и тд)

Это объект, который собирается из низкоуровневых данных (`entities`), а затем используется на слое `service` и `controller`. Те, при обращении к `Repository` из `service`, последний получает `entity` (набор табличных записей, которые возвращаются в результате выполнения `SQL-запроса`), преобразует его в `DTO` и отдает на слой `web`.

>`DTO` можно рассматривать как хранилище информации, единственная цель которого — передать данные получателю

Примером `DTO` является любой класс, который содержит только поля и методы по извлечению этих данных

`DTO` может собираться (состоять) из нескольких `entity`, взятых из базы данных
 
### <a name="7">7. DAO</a>
**D**ata **A**ccess **O**bject, объект доступа к данным — абстрактный интерфейс к какому-либо типу базы данных или иному механизму хранения

**Описание проблемы**

Способ доступа к данным бывает разным и зависит от источника данных. Способ доступа к базе данных зависит от типа хранилища (реляционные базы данных, объектно-ориентированные базы данных, однородные или «плоские» файлы и т.д.). Унифицированный API для доступа к этим несовместимым системам отсутствует. А использование конкретного способа доступа создает зависимость между кодом приложения и кодом доступа к данным

Такая зависимость кода может сделать миграцию приложения от одного типа источника данных к другому трудной и громоздкой

>Для доступа к данным хотелось бы использовать какой-либо универсальный способ (паттерн), позволяющий скрыть процесс их получения

Для решения вышеперечисленной проблемы обычно используют паттерн `DAO`

**Описание паттерна**

Данный паттерн абстрагирует и инкапсулирует доступ к источнику данных, а также управляет соединением с ним для получения и записи данных

В самом широком смысле, `DAO` — это класс, содержащий CRUD-методы для конкретной сущности

`DAO` полностью скрывает от клиента детали реализации взаимодействия с хранилищем данных. Поскольку при изменениях источника данных представляемый `DAO` интерфейс не изменяется, этот паттерн дает возможность `DAO` принимать различные схемы хранилищ без влияния на клиента или бизнес-компоненты. По существу, `DAO` выполняет функцию адаптера между компонентом и источником данных

Шаблон `DAO` используется для связи программы, написанной на Java с реляционными базами данных через интерфейс `JDBC`

`JDBC API` позволяет в приложениях использовать SQL-команды, являющиеся стандартным средством доступа к таблицам

Также `DAO` — это промежуточный слой, который скрывает от клиента реализацию взаимодействия с разными хранилищами данных, способы и механизмы хранения, предлагая при этом единые требования по функционалу в виде интерфейса
	 	 		 	 	
**Паттерн DAO позволяет:**

- отделить интерфейс(логика) от реализации: в начале определяем интерфейсы, а потом под них делаем любую реализацию в любом количестве, которую можно переключать в любое время — клиент ничего не заметит
- клиенту не зависеть от конкретного источника данных — ему все равно кто передает эти данные, он лишь дергает методы
- внедрять большое количество реализаций интерфейсов

**Проблемы DAO**

Большая часть людей представляет `DAO` некими вратами к базам данных, беспрепятственно добавляя в него множество методов для доступа к базе. Поэтому нередко можно увидеть слишком раздутое `DAO` c большим количеством методов на все случаи жизни

Чтобы уйти от этого, предлагается использовать паттерн `Repository`


### <a name="8">8. Repository</a>
	
«Хранилище» ⎼ паттерн, выполняющий роль коллекции объектов из `domain layer` в оперативной памяти

Хранилище позволяет добавлять или удалять объекты, как будто мы работаем с обычными коллекциями

Тот факт, что объект из `domain layer` на самом деле не находятся в хранилище, полностью скрыт от клиентских программ. Для них ⎼ это выглядит, как коллекция в памяти 

Данный слой используется в `Spring Data JPA`

### <a name="9">9. Разница между слоем DAO и Repository:</a>
	
- `DAO` ⎼ это низкоуровневый слой, который работает с объектами базы
- `Repository` ⎼ работает с объектами `DTO`
- Если `Entity` не переводятся в `DTO`, то два этих слоя сливаются, выполняя в итоге одну и туже функцию, т.е. `Repository` становится `DAO`


### <a name="10">10. REST</a>
**RE**presentational **S**tate **T**ransfer, передача состояния представления

`REST` определяет набор операций, которые, например `User` может выполнять над `Meals`

`REST-Controller` - способ общения с внешними приложениями через установленный набор методов, за которые они могли бы дёргать наше приложение

---
**Дополнительная информация:**
- [Хорошее видео про DAO](https://www.youtube.com/watch?v=B9vZzf65LXs)
- [Книга - Spring 4 для профессионалов.](http://www.ozon.ru/context/detail/id/33056979/) В главе "Поддержка JDBC в Spring" дается описание недостатков реализации DAO с помощью чистого JDBC
