# Создать свой RPM пакет
Для данного задания нам понадобятся следующие установленные пакеты:
```
yum install -y \redhat-lsb-core \wget \rpmdevtools \rpm-build \createrepo \yum-utils
```
Для примера возьмем пакет NGINX и соберем его с поддержкой openssl
Загрузим SRPM пакет NGINX для дальнейшей работы над ним:
```
wget https://nginx.org/packages/centos/7/SRPMS/nginx-1.14.1-1.el7_4.ngx.src.rpm
```
При установке такого пакета в домашней директории создается древо каталогов для сборки: 
```
rpm -i nginx-1.14.1-1.el7_4.ngx.src.rpm
```
Также нужно скачать и разархивировать последний исходники для openssl - он потребуется при сборке (latest не работало, скачал конкретную последнюю версию):
```
https://www.openssl.org/source/openssl-3.0.2.tar.gz
tar -xvf openssl-3.0.2.tar.gz
```
Заранее поставим все зависимости чтобы в процессе сборки не было ошибок:
```
yum-builddep rpmbuild/SPECS/nginx.spec
```
Далее нужно поправить spec файл чтоы NGINX собирался с необходимыми нам опциями:
```
checking for C compiler ... not found

./configure: error: C compiler cc is not found

error: Bad exit status from /var/tmp/rpm-tmp.k7pbeH (%build)


RPM build errors:
    Bad exit status from /var/tmp/rpm-tmp.k7pbeH (%build)
```
И собрать rpm покет:
```
rpmbuild -bb rpmbuild/SPECS/nginx.spec
```
Ошибка при сборке:
checking for C compiler ... not found
./configure: error: C compiler cc is not found
error: Bad exit status from /var/tmp/rpm-tmp.k7pbeH (%build)
RPM build errors:
    Bad exit status from /var/tmp/rpm-tmp.k7pbeH (%build)
    
Нужно доставить компилятор C/GCC:
```
yum install gcc
```
Еще раз пробуем собрать и убеждаемся что пакеты создались:
```
[root@yarkozloff ~]# rpmbuild -bb rpmbuild/SPECS/nginx.spec
[root@yarkozloff ~]# ll rpmbuild/RPMS/x86_64/
total 3072
-rw-r--r-- 1 root root  769976 Mar 23 00:46 nginx-1.14.1-1.el7_4.ngx.x86_64.rpm
-rw-r--r-- 1 root root 2375128 Mar 23 00:46 nginx-debuginfo-1.14.1-1.el7_4.ngx.x86_64.rpm
```
Теперь можно установить наш пакет и убедиться что nginx работает:
```
yum localinstall -y \rpmbuild/RPMS/x86_64/nginx-1.14.1-1.el7_4.ngx.x86_64.rpm

[root@yarkozloff ~]# systemctl start nginx
[root@yarkozloff ~]# systemctl status nginx
● nginx.service - nginx - high performance web server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2022-03-23 00:51:22 MSK; 5s ago
```

# Создать свой репозиторий и разместить там ранее собранный RPM
Директория для статики у NGINX по умолчанию /usr/share/nginx/html. Создадим там каталог repo:
```
mkdir /usr/share/nginx/html/repo
```
Копируем туда наш собранный RPM
```
cp rpmbuild/RPMS/x86_64/nginx-1.14.1-1.el7_4.ngx.x86_64.rpm /usr/share/nginx/html/repo/
```
Инициализируем репозиторий:
```
createrepo /usr/share/nginx/html/repo/
```
Для прозрачности настроим в NGINX доступ к листингу каталога:
● В location / в файле /etc/nginx/conf.d/default.conf добавим директиву autoindex on. В результате location будет выглядеть так:
location / {
root /usr/share/nginx/html;
index index.html index.htm;
autoindex on; Добавили эту директиву
}
Проверяем синтаксис и перезапускаем HGINX:
```
[root@yarkozloff ~]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@yarkozloff ~]# nginx -s reload
```
Curl-ануть для проверки:
```
[root@yarkozloff ~]# curl -a http://localhost/repo/
<html>
<head><title>Index of /repo/</title></head>
<body bgcolor="white">
<h1>Index of /repo/</h1><hr><pre><a href="../">../</a>
<a href="repodata/">repodata/</a>                                          22-Mar-2022 22:04                   -
<a href="nginx-1.14.1-1.el7_4.ngx.x86_64.rpm">nginx-1.14.1-1.el7_4.ngx.x86_64.rpm</a>                22-Mar-2022 21:55              769976
</pre><hr></body>
</html>
```
Чтобы протестировать репозиторий добавим его в /etc/yum.repos.d:
```
[root@yarkozloff ~]# cat >> /etc/yum.repos.d/otus.repo << EOF
> [otus]
> name=otus-linux
> baseurl=http://localhost/repo
> gpgcheck=0
> enabled=1
> EOF

[root@yarkozloff ~]# cat /etc/yum.repos.d/otus.repo
[otus]
name=otus-linux
baseurl=http://localhost/repo
gpgcheck=0
enabled=1
```
Убедимся что репозиторий подключился и посмотрим что в нем есть:
````

```
