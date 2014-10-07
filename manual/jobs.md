---
title: Фоновые службы
layout: manual
---

Revel имеет мощный инструмент для запуска фоновых (аснхронных) служб,
выполняющихся независимо от обработки запросов.
Это могут быть переодические задачи по обновлению кэша или разовые по отправке
сообщения на почту.
<!--
Revel provides a framework for performing work asynchronously, outside of the
request flow.  This may take the form of recurring tasks that update cached data
or one-off tasks for sending emails.
-->

## Активация

Службы опциональны и для того, чтобы их использовать их нужно сначала активировать.
Для активации добавьте этот модуль в файл конфигураций приложения:
<!--
The framework is included as an optional module, that is not included in your
application by default.  To activate it, add the module to your app
configuration:
-->

	module.jobs = github.com/revel/revel/modules/jobs

В дополнение, для получения доступа к странице мониторинга служб, Вы можете
добавить эту строчку к маршрутам:
<!--
Additionally, in order to access the job monitoring page, you will need to add
this line to your routes:
-->

	module:jobs

Это добавит маршрут `/@jobs`
<!--
This statement will insert the `/@jobs` route at that location.
-->

## Опции

Эта пара опций определяет какого рода ограничения будут наложены на службы
<!--
There are a couple of options that tell the framework what sort of limitations
to place on the jobs that it runs.
-->

В примере показаны значения по умолчанию
<!--
This example shows them set to their default values.-->

    jobs.pool = 10                # Максимально допустимое количество одновременно запущенных служб
    jobs.selfconcurrent = false   # Позволять запускать только один экземпляр службы

## Запуск служб

Для запуска задачи при запуске приложения, используйте
[`revel.OnAppStart`](../docs/godoc/server.html#OnAppStart) для регистрации функции.
Revel запустит эти задачи последовательно перед запуском сервера. Заметим, что
этот подход не использует модуль jobs, но этот модуль может быть использован
для запуска задач, так чтобы сервер не блокировался при старте.
<!--
To run a task on application startup, use
[`revel.OnAppStart`](../docs/godoc/server.html#OnAppStart) to register a function.
Revel runs these tasks serially, before starting the server.  Note that this
functionality does not actually use the jobs module, but it can be used to
submit a job for execution that doesn't block server startup.-->

{% raw %}
<pre class="prettyprint lang-go">
func init() {
    revel.OnAppStart(func() { jobs.Now(populateCache{}) })
}
</pre>
{% endraw %}

## Повторяющиеся задачи

Служба может быть запущена по любому графику. Есть два варианта:
<!--
Jobs may be scheduled to run on any schedule.  There are two options for expressing the schedule:-->

1. Спецификация `cron`
2. Фиксированные интервалы

<!--1. A cron specification  2. A fixed interval-->

Revel использует [библиотеку cron](https://github.com/revel/cron) для парсинга расписания
и запуска службы. Детальное описание формата Вы можете найти в
[README](https://github.com/revel/cron/blob/master/README.md) библиотеки.
<!--
Revel uses the [cron library](https://github.com/revel/cron) to parse the
schedule and run the jobs.  The library's
[README](https://github.com/revel/cron/blob/master/README.md) provides a detailed
description of the format accepted.-->

В основном службы регистрируются с помощью хуков
[`revel.OnAppStart`](../docs/godoc/server.html#OnAppStart), но, в принципе,
они могут быть зарегистрированны позднее в любое время.

<!--
Jobs are generally registered using the
[`revel.OnAppStart`](../docs/godoc/server.html#OnAppStart) hook, but they may be
registered at any later time as well.-->

Вот например:
<!--Here are some examples:-->

{% raw %}
<pre class="prettyprint lang-go">
import (
    "github.com/revel/revel"
    "github.com/revel/revel/modules/jobs/app/jobs"
    "time"
)

type ReminderEmails struct {
    // ничего не нужно
}

func (e ReminderEmails) Run() {
    // Запрос к БД
    // Отправка e-mail
}

func init() {
    revel.OnAppStart(func() {
        jobs.Schedule("0 0 0 * * ?",  ReminderEmails{})
        jobs.Schedule("@midnight",    ReminderEmails{})
        jobs.Schedule("@every 24h",   ReminderEmails{})
        jobs.Every(24 * time.Hour,    ReminderEmails{})
    })
}
</pre>
{% endraw %}

## Именованные расписания

Вы можете настраивать расписания в `app.conf` и ссылаться на него где угодно.
Полезно при многократном использовании раписания, а так же позволяет дать ему
легко читаемое имя.
<!--
You can configure schedules in your `app.conf` file and reference them anywhere.
This can help provide easy reuse and a useful description for crontab specs.-->

Для создания именовонного `cron` расписания, поместите эту строчку в `app.conf`
<!--
To define your named cron schedule, put this in your `app.conf`:-->

    cron.workhours_15m = 0 */15 9-17 ? * MON-FRI

Используйте имя вместо расписания:
<!--
Use the named schedule by referencing it anywhere you would have used a
cron spec.-->

{% raw %}
<pre class="prettyprint lang-go">
func init() {
    revel.OnAppStart(func() {
        jobs.Schedule("cron.workhours_15m", ReminderEmails{})
    })
}
</pre>
{% endraw %}

> Важно:  Имя расписания должно начинаться с "cron."
<!--> Note: Your cron schedule name must begin with "cron."-->

## Одноразовые службы

Порой необходимо запустить службы в ответ на действие пользователя. Модуль
служб позволяет запускать те или иные службы один раз, когда Вам угодно.
<!--
Sometimes you will want to do something in response to a user action.  In these
cases, the jobs module allows you to submit a job to be run a single time.-->

Единственное что можно контролировать -- это время до запуска службы
(таймаут перед запуском).
<!--The only control offered is how long to wait until the job should be run.-->

{% raw %}
<pre class="prettyprint lang-go">
type AppController struct { *revel.Controller }

func (c AppController) Action() revel.Result {
    // Обработка запроса.
    ...

    // Отправить ему e-mail с подтверждением прямо сейчас (в фоновом режиме).
    jobs.Now(SendConfirmationEmail{})

    // Или сделать это спустя минуту.
    jobs.In(time.Minute, SendConfirmationEmail{})
}
</pre>
{% endraw %}

## Регистрация функций

Можно зарегистрировать функцию `func()` как службу, обернув её в `jobs.Func`
<!--
It is possible to register a `func()` as a job by wrapping it in the `jobs.Func`
type.  For example:-->

{% raw %}
<pre class="prettyprint lang-go">
func sendReminderEmails() {
    // Запрос к БД
    // Отправка e-mail
}

func init() {
    revel.OnAppStart(func() {
        jobs.Schedule("@midnight", jobs.Func(sendReminderEmails))
    })
}
</pre>
{% endraw %}

## Состояние службы

Модуль служб предоставляет страницу просмотра состояния. На ней можно
взглянуть на текущее состояние службы (**IDLE** или **RUNNING** -- соответственно ожидает и запущена),
и время предыдущего и следующего запуска.
<!--
The jobs module provides a status page that shows a list of the scheduled jobs
it knows about, their current status (**IDLE** or **RUNNING**), and their
previous and next run times.-->

![Просмотр состояния служб](../img/jobs-status.png)

В целях безопасности на страницу состояния можно взглянуть только с IP 127.0.0.1
<!--
For security purposes, the status page is restricted to requests that originate
from 127.0.0.1.-->

## Ограничение размера пула

Есть возможность ограничить количество заданий запускаемых в одно и то же время.
Это позволяет разработчику правильно распределять ресурсы, которые могут быть
потенциально использованы фоновыми службами. Как правило отзывчивость
приложения цениться выше скорости работы фоновых служб. Когда пул заполнен,
новая служба не будет запущена - пока не освободиться место (пока не завершиться
какя либо из запущенных).
<!--It is possible to configure the job module to limit the number of jobs that are
allowed to run at the same time.  This allows the developer to restrict the
resources that could be potentially in use by asynchronous jobs -- typically
interactive responsiveness if valued above asynchronous processing.  When a pool
is full of running jobs, new jobs block to wait for running jobs to complete.-->

Implementation note: The implementation blocks on a channel receive, which is
implemented to be FIFO for waiting goroutines (but not specified/required to be
so). [See here for discussion](https://groups.google.com/forum/?fromgroups=#!topic/golang-nuts/CPwv8WlqKag).

## Для разработчиков

* Доступ к мониторингу служб с помощью HTTP Basic Authentication.
* Позволить администратору управлять расписанием служб со страницы мониторинга.
* Показывать больше информации о запущенных службах, такой как размер пула, длина очереди и т.д.
