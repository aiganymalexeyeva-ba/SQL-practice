# SQL-practice

-[Фильтрация данных (WHERE)](#Фильтрация-данных-(WHERE))

-[Создание групп](#Создание-групп)

-[Объединение таблиц (JOIN)](#Объединение-таблиц-(JOIN))

-[Обновление и удаление данных (UPDATE и DELETE)](#Обновление-и-удаление-данных-(UPDATE-и-DELETE))


## 1. Фильтрация данных (WHERE)
#### Напишите запрос, который выведет Номер [Number], Дату [Date] и Сумму [Amount] заказа для всех строк таблицы [Order].
```sql
SELECT NUMBER, DATE, AMOUNT
FROM [ORDER]
```

#### Напишите запрос, который выведет названия [Name] контрагентов (таблица [Account]),где полнота заполнения профиля [Completeness] равна значению 90.
```sql
SELECT NAME, COMPLETENESS
FROM [ACCOUNT]
WHERE COMPLETENESS = 90
```

#### Напишите запрос, который выведет информацию по счетам (таблица [Invoice]), у которых дата окончания действия [DueDate] находиться в промежутке между 1 и 31 марта 2017 г. В реестр выборки вывести Номер ([Number]) и дату окончания действия ([DueDate]) счета.
```sql
SELECT NUMBER, i.DueDate
FROM Invoice i 
WHERE DUEDATE BETWEEN '2017-03-1' AND '2017-03-31'
ORDER BY DueDate
```

#### Напишите запрос, который выведет всех контрагентов (таблица [Account]), имена [Name] которых начинаются на любую из букв от ‘А'; до ‘К';.
```sql
SELECT NAME FROM ACCOUNT
WHERE NAME LIKE N'[А-К]%'
ORDER BY NAME DESC
```

#### Напишите запрос, который выведет всех контрагентов (таблица [Account]), у которых в поле категория ([AccountCategoryId]) пусто и не заполнено поле форма собственности ([OwnershipId])
```sql
SELECT NAME, AccountCategoryId,OwnershipId
FROM ACCOUNT
WHERE AccountCategoryId IS NULL
AND OwnershipId IS NULL
```

#### Напишите запрос, который выбирает всех контрагентов (таблица Account), у которых в поле категория ([AccountCategoryId]) пусто и выполняется одно из условий:  Заполнено поле Форма собственности ([OwnershipId]) или Полнота наполнения данными ([Completeness]) находиться в диапазоне между 80 и 100%.
```sql
SELECT NAME, AccountCategoryId, OwnershipId , Completeness FROM ACCOUNT
WHERE AccountCategoryId IS NULL
AND OwnershipId IS NOT NULL OR Completeness >80
```

## 2. Создание групп
####  Напишите запрос, который покажет количество контрагентов [Account] в каждой отрасли [IndustryId].
```sql
SELECT
COUNT (INDUSTRYID) AS ContactWithIndustryId
FROM ACCOUNT
```

#### Напишите запрос, который выберет наименьшую сумму [Amount] заказа (таблица [Order]) по каждому контрагенту [AccountId].
```sql
SELECT
MIN (o.Amount) 
FROM [Order] o
```

#### Необходимо получить информацию о количестве заказов (таблица [Order]) в разрезе статуса [StatusId] . В список необходимо вывести только те статусы, на которых находятся от 5 заказов (включительно).
```sql
SELECT COUNT(*), StatusID
from [order]
group by StatusId
having count(*)>=5
```

#### Напишите запрос к таблице с контрагентами ([Account]), который для каждого типа контрагента ([TypeId]) определит максимальный полноту заполнения профиля ([Completeness]). Полученному значению присвоить псевдоним (alias) - MaxCompletness. Отсортируйте информацию по значению «максимальная полнота заполнения профиля» в порядке убывания.
```sql
select TypeId,
max(Completeness) as MaxCompleteness
from account
group by TypeId
order by MaxCompleteness desc
```

#### Необходимо вывести по контактам (таблица [Contact]) следующую информацию: 
#### 1. ФИО контакта [Name].
#### 2. Название должности [JobTitle] в верхнем регистре.
#### 3. Конкатенация значений ФИО и Название должности. в формате [ФИО контакта] (Должность: [Название должности]), например «Петриченко Кирилл Олегович (Должность: Коммерческий директор)». Полученному значению присвоить alias NameWithJob.
#### 4. Колонку, которая будет содержать информацию о длине значения в поле ФИО.
```sql
SELECT NAME, UPPER(JOBTITLE) as JobTitle,
NAME + N' Должность: (' + c.JobTitle + ')'AS NameWithJob,
LEN(NAME) AS NameLength
FROM Contact c
```

#### Необходимо получить следующую агрегированную информацию по данным в таблице "Заказ"; ([Order]): 
#### 1. Количество записей в таблице
#### 2. Сумму по колонке "Сумма заказа"; ([Amount]). 
#### 3. Максимальное значение в колонке "Сумма заказа"; ([Amount]).
#### 4. Минимальное значение в колонке "Сумма заказа"; ([Amount]).
#### 5. Среднее значение в колонке "Сумма заказа"; ([Amount]).
#### 6. Разницу между средним значением "Сумма заказа"; ([Amount]) и средним значением &quot;Сумма оплаты&quot; ([PaymentAmount]). В вычислении необходимо учитывать только уникальные значения
```sql
SELECT
COUNT(*),
SUM(AMOUNT) AS Sum,
MAX(AMOUNT) AS MaxAmount,
MIN(AMOUNT) AS MinAmount,
AVG(AMOUNT) AS Avarage,
AVG(distinct AMOUNT)-AVG(distinct PaymentAmount) as AvarageAmountAndPaymentAmountDifference
FROM [Order] o 
```

## 3. Объединение таблиц (JOIN)

#### С помощью подзапроса найдите все заказы (Таблица Order), которые были оформлены по контрагенту с именем (Account.Name) = «Аксиома» (поиск по имени, предположим, что его уникальный идентификатор неизвестен). В реестр выборки вывести Номер (Number) и Дату заказа (Date).
```sql
select o.Number, o.[Date] 
from [Order] o
where o.AccountId  IN
(select a.Id 
from Account a
where a.Name = N'Аксиома')
```

#### Необходимо получить Топ 5 контрагентов (таблица Account) по общему количеству заказов. В реестр вывести Название контрагента (Name) и общее количество заказов по контрагенту (данные из таблицы Order). Важно: Поиск количества заказов выполнить с помощью подзапроса.
```sql
select top 5
name,
(select COUNT(*) 
from [Order] o
where o.AccountId  = a.Id ) as AmountOfOrders from Account a
order by AmountOfOrders desc
```

#### Необходимо получить список контактов (таблица Contact) с информацией по счетам (таблица Invoice). В реестр вывести ФИО контакта (Contact.Name) и номер заказа (Invoice.Number). Важно: Список должен включать только тех, контактов, у которых есть счета
```sql
select
Contact.Name as ContactName,
Invoice.number as InvoiceNumber
from contact,
Invoice
WHERE Contact.ID =INVOICE.ContactId
```

#### Необходимо получить список контактов (таблица Contact) с информацией по счетам (таблица Invoice). В реестр вывести ФИО контакта (Contact.Name) и номер заказа (Invoice.Number). Важно: Список должен включать всех контактов, даже тех, у которых нет счетов
```sql
select
Contact.Name as ContactName,
Invoice.number as InvoiceNumber
from contact
left join Invoice
on Contact.ID =INVOICE.ContactId
```

#### Необходимо получить список контрагентов (таблица Account), у которых есть родительская компания (ParentId) и у этой родительской компании категория (AccountCategoryId) = "A";.
```sql
select a.Name, a.ParentId 
from Account a
where a.ParentId in (
select a.ParentId 
from Account a
where a.AccountCategoryId  in
(select ac.Id 
from AccountCategory ac 
where ac.Name = 'A'))
```

## 4. Обновление и удаление данных (UPDATE и DELETE)

#### Напишите запрос, который выведет всех клиентов, как контактов (таблица Contact), так и контрагентов (таблица Account). Выборка должна содержать 2 поля:
#### Тип клиента: Контакт или Контрагент
#### Название клиента: для контактов – это поле ФИО (Contact.Name), для контрагентов – Название компании (Account.Name)
#### Важно: Запрос необходимо сформировать, используя UNION
```sql
select N'Контакт' as N'Тип клиента',
c.Name  as Name 
from contact c
UNION
select N'Контрагент' as N'Тип клиента' ,
a.Name as Name
from Account a
```

#### Скопируйте данные справочника Города (таблица City) в новую таблицу. Для создания новой таблицы используйте оператор SELECT INTO. Название новой таблицы сформировать по следующему правилу: CitySurname. Где Surname - ваша фамилия. Например, CityKiper
```sql
select *  into CityAlexeyeva
from City 
```

#### Подготовить запрос на добавления 3-х новых городов в Нидерландах в созданную ранее таблицу (см. предыдущее задание): Rotterdam, Den Haag и Utrecht
```sql
 insert into
CityAlexeyeva (Id, Name, Description,CountryId)
VALUES 
(NewId(), 'Rotterdam','New Record', '4A5F3DF5-5818-4D3D-8992-3F6D505E0369'),
(NewId(), 'Den Haag','New Record', '4A5F3DF5-5818-4D3D-8992-3F6D505E0369'),
(NewId(), 'Utrecht','New Record', '4A5F3DF5-5818-4D3D-8992-3F6D505E0369')
```

#### Для созданной ранее таблицы (см. 8 задание) подготовить запрос на обновление города Den Haag. Название должно содержать значение "The Hague";
```sql
update CityAlexeyeva
set Name = 'The Hague'
where ID = '30CC7085-8760-4D23-A814-C6679CDF8F33'
```

#### Подготовить запрос на удаление 1 записи из таблицы, которая была создана в 2 задании. Выбор записи на ваше усмотрение.
```sql
DELETE FROM CityAlexeyeva
WHERE Id = '898BBA99-AB0D-449A-9F9A-3A08AC481448'
```
