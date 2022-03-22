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
Ну и собственно поправить сам spec файл чтоы NGINX собирался с необходимыми нам опциями:
```

```
