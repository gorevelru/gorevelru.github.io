---
title: Приложение Hello World на Revel
layout: tutorial
---

Эта статья покажет как быстро создать приложение "Hello World" из 
[the Play! example](http://www.playframework.org/documentation/1.2.4/firstapp).

Начнем в проекте **myapp**, который [мы создали](createapp.html) на прошлом уроке.

Отредактируем шаблон **app/views/App/Index.html** и добавим форму после подключения шаблона `flash.html`:

	<form action="/App/Hello" method="GET">
	    <input type="text" name="myName" /><br/>
	    <input type="submit" value="Say hello!" />
	</form>

Обновим страницу и посмострим на изменение.

![The Say Hello form](../img/AlohaForm.png)

Давайте попробуем отправить данные из формы.

![Route not found](../img/HelloRouteNotFound.png)

Видим осмысленную ошибку.  Добавим дейсвтие в  **app/controllers/app.go**:

	func (c App) Hello(myName string) revel.Result {
		return c.Render(myName)
	}


Теперь мы должны создать представление.  Создадим файл
**app/views/App/Hello.html**, со следующим контентом::

{% raw %}
	{{set . "title" "Home"}}
	{{template "header.html" .}}

	<h1>Hello {{.myName}}</h1>
	<a href="/">Back to form</a>

	{{template "footer.html" .}}
{% endraw %}

После обновления страницы мы должны увидеть приветствие:

![Hello revel](../img/HelloRevel.png)

В заключении, добавляем проверку.  Имя не должно быть пустым, и должно содержать не менее 3 символов.

Для этого, используем [модуль проверки](../manual/validation.html).  Отредактируем
наше действие из **app/controllers/app.go**:

	func (c App) Hello(myName string) revel.Result {
		c.Validation.Required(myName).Message("Your name is required!")
		c.Validation.MinSize(myName, 3).Message("Your name is not long enough!")

		if c.Validation.HasErrors() {
			c.Validation.Keep()
			c.FlashParams()
			return c.Redirect(App.Index)
		}

		return c.Render(myName)
	}

Теперь мы покажем пользователю `Index()` если поле было заполнено неправильно. 
Данные и ошибка будут сохранены во временных cookie
[Flash](../manual/sessionflash.html).

Поключенный шаблон `flash.html` покажет ошибки или flash сообщения:

{% raw %}
	{{if .flash.success}}
	<div class="alert alert-success">
		{{.flash.success}}
	</div>
	{{end}}

	{{if or .errors .flash.error}}
	<div class="alert alert-error">
		{{if .flash.error}}
			{{.flash.error}}
		{{end}}
		{{if .errors}}
		<ul style="margin-top:10px;">
			{{range .errors}}
				<li>{{.}}</li>
			{{end}}
		</ul>
		{{end}}
	</div>
	{{end}}
{% endraw %}

Когда мы оправляем форму с именем, которое не проходит валидацию, мы хотим, что бы в форму было вставлено ошибочное имя и пользователь мол отредактировать его перед отправлением. 
Изменим форму в **app/views/App/Index.html**:

{% raw %}
	<form action="/App/Hello" method="GET">
		{{with $field := field "myName" .}}
			<input type="text" name="{{$field.Name}}" value="{{$field.Flash}}"/><br/>
		{{end}}
		<input type="submit" value="Say hello!" />
	</form>
{% endraw %}

Теперь, когда мы отправляем букву вместо имени:

![Example error](../img/HelloNameNotLongEnough.png)

Отлично! Мы получили ошибку и сохранили предыдущий ввод для редактирования.
