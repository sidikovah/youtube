# Видео раз. Основы helm.

## Что такое helm?

Helm - это менеджер пакетов для kuberntes.

Как любой менеджер пакетов, Helm упрощает задачу управления жизненным циклом приложений.

Апдейт и удаление приложений достаточно простые (ну почти всегда). Поэтому в
данном цикле видео, мы не будем акцентировать внимание на этих действиях. В основном мы 
займемся разбором создания собственных пакетов (chart), их установкой и кастомизацией.

### Документация

* [Helm](https://helm.sh/)
* [Go templates](https://pkg.go.dev/text/template)

### Как работает Helm

Пакет (helm chart или просто chart), обычно распространяется в виде стандартного архива в формате tar.gz
Внутри которого находятся:
* Описание чарта.
* Шаблоны манифестов.
* Конфигурационные параметры приложения по умолчанию. 
* Другие, не обязательные файлы.

Для хранения набора таких пакетов можно использовать любой WEB сервер с обязательным файлом index.yaml, в котором
описываются чарты, которые предоставляются данным сервером. Но это конечно самый простой способ создания репозитория. 
В принципе хранить архивы пакетов можно в специализированных системах или универсальных приложениях, которые 
поддерживают helm charts типа Nexus, Harbor и т.п.

Так же чарт можно хранить например в локальной файловой системе, не запаковывая его в архив, в виде структуры файлов 
и директорий. Но в этом случае затруднена версионность чарта. Т.е. для разных версий чарта необходимо создавать 
отдельные директории. Ситуацию может облегчить хранения файлов чарта в системе контроля версии, например в git.

Основная задача helm:
* Получить от пользователя информацию, какие конфигурационные параметры приложения необходимо переопределить.
Обычно для этого используется кастомный файл values.
* Учитывая параметры, сгенерировать из шаблонов файлы манифестов приложений.
* Итоговые файлы манифестов поместить в kubernetes через kubernetes API.

Разумеется кроме работы с пакетами (шаблонами) helm умеет много чего полезного и по ходу изложения материала
мы познакомимся с этими функциями.

Helm версии 3 не требует наличия в кластере kubernetes дополнительного программного обеспечения.

Helm не может управлять приложениями, установленными помимо него.

## Установка

    wget https://get.helm.sh/helm-v3.7.2-linux-amd64.tar.gz
    tar -zxvf helm-v3.7.2-linux-amd64.tar.gz
    mv linux-amd64/helm /usr/local/bin/helm
    helm version
    helm list
    rm -r helm-v3.7.2-linux-amd64.tar.gz linux-amd64

## Задача

Изучать что-либо, просто так, без поставленной задачи бессмысленно. Поэтому сформулируем задачу, которую мы должны
будем решить.

У нас есть некоторое приложение - openresty, которое мы запускаем в кластере kubernetes. Для этого приложения мы написали 
файлы [манифестов](../base-application).

Наша задача, сделать из этих манифестов чарт, так что бы мы могли при установке изменять парамеры деплоя, 
конфигурационных файлов, сервисов и т.п.

У нас не стоит задачи сделать супер chart с возможностью кастомизации всего и вся. Только базовый функционал,
на котором можно понять как работает "кухня" helm charts.

