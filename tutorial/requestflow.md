---
title: Поток выполнения
layout: tutorial
---

В [предыдущей статье](createapp.html) мы создали новое Revel приложение
названное **myapp**. В этой статье мы посмотрим как Revel обрабатывает HTTP запросы
к http://localhost:9000/.

### Маршрутизация

Первым делом Revel проверяет файл **conf/routes**  -- тут находятся маршруты:

	GET     /                                       App.Index

Это запись говорит выполнить метод **Index** контроллера **App**
когда **GET** запрос придет на  **/**.

### Действия

Проследим за вызовом до файла контроллера **app/controllers/app.go**:

	package controllers

	import "github.com/revel/revel"

	type App struct {
		*revel.Controller
	}

	func (c App) Index() revel.Result {
		return c.Render()
	}

Все контроллеры должны включать объявление `*revel.Controller`
в первой строке (прямо или косвенно). Каждый экспортируемый метод в контроллере,
возвращающий `revel.Result` может рассматриваться в качестве действия.

Контроллеры поддерживают много удобных методов для генерации Result.
В данном примере вызывается [`Render()`](../docs/godoc/mvc.html#Controller.Render),
который находит и передает в шаблон ответ (**200 OK**).

### Шаблоны

Все шаблоны находятся в директории **app/views**. Когда имя шаблона не указано явно,
Revel ищет шаблон по названию действия.
В данном случае, Revel ищет файл вида **app/views/App/Index.html** и представляет его 
как [Go шаблон](http://www.golang.org/pkg/html/template).


{% raw %}

	{{set . "title" "Home"}}
	{{template "header.html" .}}

	<header class="hero-unit" style="background-color:#A9F16C">
	  <div class="container">
	    <div class="row">
	      <div class="hero-text">
	        <h1>It works!</h1>
	        <p></p>
	      </div>
	    </div>
	  </div>
	</header>

	<div class="container">
	  <div class="row">
	    <div class="span6">
	      {{template "flash.html" .}}
	    </div>
	  </div>
	</div>

	{{template "footer.html" .}}

{% endraw %}

Помимо функций, предоставляемых шаблонами GO, Revel добавляет 
[несколько полезностей](../manual/templates.html) из свои собственных.

Данный шаблон очень прост.  Он:

1. Добавляет новое значение **title** для подставление в контекст.
2. Подключает шаблон **header.html** (который использует title).
3. Показывает приветственное сообщение.
4. Подключает шаблон **flash.html** , который показыает некоторые "flashed" сообщения.
5. Подключает **footer.html**.

Если вы посмотрите на header.html, вы сможете увидеть больше шаблонных тегов в действии:

{% raw %}

	<!DOCTYPE html>

	<html>
	  <head>
	    <title>{{.title}}</title>
	    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
	    <link rel="stylesheet" type="text/css" href="/public/css/bootstrap.css">
	    <link rel="shortcut icon" type="image/png" href="/public/img/favicon.png">
	    <script src="/public/js/jquery-1.9.1.min.js" type="text/javascript" charset="utf-8"></script>
	    {{range .moreStyles}}
	      <link rel="stylesheet" type="text/css" href="/public/{{.}}">
	    {{end}}
	    {{range .moreScripts}}
	      <script src="/public/{{.}}" type="text/javascript" charset="utf-8"></script>
	    {{end}}
	  </head>
	  <body>

{% endraw %}

Вы можете увидеть, как задачется title и подключаются JS и CSS файлы, в том числе и из переменных 
**moreStyles** и **moreScripts**.

### Горячяя перезагрузка

Давайте изменим приветственное сообщение.  В **Index.html** изменим

	<h1>It works!</h1>

на

	<h1>Hello World</h1>

Обновим страничку и увидем изменения! Revel заметил, что шаблон изменился и перезагрузил его.

Revel следит за изменениями:

* Любого кода в  **app/**
* Любого шаблона в  **app/views/**
* Любого маршрута в **conf/routes**

Изменение любого из них заставит Revel обновить и запустить приложение с новейшим кодом.

Попробуйте прямо сейчас: откройте **app/controllers/app.go** и допустите ошибку.

Измените

	return c.Render()

на

	return c.Renderx()

Обновите страницу и Revel укажет ошибку:

![A helpful error message](../img/helpfulerror.png)

В заключении, добавим некоторую информацию в шаблон.

В **app/controllers/app.go** поменяйте:

	return c.Renderx()

на:

	greeting := "Aloha World"
	return c.Render(greeting)

В **app/views/App/Index.html** поменяйте:

{% raw %}

	<h1>Hello World</h1>

{% endraw %}

на:

{% raw %}

	<h1>{{.greeting}}</h1>

{% endraw %}

Обновите страницу и увидете гавайское приветствие.

![A Hawaiian greeting](../img/AlohaWorld.png)

**Далее: [Создаем Hello World приложение](firstapp.html).**
