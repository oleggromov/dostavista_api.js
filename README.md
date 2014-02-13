# Javascript-клиент для API dostavista.ru

**Задача библиотеки:** облегчить интеграцию интернет-магазинов со службой доставки Dostavista.ru.

JS-клиент предназначен для подключения к админке интернет-магазина, работает поверх XMLHttpRequest и предоставляет простой способ отправлять заказы в службу доставки.

## Правила работы

1. **Скачать последнюю версию библиотеки и подключить** в конец кода админки магазина после всех остальных скриптов и перед закрывающим тегом `</body>`.

	```html
	...
	<script src="js/dostavista_api.min.js"></script>
	</body>
	```

2. **Вставить тег `<span class="DostavistaButton"></span>`** в тех местах, где вы хотите видеть кнопки «в Достависту». По-умолчанию, каждая кнопка будет отвечать за отправку данных одного заказа.

3. **Добавить dsta-аттрибуты, описывающие каждый заказ.** Для каждого тега `.DSTA_button` в разметке на момент инициализации Javascript должны содержаться все необходимые параметры.

	```html
	<span class="DSTA_button"
		...
		dsta-matter="Видеорегистратор"
		dsta-insurance="15000"
		...

		></span>
	```

	**Полный список всех атрибутов**

	| Атрибут | Тип и ограничения | Значение |
	|---------|-------------------|----------|
	| `dsta-matter` | Строка | Что везём, например «диктофон» или «видеорегистратор» |
	| `dsta-insurance` | Целое число от 0 до 15000 | Сумма страховки в рублях |
	| `dsta-address` | Строка | Адрес первой точки маршрута |
	| `dsta-required_time_start` | YYYY-MM-DD HH:MM:SS | Время прибытия на точку (от) * обязательно |
	| `dsta-required_time` | YYYY-MM-DD HH:MM:SS | Время прибытия на точку (до) * обязательно |
	| `dsta-contact_person` | Строка | Имя контактного лица на точке забора |
	| `dsta-phone` | 10 цифр без знака + и международного кода | Телефон контактного лица на точке забора |
	| `dsta-weight` | Целое число | Вес на точке | 
	| `dsta-taking` | Целое число | Стоимость на точке: сумма, которую должен взять курьер на точке |
	| `dsta-client_order_id` | Строка | Номер заказа в магазине, используется для уведомлений на точках |


	> Копейки и десятые доли килограмма отбрасываются, даже если их указать.


4. **Задать общие для всех обращений к API Достависты параметры: `clientId` и `token`** для однозначной идентификации. Метод `setClient` не делает XHR-запросов, а просто сохраняет общие параметры на будущее.

	```html
	<script>
		DostavistaApi.setClient({
			client_id: 1234,
			token: 'xxxx'
		});
	</script>
	```

6. **Определить общую глобальную функцию-callback, которая будет вызываться после отправки данных заказа.**

	```html
	<script>
		DSTA_success = function(status, data) {
			if (!status) {
				alert(data.error_code + "\n" + data.error_message);
				return;
			}
			
			MyShop_save({
				order_id: data.order_id, // ID заказа в системе Dostavista.ru
				delivery_cost: data.payment // приблизительная стоимость доставки.
			});
		};
	</script>
	```

	Если `status === true`, значит заказ попал в Достависту, и можно сохранять результат у себя на сервере. В примере выше это делает функция `MyShop_save()`, но вы можете использовать любую другую.


	**Параметры коллбэка**

	| Название | Тип | Описание |
	|----------|-----|----------|
	| `status` | Boolean | Результат отправки. `false` значит, что произошла ошибка |
	| `data` | Object | Хэш с описанием результата. Отличается, в зависимости от `status` |
	| `data.error_message` | Строка | Сообщение об ошибке |
	| `data.error_code` | Массив | Массив с кодами ошибок |
	| `data.order_id` | Целое число | Внутренний номер заказа в системе Dostavista.ru | 
	| `data.payment` | Целое число | Приблизительная стоимость доставки |



## Описание API
[https://docs.google.com/a/oleggromov.com/document/d/1-yjBzfkI9Zb44kkQB_rMcq5pNeLThyD6YbvXR9wl7IY/](https://docs.google.com/a/oleggromov.com/document/d/1-yjBzfkI9Zb44kkQB_rMcq5pNeLThyD6YbvXR9wl7IY/)

## Вопросы

1. В каком часовом поясе указывается время? Не будет ли ошибок с конвертацией? Можно попробовать перейти на [ISO-8061](http://ru.wikipedia.org/wiki/ISO_8601) с обязательным указанием часового пояса.
2. Как тестировать API? Упомянутые в гуглодоке способы не удалось заставить работать.


## TODO 

1. Отправка множества точек одним вызовом

### Технический TODO
1. Отказаться от jQuery
2. Тесты
3. Запихнуть весь код в [песочницу](https://github.com/a-ignatov-parc/requirejs-sandbox)

### Текущий TODO
1. Grunt.js для создания минифицированной версии
2. Обновить README

3. Колбэк «до» для задания параметров.

4. Разобраться, откуда должна взяться первая точка
5. Парсинг/конвертация телефона и даты в правильный формат

9. Заменить URL на живой
10. Найти ошибку с неправильным типом point и рассказать о ней Игорю

12. Добавить возможность устанавливать несколько колбеков на событие

13. Доработать API и клиент, чтобы можно было менять кнопки «В Достависту» на номер заказа со ссылкой.

14. Нормальная обработка JSONP-ошибок.
15. Переделать логику состояний кнопки