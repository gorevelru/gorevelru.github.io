---
title: Приступаем к работе
layout: tutorial
---

Эта статья рассказывает о процессе установки.

#### Устанавливаем Go

Для использования Revel, первым делом нужно [установить Go](http://golang.org/doc/install).

#### Переменная окружения GOPATH

Если вы не установили GOPATH на этапе установки, сделайте это сейчас. GOPATH
это директория, где будет находится весь ваш Go код.  Для установки продейлате следующие операции:

1. Создайте директорию: `mkdir ~/gocode`
2. Установите переменную окружения GOPATH: `export GOPATH=~/gocode`
3. Сохраните GOPATH для автоматической установки при загрузке: `echo GOPATH=$GOPATH >> .bash_profile`

Поздравляем! Вы установили Go.

#### Установка git и mercurial
И Git и Mercurial требуются для работы `go get`.

* [Установка Git](http://git-scm.com/book/en/Getting-Started-Installing-Git)
* [Установка Mercurial](http://mercurial.selenic.com/wiki/Download)

#### Получшение Revel
Для получения Revel, запустите команду:

	go get github.com/revel/revel

Произойдет следующие:

* Go, используя git, клонирует репозиторий в `$GOPATH/src/github.com/revel/revel/`
* Go найдет все зависимости и установит их через `go get`.


#### Сборка Revel command line tool

Revel command line tool умеет собирать, запускать и упаковывать приложения.

Используйте `go get` для установки:

	go get github.com/revel/cmd/revel

Затем проверьте, что папка $GOPATH/bin находится в PATH, что бы вы могли запускать Revel глобально.

	export PATH="$PATH:$GOPATH/bin"
	echo 'PATH="$PATH:$GOPATH/bin"' >> .bash_profile

На последок, проверяем, что все работает:

	$ revel help
	~
	~ revel! http://revel.github.io
	~
	usage: revel command [arguments]

	The commands are:

	    run         run a Revel application
	    new         create a skeleton Revel application
	    clean       clean a Revel application's temp files
	    package     package a Revel application (e.g. for deployment)

	Use "revel help [command]" for more information.

Теперь у нас есть установленный и работоспособный Revel.

**Далее: [Создаем новое приложение.](createapp.html)**
