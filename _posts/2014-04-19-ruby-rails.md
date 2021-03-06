---
author: D-side
date: 2014-04-19
layout: post
title: Ruby &mdash; рельсы
tags:
- Программирование
- Веб
- Ruby/Rails
---
Что вообще такое Rails? Это библиотека. Это философия. Это приёмы и соглашения. Но, в первую очередь, это целый набор всякого разного программного обеспечения, связанного одной целью &mdash; лёгкая разработка веб-приложений, соответствующих всяким хорошим нормам. Это фреймворк, но основанный во многом на отдельных частях, причём они работают вместе так слаженно, что возникает вопрос, что из этого является "используемым" рельсами, а что "частью" рельс. Стоп-стоп-стоп, я даже не рассказал, что такое фреймворк, а уже почти ругаюсь другими страшными словами.

**Фреймворк**, он же *framework*, очень странный зверь. Начнём с названия. Всё слово буквально перевести сложно, но образован он из `frame` (рамка) и `work` (работа). Строгого определения не имеет. Но привести несколько примеров и указать общие черты можно. Собственно, возьмём Rails, Qt, Polycode, .NET и... нет, хватит. Что у них общего? Это не программы. Их нельзя просто скомпилировать и запустить. Это библиотеки, на основе которых предлагается собрать собственную программу. Вместе с тем, они обычно не требуют использования своей IDE (среды разработки), и этим отличаются, например, от игровых конструкторов (GMS же не фреймворк). Ещё они, как правило, весьма избыточны, и реализуют многие стандартные для языка вещи, потому что стандартные реализации недостаточно круты (см. `std::string` и `QString`). Надеюсь, этого достаточно для какого-никакого представления о том, что есть фреймворк.

Для чего нужен Rails? Для быстрого сооружения приложения, которое должно работать в основном на сервере, а доступ осуществляться через браузер. Скажем, хорошая пара: Rails + GMS + HTML5, вы можете собрать так называемый back-end (серверную часть) на Rails, написать клиент на GMS, скомпилировать результат в HTML5 и таким образом получить... онлайновую игру! Но придержите лошадей, вам ещё хостинг искать, и-и-и-хи-хи `:D`. На время разработки вы сможете разместиться на Heroku или даже собственном компьютере, конечно. Но должен вас предупредить, что достаточно быстрое приложение на рельсах написать непросто. Нужно следить за массой мелочей. Это цена той простоты, которой обладает Rails, вы легко соберёте то, что хотите (вам необязательно разбираться в каждой технологии, которая там будет работать), но стоит вам развернуть это не только для собственных нужд, возникнет масса вопросов.

Что касается изучения Rails &mdash; материалов об этом полно. Есть видеокурс "Rails for Zombies" (где пишут твиттер для зомби, я серьёзно). Есть книга Майкла Хартла "Rails Tutorial" (я сам с неё начинал &mdash; там много кода, который выглядит как заклинания и слабо описывается, но это работает). Я же кратко расскажу о том, в каком порядке в Rails строится приложение. Я лично решаю с их помощью уже вторую задачу (реальную задачу, не учебную).

Итак, у вас есть идея. Чтобы с чего-то начать, вам нужно определиться, какие данные для реализации идеи необходимо обрабатывать. В первой решённой мной задаче речь шла о файлах: мы заходим на узел, загружаем туда файл, узел его сохраняет по постоянной ссылке и отдаёт её пользователю. То есть, модель данных у нас всего одна &mdash; файл. В неё входят:

* файл (разумеется)
* срок годности (истечёт &mdash; стереть)
* пароль (владельца файла, который может файл стереть или "освежить")
* какие-то служебные поля, которые сейчас нас волнуют мало (id, момент создания, момент изменения)

Дальше надо подумать, какие из решаемых вами задач точно решались другими рубистами ранее. К примеру, вместо того, чтобы городить огород с сохранением файлов, можно воспользоваться библиотекой `paperclip`, которая предоставляет ряд интересностей. Это, в основном, вещи, которые надо бы написать, но лень. К примеру, для "моделей данных" в Rails есть "валидации" (проверки на корректность данных). Конкретно `paperclip` позволяет наложить ограничения на размер файла, тип, позволяет преобразовывать изображения (решение, во многом, предполагалось для лёгкой реализации аватаров пользователей). А ещё для ежечасного удаления устаревших файлов можно воспользоваться `whenever`, о котором я писал раньше &mdash; он позволяет сделать приложению "расписание", по которому оно должно выполнять задания. Но задание нужно ещё сделать, верно?

Так продолжим с моделями. Модель &mdash; данные, которые подлежат обработке. И их надо где-то хранить (хоть и не всегда)! За это, если не углубляться, отвечает `ActiveRecord`. Это потрясающая штука, и я сейчас объясню, почему. Вам приходилось сталкиваться с реляционными базами данных? Ну, о MySQL вы наверняка слышали &mdash; это неплохой пример. Видели язык, на котором к базе пишутся запросы? Вот ActiveRecord заворачивает это дело в нехитрый язык с ООП и красивым синтаксисом. Здесь я хочу привести пример, на котором понял, что мне надо подучить SQL, потому что я не понимаю, что это за магия. Почувствуйте разницу:

{%highlight ruby%}
Page.find(5).blocks
{%endhighlight%}
{%highlight sql%}
SELECT "pages".* FROM "pages" WHERE "pages"."id" = $1 LIMIT 1 [["id", 5]]
SELECT "blocks".* from "blocks" INNER JOIN "links" ON "blocks"."id" = "links"."block_id" WHERE "links"."page_id" = $1 [["page_id", 5]]
{%endhighlight%}
Один класс, два метода: килограмм SQL. Первый запрос простой (найти не более 1 элемента с `id`=`5`). На втором остановимся подробнее.

У меня есть три модели: страницы, блоки и связующие их ссылки. На каждой странице может быть много блоков, но каждый блок может встречаться на нескольких страницах. Поэтому для указания того, что на некоторой странице лежит некоторый блок, используется ссылка: она содержит номера, `что` (блок) находится `где` (страница).

Семантика (смысл) запроса выше &mdash; найди мне пятую страницу и выведи все её блоки. Изнутри во втором запросе происходит нечто монструозное: делается выборка ссылок, к которым прикручивается (inner join) информация о соответствующих блоках, и из них выбираются только те, у которых id страницы равен 5. То, что оно просто выглядит, не означает, что оно просто работает. Моя работа сейчас &mdash; обеспечить контраст, сделать что-то очень полезное, но при этом простое на вид.

К слову, здесь же демонстрируется основной недостаток такой системы. Она работает неоптимально, делает лишние действия, поскольку не знает о природе вашей задачи. Первый запрос, фактически, делать необязательно, нам не нужен его результат. Как это записать в ActiveRecord (без SQL), я не очень понял. Вероятно, напишу с этим на StackOverflow, средоточие программистов всех мастей. Первый запрос выясняет все пятые страницы. Но id &mdash; первичный ключ, и страница с конкретным значением ключа может по умолчанию быть либо одна, либо ни одной. В условиях моей задачи на несуществующую страницу ссылки указывать не могут, но ActiveRecord идёт наиболее безопасным и буквальным путём. Нельзя судить его за это.

Так. Ладно. У нас есть удобный способ хранить информацию. Теперь надо бы научиться её изменять. Чтобы изменять &mdash; нужно сделать запрос к системе. Любой запрос по HTTP попадёт сначала на роутер, который разберётся, чего хочет этот запрос. Если он соответствует некоторому шаблону &mdash; роутер отправляет данные из запроса в место, указанное в шаблоне. Что за место? Действие.

Тут мы приходим к контроллерам, которые содержат действия. Когда пользователь хочет совершить действие, он дёргает контроллер. Контроллер совершает с моделями некоторое действие, после чего должен вывести результат. А результат, как правило, должен быть представлен в виде веб-страницы. Поэтому смотрим последнее звено цепочки: виды.

Вид &mdash; штука несложная, это попросту шаблон для данных. Контроллер может вытащить набор данных, который будет вставлен в вид и послан пользователю. В простейшем случае виды пишутся на ERB, Embedded Ruby. Он очень прост в понимании: берём страницу и вставляем туда по кусочкам Ruby, ограничивая `<%` и `%>`. Причём если открывать конструкцией `<%=`, то результат выполнения кода будет вставлен в страницу. Но для шаблонов такой подход плохо годится. Поэтому приведу альтернативу: HAML. Мне лень соображать собственный пример, поэтому приведу пример из документации к HAML:

Типичный код на ERB:
{%highlight erb%}
<div id='content'>
  <div class='left column'>
    <h2>Welcome to our site!</h2>
    <p><%= print_information %></p>
  </div>
  <div class="right column">
    <%= render partial: "sidebar" %>
  </div>
</div>
{%endhighlight%}
Он же на HAML:
{%highlight haml%}
#content
  .left.column
    %h2 Welcome to our site!
    %p= print_information
  .right.column
    = render partial: "sidebar"
{%endhighlight%}

Намного короче, однако. За счёт чего? В основном из-за закрывающих тегов. Их нет. Считается, что в указанный тег входит всё, что расположено в блоке после него. Каждая строчка в блоке выделяется дополнительной парой пробелов по сравнению с тегом. Блок кончится тогда, когда будет встречена строчка наравне с тегом. В понимании этого кода резко поможет "линейка". Грубо говоря, нечто такое:
{%highlight text%}
#c|on|tent
  |.l|eft.column
  |  |%h2 Welcome to our site!
  |  |%p= print_information
  |.r|ight.column
  |  |= render partial: "sidebar"
{%endhighlight%}
С такими "росчерками" по коду становится немножко понятнее, что такое блок. Всё, что под первыми двумя символами тега не содержит никаких символов, лежит внутри него. И я сначала не очень верил, что это существенно ускоряет дело, пока не установил и не проверил. Чем HAML интересен? Он короче. В него быстрее вносятся изменения. Распространённые конструкции в HAML по сравнению с HTML сокращены почти до неприличия:

HTML:
{%highlight html%}
<div id='content'>
</div>
{%endhighlight%}
HAML:
{%highlight haml%}
#content
{%endhighlight%}
Но не надо считать, что HAML лучше. Это не так. HAML прекрасен, когда речь идёт о структуре, содержимое в которую подаётся программным кодом. Подавать содержимое в HAML руками - адский ад. Даже по сравнению с HTML. А всё из-за правила блоков. Посмотрите, что я имею в виду:

HTML:
{%highlight html%}
А здесь я просто хочу написать немного <b>полужирного</b> текста.
{%endhighlight%}
HAML:
{%highlight haml%}
А здесь я просто хочу написать немного
  %b полужирного
текста.
{%endhighlight%}
Ну и Markdown, для сравнения:
{%highlight text%}
А здесь я просто хочу написать немного **полужирного** текста.
{%endhighlight%}
И это ещё примитивный пример, который на практике разрастается, приводя к целым весёлым лесенкам. Понятно, где слабость HAML, да? Но поскольку в основном мы описываем не данные, а их структуру, для видов HAML определённо силён.

Так, мы покрыли "модели", "контроллеры" и "виды". Всё это формирует распространённый шаблон построения ("паттерн проектирования", будь этот термин неладен) приложений: MVC, Model-View-Controller. Мы обращаемся к контроллеру, контроллер обращается к модели, модель отдаёт данные виду, вид выводит данные пользователю. Грубо говоря. Не без исключений, конечно.

В ближайшее время постараюсь рассказать о том, что и появилось не так давно, и я начал изучать совсем недавно. Peer-to-peer в HTML5, WebRTC.
