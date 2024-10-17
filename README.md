# Запросы Марафон
<h3>1.Все бегуны, которые зарегистрированы на текущий марафон отображаются в таблице, как: имя, фамилия - BibNumber (CountryCode).</h3>

```sql
select distinct u."firstname", u."lastname", rnr."countrycode"
from "User" u
inner JOIN "Role" r on u."roleid" = r."roleid"
inner join "Runner" rnr on u."email" = rnr."email"
inner join "Registration" reg on rnr."runnerid" = reg."runnerid"
inner join "RegistrationEvent" rei on reg."registrationid" = rei."registrationid"
inner join "Event" ev on rei."eventid" = ev."eventid" 
where r."roleid" = 'R' and ev."marathonid" = 5;
```

<h3>2.Таблица с результатами бегуна с предыдущих соревнований. Должен отображаться пол, возраст 
Список должен показать каждое соревнование, в котором бегун соревновался ранее.</h3> 

```sql
select g."gender",date_part('year', age(r.dateofbirth)), ev."eventname"
from "Runner" r
inner join "Registration" reg on r."runnerid" = reg."runnerid"
inner join "RegistrationEvent" rei on reg."registrationid" = rei."registrationid"
inner join "Event" ev on rei."eventid" = ev."eventid"
inner join "Gender" g on r."genderid" = g."genderid"
```

<h3>3.Выводятся следующие поля для каждого события:
• Марафон: полное название марафона.
• Событие: полное название события.
• Время: время гонки бегуна на события в часах, минутах и секундах.
• В целом положение бегуна в марафоне среди всех участников.
• Отдельно по категории положение бегуна на событии, среди бегунов одного того же пола и той же возрастной категории.
</h3> 

```sql
select m."marathonname", ev."eventname", to_char( (rei."racetime" ||'seconds')::interval, 'HH24:MI:SS' )
as race_time,
ROW_NUMBER() OVER(PARTITION BY ev."eventname" ORDER BY rei."racetime" asc) AS position
from "Event" ev
inner join "Marathon" m on ev."marathonid" = m."marathonid"
inner join "RegistrationEvent" rei on ev."eventid" = rei."eventid"
where ev."marathonid" = '3' and rei."racetime" != 0
order by eventname asc, race_time asc
```

<h3>4.1. Необходимо выводить сводные статистические данные по всем результатам, которые соответствуют критериям:
• Всего бегунов закончило: общее количество бегунов, у которые есть зафиксированное время соревнования.
• Среднее время гонки: Среднее значение всех результатов.
</h3> 

```sql
select ev."eventname", count(rei."racetime") as total_runners,
to_char( (round(avg(rei."racetime"), 0) ||'seconds')::interval, 'HH24:MI:SS' ) as avg_time
from "Event" ev
inner join "RegistrationEvent" rei on ev."eventid" = rei."eventid"
where rei."racetime" != 0
group by ev."eventname"
order by eventname
```

<h3>4.2. В списке показать всех бегунов, которые соответствуют критериям поиска. Следующие поля должны быть отображены:
• Ранг: положение бегуна для выбранного события, пола и возрастной категории.
• Время гонки: время гонки бегуна в категориях в часах, минутах и секундах.
• Имя бегуна: имя бегуна.
• Страна: страна бегуна.
• Должны отображаться данные бегуна, которая содержит следующую информацию: Фамилия Имя бегуна, страна, возраст в годах полных, на дату марафона, а далее в табличной форме
</h3>

```sql
select u."firstname", u."lastname",
date_part('Year', age(rnr.dateofbirth)) as age, g."gender", m."marathonname", ev."eventname", c."countryname",
TO_CHAR((rei."racetime" || ' seconds')::interval, 'HH24:MI:SS') AS race_time,
ROW_NUMBER() OVER (PARTITION BY ev."eventname" ORDER BY rei."racetime" ASC) AS position
FROM "Event" ev
INNER JOIN "Marathon" m ON ev."marathonid" = m."marathonid"
inner join "Country" c on m."countrycode" = c."countrycode"
INNER JOIN "RegistrationEvent" rei ON ev."eventid" = rei."eventid"
inner join "Registration" reg on rei."registrationid" = reg."registrationid"
inner join "Runner" rnr on reg."runnerid" = rnr."runnerid"
inner join "User" u on u."email" = rnr."email"
inner join "Gender" g on rnr."genderid" = g."genderid"
WHERE ev."marathonid" = '3' AND rei."racetime" != 0
ORDER BY ev."eventname" asc, g."gender" asc, race_time asc;
```
<h3>5.1. Запрос должен позволить просматривать список всех бегунов, которые зарегистрированы на текущий марафон. Составить запрос, который может фильтровать бегунов по статусу регистрации и выбранным типам марафона, и он может сортировать все поля таблицы результатов.</h3> 

```sql
select distinct u.firstname, u.lastname, rnr.countrycode, rgs.registrationstatus, ev.eventtypeid
from "User" u
inner JOIN "Role" r on u.roleid = r.roleid
inner join "Runner" rnr on u.email = rnr.email
inner join "Registration" reg on rnr.runnerid = reg.runnerid
inner join "RegistrationEvent" rei on reg."registrationid" = rei.registrationid
inner join "Event" ev on rei."eventid" = ev."eventid"
inner join "RegistrationStatus" rgs on reg.registrationstatusid = rgs.registrationstatusid
where rgs."registrationstatusid" = 1
order by eventtypeid asc;
```
<h3>5.2. Общее количество бегунов, которые выведены в список, должно быть отображено над списком. Имя, Фамилия, адрес электронной почты и регистрационный статус должны быть указаны для каждого бегуна.</h3> 

```sql
select count(*) from (select distinct u."firstname", u."lastname", rnr."countrycode", rgs.registrationstatus
from "User" u
inner JOIN "Role" r on u.roleid = r.roleid
inner join "Runner" rnr on u.email = rnr.email
inner join "Registration" reg on rnr.runnerid = reg.runnerid
inner join "RegistrationEvent" rei on reg."registrationid" = rei.registrationid
inner join "Event" ev on rei."eventid" = ev."eventid"
inner join "RegistrationStatus" rgs on reg.registrationstatusid = rgs.registrationstatusid
where ev."eventtypeid" = 'FM' and rgs."registrationstatusid" = 1) as  pup
```
<h3>6.Спонсоры должны быть сгруппированы по благотворительным организациям, которым они перечисляют деньги.
Также должны выводится сводные данные:
•Благотворительные организации: общее количество благотворительных организаций.
•Всего спонсорских взносов: общая сумма спонсорских пожертвований, полученная для всех благотворительных организаций.
Список должен показать наименование и общее количество спонсоров получил для каждой благотворительной организации.
</h3> 

```sql
select count(distinct c."charityname") as charity, sum(s."amount") as pop
from "Charity" c
inner join "Registration" r ON c."charityid" = r."charityid"
inner join "Sponsorship" s on r."registrationid" = s."registrationid"
group by c."charityname"  
order by charity;
```

<h3>7..Проверка бегуна по условиям:
Все поля обязательно заполнены.
Пароль должен отвечать следующим требованиям:
•	Минимум 6 символов
•	Минимум 1 прописная буква
•	Минимум 1 цифра
•	По крайней мере один из следующих символов: ! @ # $ % ^
"Дата рождения" должна быть правильной датой и бегуну должен быть не менее 10 лет на момент регистрации.
</h3>

```sql
select u."password", date_part('year', age(rnr.dateofbirth))
from "User" u
inner join "Runner" rnr on u.email = rnr.email
where u."password" like '______%'
and u."password" ~ '[a-z]'
and u."password" ~ '[1-9]'
and u."password" ~ '[!@#$%^]' 
and date_part('year', age(rnr.dateofbirth)) >10;
