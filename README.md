## Домашнее задание Cтенд c OSPF

### Описание домашнего задания
```
- развернуть 3 VMs согласно схеме
- настроить OSPF между VMs на базе FRR;
- изобразить ассиметричный роутинг
- сделать один из линков "дорогим", так чтобы при этом роутинг был симметричным
- VMs разворачивается в среде VMware vSphere 7 из шаблона Ubuntu Server 22.04
  
Файлы:

- network.yml:              ansible playbook развертывания VMs,
                            настройки OSPF на базе FRR на VMs 
- vars.yml:                 переменные для создания и наcтройки VMs
- host:                     inventory
- templates\frr.conf.j2:    шаблон для /etc/frr/frr.conf

в скриншотах:
- screens\ansible1.jpg:     протокол работы ansible-playbook network.yml --tags deploy
- screens\ansible1.jpg:     протокол работы ansible-playbook network.yml --tags install

- screens\Схема_сети.jpg:   схема сети

- screens\доступность_сетей_с_router1.jpg: порверка с router1 работоспособности OSPF
                            ping 192.168.30.1
                            traceroute 192.168.30.1
						 
                            отключение интерфейса ens192 на router1 и
                            проверка доступности сети:
                            ifconfig ens192 down						 
                            traceroute 192.168.30.1
                            после отключения интерфейса сеть 192.168.30.0/24 доступна						 
                            смотрим маршруты на данный момент:
                            vtysh
                            show ip route OSPF

Настройка и проверка ассиметричного роутинга
						 
- screens\router1_as_rout.jpg:
                            выключить блокировку ассиметричной маршрутизации:
                            sysctl net.ipv4.conf.all.rp_filter=0
                            на router1 изменим «стоимость интерфейса»:
                            vtysh
                            conf t
                            int ens224
                            ip ospf cost 1000
                            exit
                            exit
                            show ip route ospf
- screens\router2_as_rout1.jpg:  на router2:
                            выключить блокировку ассиметричной маршрутизации:
                            sysctl net.ipv4.conf.all.rp_filter=0
                            vtysh
                            show ip route ospf
                            exit
 
после внесения данных настроек, мы видим, что маршрут до сети 192.168.20.0/30
через router2, но обратный трафик от router2 пойдёт по другому пути:
- screens\router1_as_rout2.jpg:	
                            router1# ping -I 192.168.10.1 192.168.20.1						

- screens\router2_as_rout2.jpg: на router2: 
                            запускаем tcpdump сначала на интерфейсе ens192, затем ens224 
                            tcpdump -i ens192
                            видим, что данный интерфейс получает ICMP-трафик с адреса 192.168.10.1
                            tcpdump -i ens224
                            видим, что данный интерфейс отправляет ICMP-трафик на адрес 192.168.10.1

Настройка и проверка симетричного роутинга

  у нас есть один «дорогой» интерфейс,
  надо добавить ещё один «дорогой» интерфейс,
  чтобы перестала работать ассиметричная маршрутизация
- screens\router2_sim_rout1.jpg:  на router2: 
                            поменяем стоимость интерфейса ens224 
                            vtysh
                            conf t
                            int ens224
                            ip ospf cost 1000
                            exit
                            exit
                            show ip route ospf
                            exit
                            tcpdump -i ens192 
  на router1 запускаем пинг: ping -I 192.168.10.1 192.168.20.1
						
- screens\router2_sim_rout2.jpg: на router2  
                            смотрим трафик только на ens192						 
                            трафик между роутерами ходит симметрично
