# NhkReg2026 - Мой вариант решения регионального этапа чемпионата "Профессионалы" по СиСА 2026 года. Alt + Ecorouter

## МОДУЛЬ Б:
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


- Создадим интерфейс с именем isp и назначим на него IP-адрес 178.207.179.4/29, также зададим для данного интерфейса описание


``` cisco
rtr-cod(config)#interface isp
rtr-cod(config-if)#ip address 178.207.179.4/29
rtr-cod(config-if)#exit
rtr-cod(config)#
```


