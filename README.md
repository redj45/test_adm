# Тестовое задание

## Выполненное задание должно быть доступно в публичном git репозитории и воспроизводиться в окружении соответствующем требованиям (требования, при наличии, описать в README.md):

### Ожидаемое содержимое репозитория:
- Vagrantfile || .kitchen.yml
- playbook/role || cookbooks etc… разработанные в ходе решения.
- README.md с текстом как что запустить, чтобы получить итоговую конфигурацию с вышеописанными требованиями без необходимости что-либо вносить в attributes/group_vars/host_vars etc…

#### Реализовать архитектурное решение (см. схема, под */** реализовывать опционально, по возможности):
##### Автоматизировать развертывание:
1. 2 nginx балансировщика с принудительным редиректом на https с самоподписанным сертификатом
1. Consul cluster (3 server, 2 agent), c веб-интерфейсом доступным через балансировщики nginx, с авторизацией по логину и паролю
1. С установленным redis рядом с каждым агентом
1. Авторегистрация сервиса redis в consul cluster по заданным условиям *
1. Smoke тестирование **

#### Требования:
1. ОС: ubuntu:16.04 || centos 7.2
1. Описание окружения: vagrant || kitchen (docker,vagrant);
1. Средства автоматизации: ansible:2.6 || chef-solo:14.8
1. Два балансировщика nginx (lb[0,1].local):
    - version: latest
    - port 80 принудительный редирект на 443
    - server_name: test-consul-webui.local
    - selfsign ssl
    - port 443
    - upstream consul_cluster ведет на consul-cluster port web-ui
    - auth basic:
    - login: admin
    - pass: 1235813
    - keepalived:
    - lb0:
       - master: 10.0.0.1/24
       - backup: 10.0.0.2/24
    - lb1:
       - master: 10.0.0.2/24
       - backup: 10.0.0.1/24
    - iptables: пропускает 22,80,443
    - iptables: пропускает всё что надо для keepalived
    - iptables: всё остальное drop
1. Три consul ноды в режиме server, объеденные в кластер (consul-server[0,1,2].local):
    - version: latest
    - name dc: test-dc
    - iptables: пропускает lb на порт web-ui
    - iptables: пропускает consul-client на порт consul-cluster
    - iptables: пропускает ssh
    - iptables: всё остальное drop
1. Две consul ноды в режиме agent (consul-agent[0,1].local):
    - version: latest
    - join to cluster
    - redis: latest
    - регистрацию сервиса redis (установленный локально на каждой ноде consul-agent) *
    - Авторегистрацию сервиса redis в consul-cluster (DCS) со следующими условиями: *
    - redis0 основной redis в service-discovery, даже если redis1 выключен *
    - redis1 основной redis в service-discovery, если redis0 вышел из строя *
    - redis0 становится основым, если redis0 готов к бою, несмотря на то, что redis1 основной. *
    - redis0 и redis1 не в кластере *
    - iptables: пропускает на порт для взаимодействия consul-agent и consul-cluster, redis, ssh
    - iptables: всё остальное drop
1. Написать smoke тесты на inSpec: 3.2.6 **

![Схема](https://github.com/redj45/test_adm/blob/master/test_test_linux_adm%20(1).jpg)
