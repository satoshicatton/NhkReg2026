# NhkReg2026 - Мой вариант решения регионального этапа чемпионата "Профессионалы" по СиСА 2026 года. Alt + Ecorouter
Ниже были использованы материалы с sysahelper.ru

Копия - https://www.markdownpaste.com/document/nhkreg2026

## МОДУЛЬ Б:

Важно! В модуле Б, при настройке коммутации на свитчах sw*-cod, sw*-a надо в файл /etc/net/ifaces/mgmt/options добавить OVS_OPTIONS='vlan_mode=native_untagged', иначе после перезагрузки устройства эта настройка сбросится.

Таблица адресации:

<img width="718" height="514" alt="изображение" src="https://github.com/user-attachments/assets/bada74c4-5295-4cd5-9e14-5693972206dc" />

### 1. Настройка доменных имён на RTR-COD и RTR-A:
  **RTR-COD:**
   
  ``` cisco
  ecorouter>enable
ecorouter#configure terminal 
Enter configuration commands, one per line.  End with CNTL/Z.
ecorouter(config)#hostname rtr-cod
rtr-cod(config)#ip domain-name cod.ssa2026.region
rtr-cod(config)#write memory
Building configuration...

rtr-cod(config)#

  ```

Проверяем имя устройства:


<img width="315" height="78" alt="изображение" src="https://github.com/user-attachments/assets/0c920880-2189-4e8d-96fa-6890177acf46" />


Проверить имя доменное:


<img width="485" height="98" alt="изображение" src="https://github.com/user-attachments/assets/921b1485-2c98-4270-b993-103b1d2c9ee7" />


**Назначение IP на устройство:**


  <img width="576" height="111" alt="изображение" src="https://github.com/user-attachments/assets/4a8d0de9-5feb-4626-8015-a23bca227377" />


- Создадим интерфейс с именем isp и назначим на него IP-адрес 178.207.179.4/29

``` cisco
rtr-cod(config)#interface isp
rtr-cod(config-if)#ip address 178.207.179.4/29
rtr-cod(config-if)#exit
rtr-cod(config)#
```

- Создадим интерфейс с именем fw-cod и назначим на него IP-адрес 172.16.1.1/30, также зададим для данного интерфейса описание

``` cisco
rtr-cod(config)#interface fw-cod
rtr-cod(config-if)#ip address 172.16.1.1/30
rtr-cod(config-if)#exit
rtr-cod(config)#
```

- Проверить назначенные IP-адреса на интерфейсы можно командой show ip interface brief
  - созданные интрфейсы пока не добавлены в какие-либо **Service instance**, а значит не привязаны и к порту, отсюда и статус **down**


<img width="672" height="122" alt="изображение" src="https://github.com/user-attachments/assets/7c5cff2d-1930-4a57-b1e9-8405a53739fd" />

- В режиме конфигурирования порта te0 необходимо создать service-instance с произвольным именем, например te0/isp:

``` cisco

rtr-cod(config)#port te0
rtr-cod(config-port)#service-instance te0/isp
rtr-cod(config-service-instance)#encapsulation untagged 
rtr-cod(config-service-instance)#connect ip interface isp 
rtr-cod(config-service-instance)#exit
rtr-cod(config-port)#exit
rtr-cod(config)#

```

- В режиме конфигурирования порта te1 необходимо создать service-instance с произвольным именем, например te1/fw-cod:

``` cisco

rtr-cod(config-port)#service-instance te1/fw-cod
rtr-cod(config-service-instance)#encapsulation untagged 
rtr-cod(config-service-instance)#connect ip interface fw-cod 
rtr-cod(config-service-instance)#exit
rtr-cod(config-port)#exit
rtr-cod(config)# write memory

```

- Проверить IP через show ip interface brief (in enable mode)
  **ISP и FW-COD должны быть UP**


- Проверить Service instances через show-service instance brief

<img width="791" height="202" alt="изображение" src="https://github.com/user-attachments/assets/0c50b9bd-02b2-4f65-a3e2-aee396fb6504" />

- Проверяем связность с ISP

  ` ping 178.207.179.1 `

 **RTR-A**:

 Аналогично с RTR-COD задаем hostname & domain-name:

 ``` cisco
  ecorouter>enable
  ecorouter#configure terminal 
  Enter configuration commands, one per line.  End with CNTL/Z.
  ecorouter(config)#hostname rtr-a
  rtr-cod(config)#ip domain-name office.ssa2026.region
  rtr-cod(config)#write memory

  ```

Назначение IP на устройство:

- создание интерфейса в сторону ISP

``` cisco

rtr-a(config)#interface isp
rtr-a(config-if)#ip address 178.207.179.4/29
rtr-a(config-if)#exit
rtr-a(config)#

```

<img width="684" height="139" alt="изображение" src="https://github.com/user-attachments/assets/ee09c11a-e992-4838-8993-588d2685cce6" />

- создание service instance on te0 port

``` cisco

rtr-a(config)#port te0
rtr-a(config-port)#service-instance te0/isp
rtr-a(config-service-instance)#encapsulation untagged 
rtr-a(config-service-instance)#connect ip interface isp 
rtr-a(config-service-instance)#exit
rtr-a(config-port)#exit
rtr-a(config)#

```


<img width="799" height="261" alt="изображение" src="https://github.com/user-attachments/assets/8d79b21d-9f14-4022-9545-8997fcbeee0f" />


Задаем статический маршрут на ISP

``` cisco

rtr-a(config)#ip route 0.0.0.0/0 178.207.179.25
rtr-a(config)#

```

<img width="633" height="118" alt="изображение" src="https://github.com/user-attachments/assets/b69c21f1-6bc3-4b38-94f9-1fa81fdfa256" />

<img width="609" height="210" alt="изображение" src="https://github.com/user-attachments/assets/46264dc4-dcd7-425f-8d18-93523cdb3052" />


- Реализация VLAN
  - создание подинтерфейсов для VLAN, назначаем на них соответствующие IP адреса:

``` cisco

rtr-a(config)#interface vl100
rtr-a(config-if)#ip address 172.20.10.254/24
rtr-a(config-if)#description "VLAN - SRV"
rtr-a(config-if)#exit
rtr-a(config)#
rtr-a(config)#interface vl200
rtr-a(config-if)#ip address 172.20.20.254/24
rtr-a(config-if)#description "VLAN - CLI"
rtr-a(config-if)#exit
rtr-a(config)#
rtr-a(config)#interface vl300
rtr-a(config-if)#ip address 172.20.30.254/24
rtr-a(config-if)#description "VLAN - MGMT"
rtr-a(config-if)#exit
rtr-a(config)#

```

- проверяем IP на интерфейсах

<img width="703" height="162" alt="изображение" src="https://github.com/user-attachments/assets/7ae631e5-dc7e-410e-b702-8d79c366af38" />

- производим подвязку каждого подинтерфейса VLAN к физическому интерфейсу te1

``` cisco

rtr-a(config)#port te1
rtr-a(config-port)#service-instance te1/vl100
rtr-a(config-service-instance)#encapsulation dot1q 100
rtr-a(config-service-instance)#rewrite pop 1
rtr-a(config-service-instance)#connect in
rtr-a(config-service-instance)#connect ip interface vl100                   
rtr-a(config-service-instance)#exit
rtr-a(config-port)#
rtr-a(config-port)#service-instance te1/vl200
rtr-a(config-service-instance)#encapsulation dot1q 200
rtr-a(config-service-instance)#rewrite pop 1
rtr-a(config-service-instance)#connect ip interface vl200 
rtr-a(config-service-instance)#exit
rtr-a(config-port)#
rtr-a(config-port)#service-instance te1/vl300
rtr-a(config-service-instance)#encapsulation dot1q 300
rtr-a(config-service-instance)#rewrite pop 1
rtr-a(config-service-instance)#connect ip interface vl300 
rtr-a(config-service-instance)#exit
rtr-a(config-port)#exit
rtr-a(config)#write memory

```

Проверяем IP и статус интерфейсов

<img width="631" height="158" alt="изображение" src="https://github.com/user-attachments/assets/aa501d3a-ab2b-44a8-8cd4-3797721610d5" />

<img width="799" height="252" alt="изображение" src="https://github.com/user-attachments/assets/bec21b8b-140d-40be-986a-4fdc504173bc" />



### 2. fw-cod (ideco):

#### Настройка интерфейса управления

-   Для корректной идентификации сетевой карты используйте MAC-адрес сетевой карты
-   Для настройки Ideco NGFW Novum через веб-интерфейс настройте Control Plane интерфейс в локальном меню
    -   Control Plane - интерфейс администрирования, используется для настройки NGFW Novum через браузер и должен иметь свой выход в Интернет
-   Выполните вход из под созданного пользователя **admin** с паролем **P@ssw0rd1234**

 <img width="458" height="228" alt="изображение" src="https://github.com/user-attachments/assets/1ac743b6-3c07-4d25-9201-84ff1c13b3cb" />


-   Перейдите в раздел **Ethernet-интерфейсы (3)** → **Создать интерфейс (3)**:

<img width="841" height="800" alt="изображение" src="https://github.com/user-attachments/assets/5103c4ef-0cd1-4425-b2c9-7197a0b1d637" />


##### Пояснения в контексте текущего задания:

-   Поскольку Ideco рекомендуется настраивать через веб-интерфейс, надо настроить IP-адрес для этого доступа
-   Поскольку в сod.ssa2026.region между fw-cod и sw1-cod необходимо организовать агрегированное соединение 802.3ad, (**не актуально, от 802.3ad в задании отказались**)
    -   а также за маршрутизацию между VLAN-ами по топологии будет отвечать fw-cod,
    -   то так называемые под-интерфейсы необходимо реализовывать поверх агрегированного канала 802.3ad,
    -   в консоле Ideco - это реализовать не получится, править конфигурационные файлы в ideco плохо, можно лишиться технической поддержки.
-   Поэтому как один из способов доступа к веб-интерфейсу это:
    -   использование интерфейса в сторону rtr-cod,
    -   после реализации туннеля и маршрутизации между rtr-cod и rtr-a, 
    -   а также коммутации между sw1-a и sw2-a,
    -   у нас появится возможность конфигурировать fw-cod через веб-интерфейс, например с cli1-a или cli2-a,
    -   вследствии чего и полная связность между устройствами сod.ssa2026.region и office.ssa2026.region

##### Продолжение настройки:

-   Выберите порт для доступа к веб-интерфейсу:
    -   сравнив МАС-адреса на уровне виртуальной машины

<img width="1111" height="731" alt="изображение" src="https://github.com/user-attachments/assets/e9e28d9c-3e66-45f4-bd34-26a98c045c69" />

-   Выберите порт для доступа к веб-интерфейсу:
    -   в текущем случае выбирается порт в сторону виртуальной машины **rtr-cod**

<img width="848" height="253" alt="изображение" src="https://github.com/user-attachments/assets/f8c0c79b-19b9-4cc1-809a-b9fb1ee6468f" />


-   Введите имя интерфейса:

<img width="552" height="97" alt="изображение" src="https://github.com/user-attachments/assets/4d84c432-006f-445f-ad11-bc236b20d5a8" />


-   Выберите роль **LAN**:

<img width="398" height="163" alt="изображение" src="https://github.com/user-attachments/assets/1607c709-c5fd-4308-8684-155efd4f44d4" />


-   Выберите **Корневой контекст**:

<img width="365" height="133" alt="изображение" src="https://github.com/user-attachments/assets/1a5bd115-54f8-4126-a2f5-b0a217b1437b" />


-   Настройте локальную сеть **Вручную**:

<img width="378" height="159" alt="изображение" src="https://github.com/user-attachments/assets/ec310178-3984-4b24-9d51-298dca41e604" />


-   Ведите локальный IP-адрес и маску подсети в формате **ip/маска** и нажмите **Enter**:

<img width="429" height="112" alt="изображение" src="https://github.com/user-attachments/assets/f3cf772e-0124-41a8-855f-cfab4d4d8df8" />


-   Проверить можно выбрав соответствующий пункт меню:

<img width="961" height="347" alt="изображение" src="https://github.com/user-attachments/assets/aa32c0cf-bf78-40e0-8ab9-c12db78fb3cc" />


-   Также при вводе с клавиатуры "**с**" и нажатие **Enter** или же сочетание клавич **Ctrl + D**:
    -   можно увидеть адрес и порт для доступа к веб-интерфейсу для дальнейшей настройки

<img width="740" height="376" alt="изображение" src="https://github.com/user-attachments/assets/4a5e01fd-37ca-40e7-863d-2d9bf1fbe380" />

### rtr-cod (ecorouter):

#### Базовая настройка BGP:

-   Запустите протокол BGP, указав нужную автономную систему, командой: **router bgp <№>**:

```
rtr-cod(config)#router  bgp 64500
rtr-cod(config-router)#
```

-   Указать уникальный идентификатор маршрутизатора в протоколе BGP, командой: **bgp router-id <IP>**:

```
rtr-cod(config-router)#bgp router-id 178.207.179.4
rtr-cod(config-router)#
```

-   Сконфигурируйте BGP соседство c Интернет провайдером **ISP**, указав адрес соседа и номер локальной AS, используя команду: **neighbor <NEIGHBOR\_IP> remote-as <$>**:

```
rtr-cod(config-router)#neighbor 178.207.179.1 remote-as 31133
rtr-cod(config-router)#exit
rtr-cod(config)#write memory
Building configuration...

rtr-cod(config)#
```

-   Проверить состояние всех соединений BGP можно командой **show ip bgp summary** из режима администрирования (**enable**):

<img width="844" height="253" alt="изображение" src="https://github.com/user-attachments/assets/d85932b9-185e-4f9c-8dd8-98435201ab47" />


-   Также по условиям задания **rtr-cod** должен получать маршрут по умолчанию по BGP
-   Проверить маршрут по умолчанию можно командой **show ip route** из режима администрирования (**enable**):
<img width="755" height="273" alt="изображение" src="https://github.com/user-attachments/assets/aac91464-a47a-460b-81fd-d8848ed94e67" />


-   Проверить доступ в сеть Интернет:
