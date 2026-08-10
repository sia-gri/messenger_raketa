<h1 style="text-align: center;"><strong>Тестовое задание </strong><a href="https://www.raketa.world/"><strong>RAKETA</strong></a></h1>
<p style="text-align: right;"><strong>Аналитик: Анастасия Гришкова</strong></p>
<p style="text-align: center;"><strong>Разработка текстового мессенджера</strong></p>
<p style="text-align: justify;"><em><span style="font-weight: 400;">*При выполнении задания учитываю, что мессенджер разрабатывается, как отдельный веб сайт в десктопной версии без интеграции и без настроек вида отображения. Бизнес логика мессенджера описана только для диалога 1-1, без возможности создавать другие чаты, всего 2 окна авторизации и самого диалога. Нормы ГОСТ 34 в текущей задаче не применяю.</span></em></p>
<h2><strong><strong>1. Задача для разработчика</strong></strong></h2>
<p style="text-align: justify;"><strong>Название: </strong><span style="font-weight: 400;">00001.</span> <span style="font-weight: 400;">MVP. EPIC. Разработка мессенджера &laquo;Raketa 1-1&raquo;.</span></p>
<p style="text-align: justify;"><strong>Цель:</strong> <span style="font-weight: 400;">разработать текстовый мессенджер для двух абонентов с функционалом регистрации/авторизации, обмена текстовыми сообщениями, хранения истории переписки, отображения статуса собеседника и статусов доставки сообщений, с использованием команды из 4 разработчиков до 5 октября 2026 года (8 недель).</span></p>
<p style="text-align: justify;"><strong>Окружение: </strong><span style="font-weight: 400;">Desktop, Linux (Ubuntu)</span></p>
<p style="text-align: justify;"><strong>Функциональные требования:</strong></p>
<ul style="text-align: justify;">
<li style="font-weight: 400;"><span style="font-weight: 400;">Система должна обеспечивать создание учетных записей с уникальным логином и надежным паролем, а также аутентификацию при входе. Пароли хранятся в хешированном виде с использованием bcrypt. При успешной авторизации пользователь получает JWT-токен, статус меняется на &laquo;В сети&raquo;, фиксируется время последней активности. При выходе статус становится &laquo;Не в сети&raquo;.</span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">Система должна обеспечивать передачу текстовых сообщений от одного пользователя другому с подтверждением доставки. Сообщение не должно быть пустым и не должно превышать 4096 символов. При отправке сообщение сохраняется в БД со статусом &laquo;отправлено&raquo;, при доставке статус меняется на &laquo;доставлено&raquo;, при прочтении на &laquo;прочитано&raquo;.&nbsp;</span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">Система должна предоставлять доступ к истории переписки с собеседником, загружая по 50 сообщений. Реализован бесконечный скролл для загрузки более ранних сообщений.</span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">Система должна информировать о присутствии собеседника в сети и о состоянии отправленных сообщений. Статус собеседника обновляется в реальном времени через WebSocket.</span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">Система должна подсчитывать и отображать количество сообщений, полученных пользователем, но еще не прочитанных. При открытии чата все непрочитанные сообщения автоматически отмечаются как прочитанные, счетчик обнуляется.</span></li>
</ul>
<p style="text-align: justify;"><strong>Нефункциональные требования:</strong></p>
<ul>
<li style="font-weight: 400; text-align: justify;"><span style="font-weight: 400;">Среднее время отклика API не должно превышать 150 мс. Загрузка 50 сообщений выполняется не более чем за 300 мс. Доставка сообщения от отправителя до получателя не должна превышать 200 мс.</span></li>
<li style="font-weight: 400; text-align: justify;"><span style="font-weight: 400;">Пароли передаются и хранятся только в виде хэша. Все соединения работают по HTTPS и WebSocket Secure. Аутентификация через JWT (access token 1 час, refresh 7 дней).&nbsp;</span></li>
<li style="font-weight: 400; text-align: justify;"><span style="font-weight: 400;">Интерфейс интуитивно понятен, не требует обучения. Все элементы на русском языке.&nbsp;</span></li>
</ul>
<h3 style="text-align: center;"><strong>Общая логика</strong></h3>
<p style="text-align: justify;"><strong>Авторизация: </strong><span style="font-weight: 400;">Пользователь вводит уникальный логин и пароль. Сервер проверяет учетные данные и возвращает токен сессии (или статус ошибки).</span></p>
<p style="text-align: justify;"><strong>Активность: </strong><span style="font-weight: 400;">Сервер хранит состояние &laquo;В сети / Не в сети&raquo; для каждого пользователя. При выходе из приложения состояние меняется на &laquo;Не в сети&raquo;.</span></p>
<p style="text-align: justify;"><strong>Отправка сообщения:</strong></p>
<ul style="text-align: justify;">
<li style="font-weight: 400;"><span style="font-weight: 400;">Клиент А отправляет сообщение на сервер с указанием recipient_id (получатель Б).</span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">Сервер сохраняет сообщение в БД status</span> <span style="font-weight: 400;">load/send/delivered.</span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">Сервер проверяет, есть ли у получателя (Б) активное WebSocket/HTTPS-соединение.</span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">Если Б в сети, сервер мгновенно пушит сообщение клиенту Б.</span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">Если Б не в сети, сообщение сохраняется в БД до момента его следующего подключения.</span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">После в БД уходит status=read.</span></li>
</ul>
<h3 style="text-align: center;"><strong>Интерфейс основного окна</strong></h3>
<p style="text-align: justify;"><span style="font-weight: 400;">Окно разделено на 3 зоны. Дизайн максимально простой (опираться на Telegram).</span></p>
<ol style="text-align: justify;">
<li style="font-weight: 400;"><span style="font-weight: 400;">Верхняя панель (Header):</span></li>
</ol>
<ul style="text-align: justify;">
<li style="font-weight: 400;"><span style="font-weight: 400;">Имя собеседника (крупно).</span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">Статус собеседника (Online/Offline) серого/зеленого цвета.</span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">Кнопка &laquo;Выход&raquo;.</span></li>
</ul>
<ol style="text-align: justify;">
<li style="font-weight: 400;"><span style="font-weight: 400;">Центральная часть:</span></li>
</ol>
<ul style="text-align: justify;">
<li style="font-weight: 400;"><span style="font-weight: 400;">Список сообщений, подгружаемый сверху вниз (скролл).</span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">Блоки сообщений выровнены по правому краю (Мои) и левому краю (Собеседника).</span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">Для сообщений собеседника отображается время отправки.</span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">Рядом с моими сообщениями отображается статус доставки/прочтения (лоадер = оправляется, одна галочка = доставлено, две галочки = прочитано).&nbsp;</span></li>
</ul>
<ol style="text-align: justify;">
<li style="font-weight: 400;"><span style="font-weight: 400;">Нижняя часть:</span></li>
</ol>
<ul style="text-align: justify;">
<li style="font-weight: 400;"><span style="font-weight: 400;">Текстовое поле ввода (многострочное, поддерживающее Enter).</span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">Кнопка &laquo;Отправить&raquo;.</span></li>
</ul>
<h3 style="text-align: center;"><strong>Data Flow</strong></h3>
<p style="text-align: justify;"><span style="font-weight: 400;"><img src="https://github.com/sia-gri/messenger_raketa/blob/d0fa178b8f2bd85c3f3965b3eb9c55addd604880/UML%20backend.jpg" alt="" width="450" height="700" /></span></p>
<p style="text-align: justify;"><span style="font-weight: 400;">Рис.1. UML диаграмма потока данных мессенджера (без авторизации)</span></p>
<p style="text-align: justify;"><strong>Критерии приемки:</strong></p>
<ul style="text-align: justify;">
<li style="font-weight: 400;"><span style="font-weight: 400;">Все функциональные требования реализованы полностью.</span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">Проведены успешно все тестовые сценарии.</span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">Время отклика на действия пользователя не превышает 500 мс.</span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">Отсутствуют критические ошибки.</span></li>
</ul>
<h2><strong><strong><br />2. База данных&nbsp;</strong></strong></h2>
<p style="text-align: justify;"><span style="font-weight: 400;">База данных разработана в СУБД MySQL. Типы данных также подобраны с учетом того, что это мини-версия мессенджера, не подразумевающая большой нагрузки на сервер. При таких вводных получаем две таблицы. См. рис.2.</span></p>
<p style="text-align: justify;"><strong>USERS</strong></p>
<ul style="text-align: justify;">
<li style="font-weight: 400; text-align: justify;"><span style="font-weight: 400;">user_id &ndash; уникальный идентификатор пользователя, Primary Key, Foreign Key для sender_id и recipient_id в таблице messages.&nbsp;</span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">name &ndash; имя пользователя, которое указывается в диалоге, не нужно для авторизации.</span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">login &ndash; уникальный логин пользователя с символьным ограничением 50.</span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">password_hash &ndash; пароль, храним только хэш.</span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">last_seen &ndash; время последнего входа в приложение.</span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">online_now &ndash; булевый флажок, необходимый для отображения пользователя, который в данным момент находится в сети, DEFAULT FALSE.</span></li>
</ul>
<p style="text-align: justify;"><span style="font-weight: 400;">*При масштабируемости следует вынести отдельно логины и пароли для защиты и безопасности данных.</span></p>
<p style="text-align: justify;"><strong>MESSAGES</strong></p>
<ul style="text-align: justify;">
<li style="font-weight: 400;"><span style="font-weight: 400;">message_id &ndash; уникальный идентификатор сообщения, Primary Key.</span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">sender_id &ndash; id отправителя сообщения, Foreign Key для user_id в таблице users.</span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">recipient_id &ndash; id получателя сообщения, Foreign Key для user_id в таблице users.</span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">text &ndash; текст сообщения. Не может быть пустым.</span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">send_time &ndash; время отправки сообщения.</span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">read_time &ndash; время прочтения сообщения получателем.&nbsp;</span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">status &ndash; статус отправки и прочтения сообщения. Может быть 4 видов:&nbsp;</span></li>
</ul>
<ol style="text-align: justify;">
<li style="font-weight: 400;"><span style="font-weight: 400;">load &ndash; отправляется.</span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">send &ndash; отправлено.</span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">delivered &ndash; доставлено.</span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">read &ndash; прочитано.</span></li>
</ol>
<p style="text-align: justify;">&nbsp;</p>
<p><strong>*<em>SQL-запрос к спроектированной БД: посчитать текущее количество непрочитанных сообщений пользователей, полученных за последние сутки.</em></strong></p>
<p><span style="font-weight: 400;">SELECT user_id, COUNT(*) AS unread_messages</span></p>
<p><span style="font-weight: 400;">FROM users</span></p>
<p><span style="font-weight: 400;">INNER JOIN messages ON user_id = recipient_id</span></p>
<p><span style="font-weight: 400;">WHERE (status = 'send' OR status = 'delivered' OR status = 'load')</span></p>
<p><span style="font-weight: 400;">AND send_time &gt;= NOW() - INTERVAL 1 DAY</span></p>
<p><span style="font-weight: 400;">GROUP BY user_id;</span></p>
<p><span style="font-weight: 400;"><img src="https://github.com/sia-gri/messenger_raketa/blob/241e70a347e4c37a83384205554384c39f24fd27/ERD%20%D0%9C%D0%B5%D1%81%D1%81%D0%B5%D0%BD%D0%B4%D0%B6%D0%B5%D1%80%20RAKETA%201-1.jpg" alt="" width="530" height="580" /></span></p>
<p><span style="font-weight: 400;">Рис. 2. ER-диаграмма базы данных веб-сайта &laquo;Raketa 1-1&raquo;</span></p>
<h2><strong><strong>3. Сценарий тестирования</strong></strong></h2>
<p style="text-align: justify;"><span style="font-weight: 400;">Описала один сценарий основной функциональности. Для полного тестирования необходимы еще тест-кейсы авторизации пользователя, регистрации и выхода из системы, отправки сообщений офлайн пользователям, просмотр истории и поиск по сообщениям, счетчик непрочитанных сообщений, валидацию текста в поле ввода сообщений, отображение и работу индикатора &laquo;отправки-прочтения&raquo; сообщений, даты и времени отправки, отдельно дополнительно проверить корректность отработки БД, а также негативные и нефункциональные сценарии и проверку на безопасность.</span></p>
<h3 style="text-align: center;"><strong>Тест-кейс &laquo;Отправка сообщения в существующем диалоге&raquo;</strong></h3>
<p style="text-align: justify;"><strong>Предусловие: </strong><span style="font-weight: 400;">Мессенджер открыт на двух устройствах. Два тестовых пользователя зарегистрированы в системе.</span></p>
<p><strong>Креды для авторизации пользователей user1 и user2 (логин, пароль).</strong></p>
<p><strong>Шаги:</strong></p>
<ol>
<li style="font-weight: 400;"><span style="font-weight: 400;">Открыть страницу входа.</span></li>
<li style="font-weight: 400; text-align: justify;"><span style="font-weight: 400;">Авторизоваться на первом устройстве под пользователем user1.</span></li>
<li style="font-weight: 400; text-align: justify;"><span style="font-weight: 400;">Авторизоваться на втором устройстве под пользователем user2.</span></li>
<li style="font-weight: 400; text-align: justify;"><span style="font-weight: 400;">Убедиться, что диалог user1 с user2 открыт на устройстве первого пользователя.</span></li>
<li style="font-weight: 400; text-align: justify;"><span style="font-weight: 400;">Ввести валидное текстовое сообщение (&laquo;Привет! Мне отправили оффер на позицию аналитика в RAKETA! Ура!&raquo;.</span></li>
<li style="font-weight: 400; text-align: justify;"><span style="font-weight: 400;">Отправить сообщение (кликом мыши на &laquo;Отправить&raquo; и через enter).</span></li>
<li style="font-weight: 400; text-align: justify;"><span style="font-weight: 400;">В интерфейсе пользователя user2 проверить, что отправленное сообщение пришло от user1 с допустимой погрешностью 2-3 сек.&nbsp;</span></li>
</ol>
<p style="text-align: justify;"><strong>Ожидаемые результаты:&nbsp;</strong></p>
<ul>
<li style="font-weight: 400; text-align: justify;"><span style="font-weight: 400;">Система отображает диалоговое окно.</span></li>
<li style="font-weight: 400; text-align: justify;"><span style="font-weight: 400;">Система отображает историю переписки с выбранным пользователем.</span></li>
<li style="font-weight: 400; text-align: justify;"><span style="font-weight: 400;">В нижней панели в правой части поле ввода сообщения активно для ввода текста.</span></li>
<li style="font-weight: 400; text-align: justify;"><span style="font-weight: 400;">Отправленное сообщение мгновенно появляется в диалоге, а также отображается в соответствии с прототипом.</span></li>
<li style="font-weight: 400; text-align: justify;"><span style="font-weight: 400;">Под сообщением индикаторы статуса работают в логике &laquo;Отправляется&raquo; - &laquo;Отправлено&raquo; - &laquo;Доставлено&raquo; - &laquo;Прочитано&raquo; в соответствии с прототипом.</span></li>
<li style="font-weight: 400; text-align: justify;"><span style="font-weight: 400;">Поле ввода сообщение очищено, после отправки сообщения.</span></li>
<li style="font-weight: 400; text-align: justify;"><span style="font-weight: 400;">Сообщение корректно сохранилось в БД в таблице messages (в соответствии со спецификацией API).</span></li>
<li style="font-weight: 400; text-align: justify;"><span style="font-weight: 400;">Пользователю user2 пришло сообщение в диалог с пользователем user1. </span></li>
</ul>
