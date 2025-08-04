# Домашнее задание к занятию «ELK»

## Михеев Алексей Александрович

### Инструкция по выполнению домашнего задания

1. Сделайте fork [репозитория c шаблоном решения](https://github.com/netology-code/sys-pattern-homework) к себе в Github и переименуйте его по названию или номеру занятия, например, https://github.com/имя-вашего-репозитория/gitlab-hw или https://github.com/имя-вашего-репозитория/8-03-hw).
2. Выполните клонирование этого репозитория к себе на ПК с помощью команды `git clone`.
3. Выполните домашнее задание и заполните у себя локально этот файл README.md:
   - впишите вверху название занятия и ваши фамилию и имя;
   - в каждом задании добавьте решение в требуемом виде: текст/код/скриншоты/ссылка;
   - для корректного добавления скриншотов воспользуйтесь инструкцией [«Как вставить скриншот в шаблон с решением»](https://github.com/netology-code/sys-pattern-homework/blob/main/screen-instruction.md);
   - при оформлении используйте возможности языка разметки md. Коротко об этом можно посмотреть в [инструкции по MarkDown](https://github.com/netology-code/sys-pattern-homework/blob/main/md-instruction.md).
4. После завершения работы над домашним заданием сделайте коммит (`git commit -m "comment"`) и отправьте его на Github (`git push origin`).
5. Для проверки домашнего задания преподавателем в личном кабинете прикрепите и отправьте ссылку на решение в виде md-файла в вашем Github.
6. Любые вопросы задавайте в чате учебной группы и/или в разделе «Вопросы по заданию» в личном кабинете.

Желаем успехов в выполнении домашнего задания.

---

## Дополнительные ресурсы

При выполнении задания используйте дополнительные ресурсы:
- [docker-compose elasticsearch + kibana](11-03/docker-compose.yaml);
- [поднимаем elk в docker](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/docker.html);
- [поднимаем elk в docker с filebeat и docker-логами](https://www.sarulabs.com/post/5/2019-08-12/sending-docker-logs-to-elasticsearch-and-kibana-with-filebeat.html);
- [конфигурируем logstash](https://www.elastic.co/guide/en/logstash/7.17/configuration.html);
- [плагины filter для logstash](https://www.elastic.co/guide/en/logstash/current/filter-plugins.html);
- [конфигурируем filebeat](https://www.elastic.co/guide/en/beats/libbeat/5.3/config-file-format.html);
- [привязываем индексы из elastic в kibana](https://www.elastic.co/guide/en/kibana/7.17/index-patterns.html);
- [как просматривать логи в kibana](https://www.elastic.co/guide/en/kibana/current/discover.html);
- [решение ошибки increase vm.max_map_count elasticsearch](https://stackoverflow.com/questions/42889241/how-to-increase-vm-max-map-count).

**Примечание**: если у вас недоступны официальные образы, можете найти альтернативные варианты в DockerHub, например, [такой](https://hub.docker.com/layers/bitnami/elasticsearch/7.17.13/images/sha256-8084adf6fa1cf24368337d7f62292081db721f4f05dcb01561a7c7e66806cc41?context=explore).

### Задание 1. Elasticsearch 

Установите и запустите Elasticsearch, после чего поменяйте параметр cluster_name на случайный. 

*Приведите скриншот команды 'curl -X GET 'localhost:9200/_cluster/health?pretty', сделанной на сервере с установленным Elasticsearch. Где будет виден нестандартный cluster_name*.

---

![1]()

![2]()

![3]()


### После установки Elasticsearch откроем файл настроек и изменим параметр cluster.name

/etc/elasticsearch/elasticsearch.yml

### После внесения изменений в файл настроек, необходимо перезапустить сервис Elasticsearch, чтобы применить изменения:

sudo systemctl restart elasticsearch

sudo systemctl status elasticsearch

### Теперь Elasticsearch запущен с новым именем кластера. Проверяем состояние кластера с помощью следующей команды:

curl -X GET 'localhost:9200/cluster/health?pretty'

### Также для корректрой работы и вывода информации о кластере были внесены изменения в файл "/etc/elasticsearch/elasticsearch.yml":

xpack.security.enable: false

---


### Задание 2. Kibana

Установите и запустите Kibana.

*Приведите скриншот интерфейса Kibana на странице http://<ip вашего сервера>:5601/app/dev_tools#/console, где будет выполнен запрос GET /_cluster/health?pretty*.

---

![5]()

![6]()

![8]()

![7]()

### Открываем файл "/etc/kibana/kibana.yml" для внесения изменений в конфигурацию:

sudo nano /etc/kibana/kibana.yml

   server.host: "0.0.0.0"

   elasticsearch.hosts: ["http://localhost:9200"]

### Проверяем статус работы Kibana:

sudo systemctl start kibana

sudo systemctl enable kibana

sudo systemctl status kibana

### Открываем браузер и переходим на страницу Kibana:

http://localhost:5601

### Используем Dev Tools в Kibana:

http://localhost:5601/app/dev_tools#/console

### Выполним запрос GET /_cluster/health?pretty в консоли Dev Tools:

GET /_cluster/health?pretty


---

## Задание 3. Logstash

Установите и запустите Logstash и Nginx.  
С помощью Logstash отправьте access-лог Nginx в Elasticsearch.  
Приведите скриншот интерфейса Kibana, на котором видны логи Nginx.

## Решение 3. Logstash

Установка nginx.  
```
sudo apt install nginx
```
Установка logstash.  
```
sudo apt install ./logstash-8.12.2-amd64.deb 
```
Создать файл конфигурации по адресу /etc/logstash/conf.d/logstash.conf.

Далее настроить поставку access-лога nginx в elasticsearch:  
```
# Читаем файл
input {
  file {
    path => ["/var/log/nginx/access.log"]
    start_position => "beginning"
  }
}
# Извлекаем данные из событий
filter {
  grok {
    match => { "message" => "\[%{TIMESTAMP_ISO8601:timestamp}\]\[%{DATA:severity}%{SPACE}\]\[%{DATA:source}%{SPACE}\]%{SPACE}%{GREEDYDATA:message}" }
    overwrite => [ "message" ]
  }
}
# Сохраняем все в Elasticsearch
output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "nginx-logs-%{+YYYY.MM}"
  }
}
```
Запуск службы logstash.  
```
sudo systemctl daemon-reload  
sudo systemctl enable logstash.service   
sudo systemctl start logstash.service  
sudo systemctl status logstash.service
```
Создать данные для просмотра в kibana.  
Маршрут: Management > stack management	> kibana > data view > Create data view!

![]()

![]()

![]()

![]()

![]()

## Задание 4. Filebeat.

Установите и запустите Filebeat.  
Переключите поставку логов Nginx с Logstash на Filebeat.  
Приведите скриншот интерфейса Kibana, на котором видны логи Nginx, которые были отправлены через Filebeat.

##  Решение 4. Filebeat.

### Установка и настройка filebeat.  
```
sudo apt install ./filebeat-8.12.2-amd64.deb  
```
Разрешаем обработку логов Nginx:  
```
mv /etc/filebeat/modules.d/nginx.yml.disabled /etc/filebeat/modules.d/nginx.yml
```
Приводим содержимое файла /etc/filebeat/modules.d/nginx.yml к виду:
```
- module: nginx
  # Access logs
  access:
    enabled: true

    # Set custom paths for the log files. If left empty,
    # Filebeat will choose the paths depending on your OS.
    var.paths: ["/var/log/nginx/access.log"]

  # Error logs
  error:
    enabled: true

    # Set custom paths for the log files. If left empty,
    # Filebeat will choose the paths depending on your OS.
    var.paths: ["/var/log/nginx/error.log"]

  # Ingress-nginx controller logs. This is disabled by default. It could be used in Kubernetes environments to parse ingress-nginx logs
  ingress_controller:
    enabled: false
```
Файл /etc/filebeat/filebeat.yml приводим к виду:
[filebeat.yml]()

Убеждаемся, что в конфигурационном файле нет ошибок:
```  
filebeat test config -c /etc/filebeat/filebeat.yml
```
Получаем вывод 
```
Config OK
```
Запускаем службу filebeat
```
sudo systemctl enable filebeat
sudo systemctl start filebeat
```
### Установка и настройка Logstash

Для удобства отладки предлагаю разделить конфигурационный файл logstash на три части.  
Создаём файл /etc/logstash/conf.d/input-beats.conf.
```
input {
  beats {
    port => 5044
  }
}
```
Создаём файл /etc/logstash/conf.d/output-elasticsearch.conf.
```
output {
  elasticsearch {
    hosts => ["localhost:9200"]
#    manage-template => false
    index => "nginx-filebeat-%{+YYYY.MM.dd}"
  }
  stdout { codec => rubydebug }
}
```
Создаём файл /etc/logstash/conf.d/filter-nginx.conf.
```
filter {
  if [event][dataset] == "nginx.access" {
    grok {
      match => [ "message" , "%{IPORHOST:clientip} %{USER:ident} %{USER:auth} \[%{HTTPDATE:timestamp}\] \"(?:%{WORD:verb} %{NOTSPACE:request}(?: HTTP/%{NUMBER:httpversion})?|%{DATA:rawrequest})\" %{NUMBER:response} (?:%{NUMBER:bytes}|-) %{QS:referrer} %{QS:user_agent}"]
      overwrite => [ "message" ]
    }
    mutate {
      convert => ["response", "integer"]
      convert => ["bytes", "integer"]
      convert => ["responsetime", "float"]
    }
    date {
      match => [ "timestamp" , "dd/MMM/YYYY:HH:mm:ss Z" ]
      remove_field => [ "timestamp" ]
    }
  }
}
```
Запускаем службу logstash.  
```
sudo systemctl enable logstash
sudo systemctl start logstash
```
Проверяем, что logstash слушает порт 5044.
```
sudo netstat -tulpn | grep 5044
```

Смотрим, что получает kibana.

![]()

![]()

![]()

![]()

![]()

### Для отладки удобно использовать следующие журналы.

1. Журнал Filebeat на сервере с nginx:
```
sudo journalctl -ru filebeat.service
```
2. Журнал Logstash на сервере Logstash:
```
sudo journalctl -ru logstash.service
```
Также в конфигурационный файл Logstash можно добавить вывод в консоль:
```
stdout { codec => rubydebug }
```
[Пример вывода в журнал]()

3. Журналы Elasticearch на узлах:
```
sudo journalctl -ru elasticsearch.service
```
