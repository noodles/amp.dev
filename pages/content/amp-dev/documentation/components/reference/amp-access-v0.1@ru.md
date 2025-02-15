---
$category@: dynamic-content
formats:
- websites
teaser:
  text: Платный доступ и настройка подписки на AMP-страницы
---

# amp-access

С помощью этой функции обеспечивается платный доступ и настройка подписки на AMP-страницы. Издатели могут контролировать, какой контент будет доступен читателям, и задавать ограничения в зависимости от статуса подписки, количества просмотров и других факторов.

# amp-access

<!--
Copyright 2015 The AMP HTML Authors. All Rights Reserved.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS-IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->

<table>
  <tr>
    <td><strong>Доступность</strong></td>
    <td>Стабильная версия</td>
  </tr><tr>
  <td class="col-fourty"><strong>Скрипт</strong></td>
  <td>
    <div>
      <code>&lt;script async custom-element="amp-access" src="https://cdn.ampproject.org/v0/amp-access-0.1.js">&lt;/script></code>
    </div>
  </td>
</tr>
<tr>
  <td class="col-fourty"><strong>Примеры</strong></td>
  <td><a href="https://ampbyexample.com/components/amp-access/">Аннотированный пример кода для amp-access</a></td>
</tr>
</table>

## Взаимосвязь с `amp-subscriptions`

Расширение [`amp-subscriptions`](amp-subscriptions.md) по функциональным возможностям похоже на `amp-access`, но поддерживает более специализированный протокол платного доступа. Вот некоторые важные различия:

1. Ответы `amp-subscriptions` на запросы о правах похожи на авторизацию в amp-access, но строго определены и стандартизированы.
1. Расширение `amp-subscriptions` позволяет настроить несколько служб для принятия решений о платном и бесплатном доступе. Они используются параллельно, а приоритет определяется с учетом того, какая служба возвращает положительный ответ.
1. Средства просмотра AMP могут предоставлять `amp-subscriptions` подписанный ответ об авторизации на основании независимого соглашения с издателями, которое является доказательством правомерного доступа.
1. Разметка контента в `amp-subscriptions` стандартизирована, что позволяет приложениям и поисковым роботам с легкостью обнаруживать разделы с премиум-контентом.

Новым издателям и поставщикам рекомендуется использовать `amp-subscriptions`, поскольку там стандартизирована разметка, а также реализована поддержка нескольких поставщиков и улучшенного средства просмотра.

## Решение

Рекомендуемое решение позволяет издателю контролировать следующие процессы:
– Добавление пользователей и управление ими.
– Управление отслеживанием (разрешение на определенное количество бесплатных просмотров).
– Процесс входа в систему.
– Аутентификация пользователей.
– Авторизация и права доступа.
– Гибкая настройка параметров доступа для каждого отдельного документа.

Решение включает следующие компоненты:

1. [**Идентификатор читателя AMP**](#amp-reader-id) – уникальный идентификатор читателя, который присваивается ему экосистемой AMP.
1. [**Разметка контента для доступа**](#access-content-markup) – создается издателем и определяет какие разделы документа пользователи могут просматривать в зависимости от обстоятельств.
1. [**Конечная точка авторизации**](#authorization-endpoint) – предоставляется издателем и возвращает ответ с указанием того, какие разделы документа доступны читателю.
1. [**Конечная точка автоматического уведомления**](#pingback-endpoint) – предоставляется издателем и используется для регистрации просмотра документа.
1. [**Ссылка для входа и страница входа в систему**](#login-page-and-login-link) – позволяют издателю выполнять аутентификацию читателя и устанавливать его связь с идентификатором читателя AMP.

Google AMP Cache возвращает читателю документ, в котором некоторые разделы скрыты с помощью разметки контента для доступа. Библиотека AMP вызывает конечную точку авторизации и в зависимости от ответа скрывает или показывает определенные разделы. Когда читатель просматривает документ, библиотека AMP вызывает конечную точку автоматического уведомления. Издатель может использовать полученные данные для обновления счетчика бесплатных просмотров.

Издатель также может добавить в AMP-документ ссылку, по которой пользователь будет переходить на страницу входа или подписки. Там выполняется аутентификация: определяется идентификатор пользователя в системе, который соответствует идентификатору читателя AMP.

По умолчанию это решение отправляет читателю полный вариант документа, хотя некоторые разделы и скрываются с учетом ответа при авторизации. Однако есть и "серверный" вариант, когда разделы с ограниченным доступом не отправляются читателю, а загружаются только после пройденной авторизации.

Для использования amp-access издатель должен внедрить описанные выше компоненты. Разметка контента для доступа и конечная точка авторизации – обязательные свойства. Конечная точка автоматического уведомления и страница входа – необязательные.

### Идентификатор читателя AMP

*Идентификатор читателя* в системе AMP обеспечивает доступ к сервисам и сбор пользовательской статистики.

Это уникальный анонимный идентификатор, который создается экосистемой AMP. Читатель получает новый идентификатор при работе с сайтом каждого издателя. Его идентификаторы у двух разных издателей не будут совпадать. Отменить присвоение идентификатора нельзя. Он используется во всех коммуникациях с AMP и издателем, а также имеет очень высокую энтропию. Издатели таким образом могут идентифицировать читателя и соотносить его с собственными системами.

Идентификатор читателя создается с учетом пользовательского устройства и предназначен для длительного использования. Тем не менее для него используются обычные правила хранения в браузерах, в том числе и для окон в режиме инкогнито. Предполагаемый жизненный цикл идентификатора составляет один год между использованиями. Если пользователь очистит файлы cookie, идентификатор будет стерт. Его передача между разными устройствами в настоящее время не поддерживается.

Идентификатор читателя создается так же, как [ExternalCID](https://docs.google.com/document/d/1f7z3X2GM_ASb3ZCI_7tngglxwS6WoWi1EB3aKzdf6vo/edit#heading=h.hb9q0wpwwhuf). Пример: `amp-OFsqR4pPKynymPyMmplPNMvxSTsNQob3TnK-oE3nwVT0clORaZ1rkeEz8xej-vV6`.

### Решение amp-access и файлы cookie

Издатели могут использовать для аутентификации свои собственные файлы cookie или идентификаторы читателей. Поддерживается также сочетание обоих вариантов.

### Разметка контента для доступа

Эта разметка определяет, какие разделы документа будут скрыты, а какие нет. При этом учитывается ответ, полученный из конечной точки авторизации. Для разметки используются специальные атрибуты.

### Конечная точка авторизации

Эта проверенная конечная точка CORS GET, которая предоставляется издателем. К ней обращается библиотека AMP или Google AMP Cache. Она возвращает параметры доступа, которые могут использоваться разметкой контента для показа или скрытия определенных разделов документа.

### Конечная точка автоматического уведомления

Эта проверенная конечная точка CORS GET, которая предоставляется издателем. К ней обращается библиотека AMP или Google AMP Cache. Эта проверенная конечная точка CORS POST. отправляет вызов автоматически, когда читатель начинает просмотр документа. Система также обращается к ней после того, как читатель успешно завершил вход. Одна из основных целей использования такой конечной точки – обновление счетчиков.

Это необязательный компонент. Чтобы отключить его, необходимо задать для свойства `noPingback` в конфигурации значение `true`.

### Ссылка для входа и страница входа в систему

Страница входа реализуется и обслуживается издателем. К ней обращается библиотека AMP. Как правило, такая страница отображается в браузере как диалоговое окно.

Запуск страницы происходит, когда читатель нажимает на ссылку для входа в систему. Издатель может разместить ссылку в любой части документа.

## Спецификация: версия 0.1

### Конфигурация

Все конечные точки в AMP-документе настраиваются как объект JSON в разделе HEAD.

```html

<script id="amp-access" type="application/json">
  {
    "property": value,
    ...
    }
</script>

```

Свойства конфигурации перечислены в таблице ниже.

<table>
  <tr>
    <th>Свойство</th>
    <th>Значения</th>
    <th>Описание</th>
  </tr>
  <tr>
    <td class="col-fourty"><code>authorization</code></td>
    <td><code>&lt;URL&gt;</code></td>
    <td>URL HTTPS для конечной точки авторизации.</td>
  </tr>
  <tr>
    <td class="col-fourty"><code>pingback</code></td>
    <td><code>&lt;URL&gt;</code></td>
    <td>URL HTTPS для конечной точки автоматического уведомления.</td>
  </tr>
  <tr>
    <td class="col-fourty"><code>noPingback</code></td>
    <td>true/false</td>
    <td>Если задано значение true, автоматическое уведомление отключается.</td>
  </tr>
  <tr>
    <td class="col-fourty"><code>login</code></td>
    <td class="col-twenty"><code>&lt;URL&gt;</code> или<br><code>&lt;Map[string, URL]&gt;</code></td>
    <td>URL HTTPS для страницы входа или набор таких URL для разных типов страниц.</td>
  </tr>
  <tr>
    <td class="col-fourty"><code>authorizationFallbackResponse</code></td>
    <td><code>&lt;object&gt;</code></td>
    <td>Объект JSON, который используется вместо ответа, если авторизация не проходит успешно.</td>
  </tr>
  <tr>
    <td class="col-fourty"><code>authorizationTimeout</code></td>
    <td><code>&lt;number&gt;</code></td>
    <td>Время ожидания (в миллисекундах), после которого для запроса на авторизацию засчитывается сбой. Значение по умолчанию: 3000. Значения больше 3000 допускаются только в среде разработчика. </td>
  </tr>
  <tr>
    <td class="col-fourty"><code>type</code></td>
    <td>"client" или "server"</td>
    <td>Значение по умолчанию: client. Вариант server находится в разработке. Документация будет обновлена после его запуска.</td>
  </tr>
  <tr>
    <td class="col-fourty"><code>namespace</code></td>
    <td>string</td>
    <td>По умолчанию значение пустое. Пространство имен требуется, если указано несколько поставщиков доступа.</td>
  </tr>
</table>

Значения *`<URL>`* соответствуют URL HTTPS с подстановкой переменных. Более подробная информация приведена в разделе [Переменные URL](#access-url-variables) ниже.

Пример конфигурации amp-access:

```html

<script id="amp-access" type="application/json">
{
  "authorization":
      "https://pub.com/amp-access?rid=READER_ID&url=SOURCE_URL",
  "pingback":
      "https://pub.com/amp-ping?rid=READER_ID&url=SOURCE_URL",
  "login":
      "https://pub.com/amp-login?rid=READER_ID&url=SOURCE_URL",
  "authorizationFallbackResponse": {"error": true}
}
</script>

```

#### Несколько поставщиков доступа

Чтобы указать нескольких поставщиков, используйте массив, а не единственный объект. Для каждой записи добавьте `namespace`.

```html

<script id="amp-access" type="application/json">
[
  {
    "property": value,
    ...
    "namespace": value
  },
  ...
[
</script>
```

### Переменные URL

При настройке URL для разных конечных точек издатель может использовать подстановку переменных. Полный список этих переменных приведен в [спецификации переменных для AMP](https://github.com/ampproject/amphtml/blob/master/spec/amp-var-substitutions.md). Кроме того там описаны некоторые переменные для доступа, например `READER_ID` и `AUTHDATA`. Самые важные варианты описаны в таблице ниже.

<table>
  <tr>
    <th>Переменная</th>
    <th>Описание</th>
  </tr>
  <tr>
    <td class="col-thirty"><code>READER_ID</code></td>
    <td>Идентификатор читателя AMP.</td>
  </tr>
  <tr>
    <td class="col-thirty"><code>AUTHDATA(field)</code></td>
    <td>Значение в поле ответа на запрос об авторизации.</td>
  </tr>
  <tr>
    <td class="col-thirty"><code>RETURN_URL</code></td>
    <td>Метка-заполнитель для возвратного URL, который указан библиотекой AMP для диалогового окна, используемого при входе.</td>
  </tr>
  <tr>
    <td class="col-thirty"><code>SOURCE_URL</code></td>
    <td>Исходный URL AMP-документа. Если документ загружается из СДК, AMPDOC_URL представляет собой URL СДК, а SOURCE_URL – исходную ссылку на источник.</td>
  </tr>
  <tr>
    <td class="col-thirty"><code>AMPDOC_URL</code></td>
    <td>URL AMP-документа.</td>
  </tr>
  <tr>
    <td class="col-thirty"><code>CANONICAL_URL</code></td>
    <td>Канонический URL AMP-документа.</td>
  </tr>
  <tr>
    <td class="col-thirty"><code>DOCUMENT_REFERRER</code></td>
    <td>URL перехода.</td>
  </tr>
  <tr>
    <td class="col-thirty"><code>VIEWER</code></td>
    <td>URL средства просмотра AMP.</td>
  </tr>
  <tr>
    <td class="col-thirty"><code>RANDOM</code></td>
    <td>Случайное число. Помогает избежать кеширования в браузере.</td>
  </tr>
</table>

Пример URL, куда добавлены идентификатор читателя, данные об источнике ссылки и случайное число для блокировки кеша:
```text
https://pub.com/access?
  rid=READER_ID
  &url=CANONICAL_URL
  &ref=DOCUMENT_REFERRER
  &_=RANDOM
```

Переменная AUTHDATA доступна для URL как конечной точки, так и входа. С ее помощью можно передать любое поле из запроса авторизации как параметр URL (например, `AUTHDATA(isSubscriber)`). Разрешены также вложенные выражения, такие как `AUTHDATA(other.isSubscriber)`. Если вы используете пространства имен, в поле можно добавить и их, например так: `AUTHDATA(anamespace.afield)`.

### Разметка контента для доступа

Эта разметка указывает, какие разделы должны быть видимыми, а какие – скрытыми. Она состоит из двух атрибутов AMP, которые можно добавить в любой элемент HTML: `amp-access` и `amp-access-hide`.

Атрибут `amp-access` содержит выражение, которое возвращает значение true или false в зависимости от ответа на запрос, отправленный в конечную точку авторизации. Полученное значение указывает, будет ли виден элемент и его контент.

Значение `amp-access` – это логическое выражение, определенное на языке, который похож на SQL. Подробнее о его грамматике читайте в [Приложении A](#appendix-a-amp-access-expression-grammar). Пример:
```html

<div amp-access="expression">...</div>
```
Свойства и значения определяются с учетом ответа на запрос авторизации, отправленный в конечную точку. Это обеспечивает гибкость системы для поддержки различных сценариев доступа. Если вы используете пространства имен, добавьте их к именам свойств, например anamespace.aproperty.

Атрибут amp-access-hide можно применить, чтобы скрыть элемент до тех пор, пока не будет получен положительный ответ на запрос об авторизации. Этот атрибут обеспечивает невидимость по умолчанию. Ответ на запрос может его переопределить, и тогда раздел становится видимым. Если атрибут amp-access-hide отсутствует, раздел по умолчанию доступен для просмотра. Этот атрибут можно использовать только в сочетании с amp-access.
```html
<div amp-access="expression" amp-access-hide>...</div>
```

Если запрос авторизации не подтверждается, выражения amp-access не оцениваются. Видимость раздела в таком случае зависит от наличия атрибута amp-access-hide, изначально предоставленного в документе.

Мы можем расширить набор атрибутов amp-access-*, если нам понадобятся новые возможности обфускации и обработки.*

Если запрос авторизации не подтверждается, а authorizationFallbackResponse не задан в документации, выражения amp-access не оцениваются. Видимость раздела в таком случае зависит от наличия атрибута amp-access-hide, изначально предоставленного в документе.

Вот пример, где представлена ссылка для входа или весь контент в зависимости от статуса подписки:
```html
<header>
  Заголовок документа.
</header>
<div>
  Первый фрагмент документа.
</div>

<div amp-access="NOT subscriber" amp-access-hide>
  <a on="tap:amp-access.login">Подпишитесь прямо сейчас!</a>
</div>

<div amp-access="subscriber">
  Остальной контент.
</div>

```
Расшифровка:
– *subscriber* – это поле с логическим значением в ответе на запрос авторизации, отправленном конечной точкой. По умолчанию этот раздел скрыт, но вы можете изменить настройки.
– В этом примере контент показывается полностью, если иное не указано.

Вот ещё один пример, где читателю показывается информация о состоянии счетчика:
```html
{% raw %}
<section amp-access="views <= maxViews">
  <template amp-access-template type="amp-mustache">
    You are reading article {{views}} out of {{maxViews}}.
  </template>
</section>
{% endraw %}
```

А вот пример, где обладатели премиум-подписки видят дополнительный контент:
```html
<section amp-access="subscriptonType = 'premium'">
  Shhh… No one but you can read this content.
</section>
```

### Конечная точка авторизации

Для настройки авторизации используется свойство `authorization` описанное в разделе [Конфигурация amp-access](#configuration). Это проверенная конечная точка CORS GET. Подробнее о том, как обеспечить безопасность запроса, читайте в разделе [Безопасность источника CORS](#cors-origin-security).

При авторизации принимаются любые параметры, описанные в разделе [Переменные URL](#access-url-variables). Например, могут быть переданы идентификатор читателя AMP и URL документа. Помимо параметров URL издатель может использовать любые данные, которые передаются по протоколу HTTP, например IP-адрес читателя. Необходимо обязательно добавить `READER_ID`.

Конечная точка создает ответ на запрос авторизации. Он может применяться в выражениях разметки контента, чтобы показывать или скрывать определенные разделы.

Формат запроса:
```text
https://publisher.com/amp-access.json?
rid=READER_ID
&url=SOURCE_URL
```
Это объект JSON произвольной формы, который может содержать любые свойства и значения с небольшими ограничениями:
– Названия свойств должны соответствовать требованиям грамматики для `amp-access` (см. [Приложение A](#appendix-a-amp-access-expression-grammar)). Самое главное, чтобы имена свойств не содержали пробелы, тире и другие символы, не соответствующие спецификации amp-access.
– Значения параметров могут быть только трех типов: строка, число или логическое значение.
– Значения могут быть вложены в объекты со значениями тех же трех типов.
– Общий объем сериализованного ответа на запрос не может превышать 500 байт.
– Убедитесь, что в ответе отсутствует информация, позволяющая идентифицировать личность, и какие-либо персональные данные.

Вот несколько вариантов информации, которую можно получать из конечной точки авторизации:
– Данные счетчиков: ограничение по просмотрам и текущее количество просмотров.
– Статус пользователя: вход в систему или подписка.
– Более подробные сведения о подписке: базовая или премиум.
– Географические данные: страна, регион пользователя, регион публикации.

В этом примере читатель не является подписчиком. Ему доступно 10 статей в месяц и он уже просмотрел 6 из них:
```json
{
  "maxViews": 10,
  "currentViews": 6,
  "subscriber": false
}
```
В этом примере читатель вошел в систему и является премиум-подписчиком:
```json
{
  "loggedIn": true,
  "subscriptionType": "premium"
}
```
Этот RPC может быть выполнен на этапе до обработки, поэтому его нельзя использовать для изменения данных счетчика. Дело в том, что читатель пока ещё не увидел документ.

Ещё один важный момент: иногда библиотеке AMP требуется несколько раз отправить запрос в конечную точку для одного и того же показа документа. Это может быть связано с тем, что читательские параметры доступа значительно изменились, например, после успешного входа в систему.

Библиотека AMP использует запросы и расширения в трех случаях:

1. При оценке выражений `amp-access`.
2. При оценке шаблонов `<template>`, таких как `amp-mustache`.
3. При добавлении дополнительных переменных в URL конечной точки и URL для входа (с помощью `AUTHDATA(field)`).

Библиотека AMP вызывает конечную точку авторизации как проверенную конечную точку CORS. При этом используется протокол CORS. Необходим источник CORS, а также исходный источник. О том, как ограничить доступ к сервису, читайте в разделе [Безопасность источника CORS](#cors-origin-security). Эта конечная точка может использовать издательские файлы cookie. Например, таким образом устанавливается связь между идентификатором читателя и собственными данными издателя об этом пользователе. Технологии AMP эта информация не требуется, поэтому она и не запрашивается. Подробнее читайте в разделах [Идентификатор читателя AMP](#amp-reader-id) и [Решение amp-access и файлы cookie](#amp-access-and-cookies).

Библиотека AMP (или, скорее, браузер) просматривает кешированные заголовки ответа, когда обращается к конечной точке авторизации. Таким образом, кешированные ответы могут быть повторно использованы. Если же издатель не хочет их использовать, он может применить соответствующие заголовки для управления кешем и/или подстановку переменной `RANDOM` для URL конечной точки.

В случае неудачного запроса библиотека AMP Runtime возвращается к параметру authorizationFallbackResponse, если он указан в конфигурации. В этом случае ход авторизации продолжается в обычном режиме, а вместо ответа используется значение свойства authorizationFallbackResponse. ``Если это свойство не задано, запрос считается ошибочным, а выражения amp-access не оцениваются. Видимость раздела в таком случае зависит от наличия атрибута `amp-access-hide`, изначально предоставленного в документе.

Через 3 секунды для запроса авторизации автоматически регистрируется превышение времени ожидания.

В процессе авторизации библиотека AMP использует следующие классы CSS:

1. Класс `amp-access-loading` добавляется в корень документа, когда авторизация начинается, и удаляется при завершении или сбое.
2. Класс `amp-access-error` добавляется в корень документа при сбое в процессе авторизации.

В варианте *server* вызов конечной точки авторизации выполняется Google AMP Cache. При этом она воспринимается как обычная конечная точка HTTPS, а файлы cookies издателя не используются.

### Конечная точка автоматического уведомления

Автоматическое уведомление настраивается с помощью свойства `pingback`, описанного в разделе [Конфигурация amp-access](#configuration). Эта проверенная конечная точка CORS POST. Подробнее о том, как обеспечить безопасность запроса, читайте в разделе [Безопасность источника CORS](#cors-origin-security).

URL автоматического уведомления указывать не обязательно. Чтобы отключить его, добавьте фрагмент `"noPingback": true`.

URL автоматического уведомления принимает любые параметры, описанные в разделе [Переменные URL](#access-url-variables). Например, могут быть переданы идентификатор читателя AMP и URL документа. Параметр `READER_ID` является обязательным.

Автоматическое уведомление не отправляет ответ. Библиотека AMP игнорирует все ответы.

Система также обращается к конечной точке автоматического уведомления после того, как читатель успешно завершил вход или начал просмотр документа.

Издатель может использовать автоматическое уведомление в следующих целях:
– Для подсчета количества бесплатных просмотров страницы.
– Для сопоставления идентификатора читателя AMP с издательскими данными (поскольку проверенная конечная точка CORS может содержать файлы cookie издателя).

Формат запроса:
```text
https://publisher.com/amp-pingback?
rid=READER_ID
&url=SOURCE_URL
```

### Страница входа

URL страницы входа настраивается с помощью свойства `login`, описанного в разделе [Конфигурация amp-access](#configuration).

Можно задать как один URL, так и целую карту, где каждому URL соответствует определенный тип входа. Пример с одним URL:
```json
{
  "login": "https://publisher.com/amp-login.html?rid={READER_ID}"
  }
```

Пример с несколькими URL:
```json
{
  "login": {
    "signin": "https://publisher.com/signin.html?rid={READER_ID}",
    "signup": "https://publisher.com/signup.html?rid={READER_ID}"
    }
  }
```

URL входа принимает любые параметры, описанные в разделе [Переменные URL](#access-url-variables). Например, могут быть переданы идентификатор читателя AMP и URL документа. С помощью переменной `RETURN_URL` можно задать параметр запроса для возвратного URL, такой как `?ret=RETURN_URL`. Возвратный URL указывать обязательно, а если переменная `RETURN_URL` не задана, он будет автоматически добавлен с параметром запроса по умолчанию (return).

Страница входа – это обычная веб-страница без каких-либо ограничений кроме того, что она должна открываться в [диалоговом окне браузера](https://developer.mozilla.org/en-US/docs/Web/API/Window/open). Ознакомьтесь с разделом [Процесс входа в систему](#login-flow).

Формат запроса:
```text
https://publisher.com/amp-login.html?
rid=READER_ID
&url=SOURCE_URL
&return=RETURN_URL
```
Обратите внимание на то, что параметр return автоматически добавляется библиотекой AMP, если переменная `RETURN_URL` не задана. Когда действия на странице входа завершены, она должна перенаправлять пользователя на указанный возвратный URL:
```text
RETURN_URL#success=true|false
```
Обратите внимание на хеш-параметр success. Значение true или false указывает на то, успешно ли был выполнен вход в систему. Страница входа по возможности отправляет сигнал в любом случае.

Если получен сигнал `success=true`, библиотека AMP повторяет вызовы конечных точек авторизации и автоматического уведомления, чтобы обновить статус документа. При этом в новом профиле доступа регистрируется просмотр.

#### Ссылка входа

Издатель может добавить такую ссылку в любую часть документа.

URL настраивается с помощью свойства login, описанного в разделе [Конфигурация amp-access](#configuration).

Ссылка входа может быть объявлена в любом элементе HTML, который поддерживает атрибут on. Как правило, это точка привязки или кнопка. Формат для одной ссылки входа:
```html
<a on="tap:amp-access.login">Login or subscribe</a>
```

Формат для нескольких ссылок входа: `tap:amp-access.login-{type}`. Пример:
```html
<a on="tap:amp-access.login-signup">Subscribe</a>
</code>.</p>

<p>Формат при использовании пространства имен: <code>tap:amp-access.login-{namespace}</code> или <code>tap:amp-access.login-{namespace}-{type}</code>.</p>

<p>В AMP нет различий между входом в систему и подпиской. Однако соответствующие настройки можно задать на стороне издателя или используя несколько URL входа.</p>

<h2>Интеграция с <em>amp-analytics</em></h2>

<p>Интеграция с <em>amp-analytics</em> описана в файле <a href="./amp-access-analytics.md">amp-access-analytics.md</a>.</p>

<h2>Безопасность источника CORS</h2>

<p>Для авторизации и автоматического уведомления используются конечные источники CORS. Они должны соответствовать протоколу безопасности, описанному в <a href="../../../documentation/guides-and-tutorials/learn/amp-caches-and-cors/amp-cors-requests.md#cors-security-in-amp">этой спецификации</a>.</p>

<h2>Использование счетчиков</h2>

<p>Счетчики позволяют показывать премиум-контент обычным пользователям в течение определенного времени и ограниченное количество раз. Если ограничение достигнуто, читатель видит предложение платной подписки и ссылку для регистрации или входа. Контент в таком случае доступен не полностью. Пример описания счетчика: "Вы можете бесплатно читать 10 статей в месяц".</p>

<p>При настройке ограниченного доступа учитывайте следующее:

1. Для сохранения статистики по счетчикам необходим идентификатор READER_ID. Поскольку издатель не всегда может добавлять файлы cookie на сторонние компьютеры, эти данные необходимо хранить на сервере.
2. "Счетчик чтения" может быть обновлен только в конечной точке автоматического уведомления.
3. Учитываются только уникальные документы: если пользователь 10 раз откроет одну и ту же статью, для него будет засчитан один просмотр. Для этого конечные точки авторизации и автоматического уведомления могут добавлять <code>SOURCE_URL</code> и другие переменные, описанные в разделе <a href="#access-url-variables">Переменные URL</a>.</p>

  <h2>Принцип "первое нажатие бесплатно"</h2>

  <p>Описание этого принципа можно найти <a href="https://support.google.com/news/publisher/answer/40543">в этой статье</a>, а сведения о его актуальных изменениях – <a href="https://googlewebmastercentral.blogspot.com/2015/09/first-click-free-update.html">в нашем блоге</a>.</p>

  <p>Чтобы внедрить принцип "первое нажатие бесплатно", издатель должен иметь возможность определить службу обращений для каждого просмотра, а также подсчитывать ежедневное количество просмотров для каждого читателя.</p>

  <p>Оба этапа описаны в спецификации amp-access. URL перехода можно добавить в URL авторизации и автоматического уведомления с помощью <code>DOCUMENT_REFERRER</code>. Это описано в разделе <a href="#access-url-variables">Переменные+ URL</a>. Подсчет просмотров обеспечивается с помощью конечной точки автоматического уведомления на стороне сервера. Это очень похоже на реализацию <a href="#metering">счетчика</a>.</p>

  <h2>Процесс входа в систему</h2>

  <p>AMP запускает диалоговое окно входа в систему как основное или всплывающее. Также оно может быть открыто в новой вкладке. Средства просмотра AMP по возможности запускают такое окно в контексте браузера, чтобы можно было использовать высокоуровневые +API браузера.</p>

  <p>Процесс входа в систему запускается библиотекой AMP, когда читатель активирует ссылку для входа в систему и происходит следующее:

  1. Диалоговое окно для входа в систему (основное) открывается библиотекой AMP или средством просмотра. Используемый URL содержит дополнительный параметр запроса, который называется возвратным URL (<code>&amp;return=RETURN_URL</code>). В URL также могут присутствовать и другие параметры, например идентификатор читателя. Подробная информация приведена в разделе <a href="#login-page">Страница входа</a>.
  2. Издатель отображает страницу входа в свободной форме.
  3. Читатель выполняет действия, чтобы войти в систему. Например, указывает имя пользователя и пароль или входит, используя аккаунт социальной сети.
  4. Читатель отправляет сведения о входе в систему. Издатель завершает аутентификацию, устанавливает файлы cookie и, наконец, перенаправляет читателя на возвратный URL, ранее указанный в запросе. При перенаправлении указывается хеш-параметр <code>success</code> со значением <code>true</code> или <code>false</code>.
  5. Диалоговое окно поддерживает перенаправление на возвратный URL.
  6. Библиотека AMP повторно авторизует документ.</p>

    <p>От издателя требуется выполнить только действия 2–5: указать страницу входа и убедиться в том, что перенаправление выполняется верно. Страница входа – это обычная веб-страница без каких-либо ограничений кроме того, что она должна открываться в диалоговом окне браузера.</p>

    <p>Как обычно, в вызов страницы входа необходимо включить идентификатор читателя. Издатель может использовать его для сопоставления идентификационной информации. Кроме того, издатель будет получать пользовательские файлы cookie и сможет их настраивать. Если читатель уже вошел в систему на стороне издателя, рекомендуется немедленно перенаправить его на возвратный URL. Для этого отправляется ответ <code>success=true</code>.</p>

    <h2>Глоссарий по AMP</h2>

    <ul>
      <li><strong>AMP-документ</strong> – это HTML-документ в формате AMP, проверенный с помощью AMP-валидатора. Кеширование таких документов выполняется сервисом Google AMP Cache.</li>
      <li><strong>Валидатор AMP</strong> – программа, которая выполняет статический анализ HTML-документа и указывает, соответствует ли документ всем правилам формата AMP.</li>
      <li><strong>Библиотека AMP</strong> – библиотека JavaScript, которая выполняет показ AMP-документа.</li>
      <li><strong>Google AMP Cache</strong> – прокси-кеш для AMP-документов.</li>
      <li><strong>Средство просмотра AMP</strong> –веб-приложение или нативное приложение, которое отображает встроенные или непосредственно опубликованные AMP-документы.</li>
      <li><strong>Publisher.com</strong> – сайт издателя AMP.</li>
      <li><strong>Конечная точка CORS</strong> – конечная точка HTTPS для различных источников. Описание приведено в этой статье: <a href="https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS">https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS</a>. Подробнее о том, как обеспечить безопасность запроса, читайте в разделе <a href="#cors-origin-security">Безопасность источника CORS</a>.</li>
      <li><strong>Читатель</strong> – пользователь, который просматривает AMP-документы.</li>
      <li><strong>Предварительная обработка AMP</strong> – средства просмотра AMP могут использовать предварительную обработку скрытых документов перед показом. Это значительно повышает эффективность. Однако важно учитывать, что предварительная обработка не засчитывается как просмотр, поскольку читатель может не увидеть документ.</li>
    </ul>

    <h2>Версии</h2>

    <ul>
      <li>2 сентября 2016 г. Свойство конфигурации noPingback и настройка автоматического уведомления по желанию.</li>
      <li>3 марта 2016 г. Повторная отправка автоматического уведомления после входа (версия 0.5).</li>
      <li>19 февраля 2016 г. Исправлены примеры: из подставляемых переменных URL удалены символы <code>{}</code>.</li>
      <li>15 февраля 2016 г. <a href="#configuration">Конфигурация</a> и <a href="#authorization-endpoint">конечная точка авторизации</a> теперь поддерживают свойство authorizationFallbackResponse. Его можно использовать при сбое авторизации.</li>
      <li>11 февраля 2016 г. Для <a href="#authorization-endpoint">конечной точки авторизации</a> добавлено время ожидания ответа на запрос.</li>
      <li>11 февраля 2016 г. Разрешены ссылки на вложенные поля, такие как <code>object.field</code>.</li>
      <li>9 февраля 2016 г. Добавлены разделы <a href="#first-click-free">Принцип "первое нажатие бесплатно"</a> и <a href="#metering">Использование счетчиков</a>.</li>
      <li>3 февраля 2016 г. В раздел <a href="#cors-origin-security">Безопасность источника CORS</a> добавлена спецификация для исходного источника.</li>
      <li>1 февраля 2016 г. Параметр запроса return для страницы входа теперь можно настраивать с помощью подстановки RETURN_URL.</li>
    </ul>

    <h2>Приложение А: грамматика выражения "amp-access"</h2>

    <p>Самая новая грамматика BNF доступна в файле <a href="./0.1/access-expr-impl.jison">access-expr-impl.jison</a>.</p>

    <p>Выдержка из грамматики:
```javascript
search_condition:
  search_condition OR search_condition
  | search_condition AND search_condition
  | NOT search_condition
  | '(' search_condition ')'
  | predicate

predicate:
  comparison_predicate | truthy_predicate

comparison_predicate:
  scalar_exp '=' scalar_exp
  | scalar_exp '!=' scalar_exp
  | scalar_exp '<' scalar_exp
  | scalar_exp '<=' scalar_exp
  | scalar_exp '>' scalar_exp
  | scalar_exp '>=' scalar_exp

truthy_predicate: scalar_exp

scalar_exp: literal | field_ref

field_ref: field_ref '.' field_name | field_name

literal: STRING | NUMERIC | TRUE | FALSE | NULL
```

<p>Учтите, что выражения <code>amp-access</code> оцениваются библиотекой AMP или сервисом Google AMP Cache. Это НЕ ВХОДИТ в спецификацию, которую должен реализовать издатель. Информация приведена только для справки.</p>

<h2>Подробные сведения</h2>

<p>В этом разделе будет подробное объявление принципов, по которым создана спецификация amp-access, а также пояснение решений. Скоро!</p>

<h2>Валидация</h2>

<p>Ознакомьтесь с <a href="https://github.com/ampproject/amphtml/blob/master/extensions/amp-access/validator-amp-access.protoascii">правилами для amp-access</a> в спецификации Валидатора AMP.</p>
