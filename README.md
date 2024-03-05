# Домашнее задание к занятию «Уязвимости и атаки на информационные системы - Савельев Алексей SYS-25»

При выполнении данного задания использовалась информация с этого [сайта](https://nmap.org)

------

### Задание 1

Скачайте и установите виртуальную машину Metasploitable: https://sourceforge.net/projects/metasploitable/.

Это типовая ОС для экспериментов в области информационной безопасности, с которой следует начать при анализе уязвимостей.

Просканируйте эту виртуальную машину, используя **nmap**.

Попробуйте найти уязвимости, которым подвержена эта виртуальная машина.

Сами уязвимости можно поискать на сайте https://www.exploit-db.com/.

Для этого нужно в поиске ввести название сетевой службы, обнаруженной на атакуемой машине, и выбрать подходящие по версии уязвимости.

Ответьте на следующие вопросы:

- Какие сетевые службы в ней разрешены?
- Какие уязвимости были вами обнаружены? (список со ссылками: достаточно трёх уязвимостей)
  
*Приведите ответ в свободной форме.*  

---
### Ответ 1

При сканировании утилитой `nmap`

```bash 
nmap -sV 192.168.0.106 
``` 
были обнаружены следующие сервисы:
 - 1 столбец: порт/протокол
 - 2 столбец: статус порта (открыт/закрыт)
 - 3 столбец: краткое название сервиса
 - 4 столбец: расширенное название и версия сервиса

![Таблица](src/tabl.png)

1. При помощи nmap просканировал порт 22 (ssh) и обнаружил простую связку пользователь:пароль (user:user)

![ssh](src/ssh.png)

2. По примеру из лекции, с помощью `msfconsole` зашёл через backdoor по ftp:

![Поиск](src/search.png)
![Консоль](src/msfconsole.png)
![Задняя дверь](src/backdoor.png)

3. Так же обнаружены куки-файлы на сервере:
```bash 
nmap -sV -p 80 192.168.0.106 --script http-c*
```
![Куки](src/coockie.png)

---

### Задание 2

Проведите сканирование Metasploitable в режимах SYN, FIN, Xmas, UDP.

Запишите сеансы сканирования в Wireshark.

Ответьте на следующие вопросы:

- Чем отличаются эти режимы сканирования с точки зрения сетевого трафика?
- Как отвечает сервер?

*Приведите ответ в свободной форме.*

---
### Ответ 2
SYN-сканирование:
Эту технику часто называют сканированием с использованием полуотрытых соединений, т.к. не открываестя полное TCP соединение. Я посылаю SYN пакет, как если бы хотел установить реальное соединение и жду. Ответы SYN/ACK указывают на то, что порт прослушивается (открыт), а RST (сброс) на то, что не прослушивается. Если после нескольких запросов не приходит никакого ответа, то порт помечается как фильтруемый. Порт также помечается как фильтруемый, если в ответ приходит ICMP сообщение об ошибке недостижимости (тип 3, код 1,2, 3, 9, 10 или 13). Домонстрация по порту 23 (выделен голубым цветом)

![sS](src/sS.png)
---

UDP - сканирование работает путем посылки пустого (без данных) UDP заголовка на каждый целевой порт. Если в ответ приходит ICMP ошибка о недостижимости порта (тип 3, код 3), значит порт закрыт. Другие ICMP ошибки недостижимости (тип 3, коды 1, 2, 9, 10 или 13) указывают на то, что порт фильтруется. Иногда, служба будет отвечать UDP пакетом, указывая на то, что порт открыт. Если после нескольких попыток не было получено никакого ответа, то порт классифицируется как открыт/фильтруется. Это означает, что порт может быть открыт, или, возможно, пакетный фильтр блокирует его. Функция определения версии (-sV) может быть полезна для дифференциации действительно открытых портов и фильтруемых.

![UDP](src/udp.png)
---
FIN и Xmas-сканирование:
Когда сканируется система отвечающая требованиям RFC, любой пакет, не содержащий установленного бита SYN, RST или ACK, повлечет за собой отправку RST в ответ, в случае если порт закрыт, или не повлечет никакого ответа, если порт открыт. Т.к. ни один из этих битов не установлен, то комбинация FIN будет являться правильной. Nmap использует это в трех типах сканирования:FIN, Xmas, Null. Если в ответ приходит RST пакет, то порт считается закрытым, отсутствие ответа означает, что порт открыт/фильтруется. Порт помечается как фильтруется, если в ответ приходит ICMP ошибка о недостижимости (тип 3, код 1, 2, 3, 9, 10 или 13).
![Запрос](src/reqest.png)
![Нет ответа](src/noansw.png)

Ключевой особенностью этих типов сканирования является их способность незаметно обойти некоторые не учитывающие состояние (non-stateful) брандмауэры и роутеры с функцией пакетной фильтрации. Еще одним преимуществом является то, что они даже чуть более незаметны, чем SYN сканирование. Все же не надо на это полагаться - большинство современных IDS могут быть сконфигурированы на их обнаружение. Большим недостатком является то, что не все системы следуют RFC 793 дословно. Некоторые системы посылают RST ответы на запросы не зависимо от того, открыт порт или закрыт. Это приводит к тому, что все порты помечаются как закрытые. Основными системами ведущими себя подобным образом являются Microsoft Windows, многие устройства Cisco, BSDI и IBM OS/400. Хотя такое сканирование применимо к большинству систем, основанных на Unix. Еще одним недостатком этих видов сканирования является их неспособность разделять порты на открытые и фильтруемые, т.к. порт помечается как открыт|фильтруется.
---


---