---
title: Создаем новое Revel приложение.
layout: tutorial
---

Используя Revel command line tool создадим пустой проект в GOPATH и запустим его:

	$ cd $GOPATH

	$ revel new myapp
	~
	~ revel! http://revel.github.io
	~
    Your application is ready:
        /Users/revel/code/gocode/src/myapp

    You can run it with:
        revel run myapp

	$ revel run myapp
	~
	~ revel! http://revel.github.io
	~
	2012/09/27 17:01:54 run.go:41: Running myapp (myapp) in dev mode
	2012/09/27 17:01:54 harness.go:112: Listening on :9000

С помощью браузера открываем http://localhost:9000/ и смотрим сообщение что наше приложение работает.

![Your Application Is Ready](../img/YourApplicationIsReady.png)

Созданная структура приложения описывается
[сдесь](../manual/organization.html).

**Далее: [Изучем обработку запросов.](requestflow.html)**
