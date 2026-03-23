# OTUS-Angie-2025-10-06
# Домашняя работа OTUS-Angie-2025-10-06 OTUS_Angie_06_Обратный_прокси_(reverse_proxy)

#### 1. Развернуть на вашей учебной виртуальной машине копию CMS Wordpress или аналогичное веб-приложение с использованием стандартного дистрибутива или Docker-контейнера.

```
root@ubuntu01:/home/user/docker-wordpress# docker ps -a
CONTAINER ID   IMAGE                                   COMMAND                  CREATED       STATUS       PORTS                                 NAMES
55f4bb784b90   docker.angie.software/angie:templated   "/init"                  2 hours ago   Up 2 hours   0.0.0.0:80->80/tcp, [::]:80->80/tcp   webserver
3142c539d03d   wordpress:6.0.1-php8.0-fpm-alpine       "docker-entrypoint.s…"   2 hours ago   Up 2 hours   9000/tcp                              wordpress
2a1344ed621c   mysql:8.0                               "docker-entrypoint.s…"   2 hours ago   Up 2 hours   3306/tcp, 33060/tcp                   db
```

#### 2. Настроить Angie в качестве обратного прокси для бэкенда.
 
```
root@ubuntu01:/home/user/docker-wordpress# docker exec -it 55f4bb784b90 angie -v
Angie version: Angie/1.11.3
root@ubuntu01:/home/user/docker-wordpress# docker exec -it 55f4bb784b90 nginx -v
OCI runtime exec failed: exec failed: unable to start container process: exec: "nginx": executable file not found in $PATH: unknown
```

#### 3. Разделить обработку статических и динамических запросов.

Запросы, приходящие на Angie (web-tier):
```
#--- docker logs -f docker logs webserver
#
192.168.31.138 - - [23/Mar/2026:09:04:22 +0000] "GET /wp-admin HTTP/1.1" 301 169 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36" "-"
192.168.31.138 - - [23/Mar/2026:09:04:22 +0000] "GET /wp-admin/ HTTP/1.1" 302 5 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36" "-"
192.168.31.138 - - [23/Mar/2026:09:04:22 +0000] "GET /wp-login.php?redirect_to=http%3A%2F%2F192.168.31.122%2Fwp-admin%2F&reauth=1 HTTP/1.1" 200 10394 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36" "-"
192.168.31.138 - - [23/Mar/2026:09:04:22 +0000] "GET /wp-admin/load-styles.php?c=1&dir=ltr&load%5Bchunk_0%5D=dashicons,buttons,forms,l10n,login&ver=6.9.4 HTTP/1.1" 200 103193 "http://192.168.31.122/wp-login.php?redirect_to=http%3A%2F%2F192.168.31.122%2Fwp-admin%2F&reauth=1" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36" "-"
192.168.31.138 - - [23/Mar/2026:09:04:22 +0000] "GET /wp-includes/js/dist/i18n.min.js?ver=c26c3dc7bed366793375 HTTP/1.1" 200 5314 "http://192.168.31.122/wp-login.php?redirect_to=http%3A%2F%2F192.168.31.122%2Fwp-admin%2F&reauth=1" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36" "-"
192.168.31.138 - - [23/Mar/2026:09:04:22 +0000] "GET /wp-admin/js/password-strength-meter.min.js?ver=6.9.4 HTTP/1.1" 200 1123 "http://192.168.31.122/wp-login.php?redirect_to=http%3A%2F%2F192.168.31.122%2Fwp-admin%2F&reauth=1" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36" "-"
192.168.31.138 - - [23/Mar/2026:09:04:22 +0000] "GET /wp-includes/js/wp-util.min.js?ver=6.9.4 HTTP/1.1" 200 1431 "http://192.168.31.122/wp-login.php?redirect_to=http%3A%2F%2F192.168.31.122%2Fwp-admin%2F&reauth=1" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36" "-"
192.168.31.138 - - [23/Mar/2026:09:04:22 +0000] "GET /wp-includes/js/dist/dom-ready.min.js?ver=f77871ff7694fffea381 HTTP/1.1" 200 457 "http://192.168.31.122/wp-login.php?redirect_to=http%3A%2F%2F192.168.31.122%2Fwp-admin%2F&reauth=1" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36" "-"
192.168.31.138 - - [23/Mar/2026:09:04:22 +0000] "GET /wp-includes/js/dist/a11y.min.js?ver=cb460b4676c94bd228ed HTTP/1.1" 200 2212 "http://192.168.31.122/wp-login.php?redirect_to=http%3A%2F%2F192.168.31.122%2Fwp-admin%2F&reauth=1" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36" "-"
192.168.31.138 - - [23/Mar/2026:09:04:22 +0000] "GET /wp-admin/js/user-profile.min.js?ver=6.9.4 HTTP/1.1" 200 7997 "http://192.168.31.122/wp-login.php?redirect_to=http%3A%2F%2F192.168.31.122%2Fwp-admin%2F&reauth=1" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36" "-"
192.168.31.138 - - [23/Mar/2026:09:04:22 +0000] "GET /wp-admin/load-scripts.php?c=1&load%5Bchunk_0%5D=clipboard,jquery-core,jquery-migrate,zxcvbn-async,wp-hooks&ver=6.9.4 HTTP/1.1" 200 116296 "http://192.168.31.122/wp-login.php?redirect_to=http%3A%2F%2F192.168.31.122%2Fwp-admin%2F&reauth=1" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36" "-"
192.168.31.138 - - [23/Mar/2026:09:04:22 +0000] "GET /wp-includes/js/underscore.min.js?ver=1.13.7 HTTP/1.1" 200 18905 "http://192.168.31.122/wp-login.php?redirect_to=http%3A%2F%2F192.168.31.122%2Fwp-admin%2F&reauth=1" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36" "-"
192.168.31.138 - - [23/Mar/2026:09:04:22 +0000] "GET /wp-admin/images/wordpress-logo.svg?ver=20131107 HTTP/1.1" 200 1521 "http://192.168.31.122/wp-admin/load-styles.php?c=1&dir=ltr&load%5Bchunk_0%5D=dashicons,buttons,forms,l10n,login&ver=6.9.4" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36" "-"
192.168.31.138 - - [23/Mar/2026:09:04:22 +0000] "GET /wp-includes/js/zxcvbn.min.js HTTP/1.1" 200 822237 "http://192.168.31.122/wp-login.php?redirect_to=http%3A%2F%2F192.168.31.122%2Fwp-admin%2F&reauth=1" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36" "-"
```

Запросы, приходящие на WordPress (app-tier):
```
#--- docker logs -f wordpress
#
172.18.0.4 -  23/Mar/2026:09:04:22 +0000 "GET /wp-admin/index.php" 302
172.18.0.4 -  23/Mar/2026:09:04:22 +0000 "GET /wp-login.php" 200
172.18.0.4 -  23/Mar/2026:09:04:22 +0000 "GET /wp-admin/load-styles.php" 200
172.18.0.4 -  23/Mar/2026:09:04:22 +0000 "GET /wp-admin/load-scripts.php" 200
```
