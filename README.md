# module2026
Общая цель:
Настроить сетевую инфраструктуру из двух офисов (HQ и BR), соединённых через провайдера ISP, с обеспечением:
- доступности сервисов между офисами;
- выхода в интернет;
- доменной службы, DNS, NTP, файлового хранилища;
- веб-приложений (Apache + MariaDB и Docker + MariaDB);
- централизованного управления (Ansible);
- безопасности удалённого доступа и проброса портов.

Основные компоненты и их настройки
1. Базовая адресация и имена устройств
Заданы имена хостов (ISP, HQ-RTR, BR-RTR, HQ-SRV, BR-SRV, HQ-CLI).
Назначены IP-адреса, маски, шлюзы согласно таблице.
VLAN:
VLAN 100 – для HQ-SRV (до 32 адресов)
VLAN 200 – для HQ-CLI (до 16 адресов)
VLAN 999 – управление (до 8 адресов)

2. Маршрутизация и туннель
GRE-туннель между HQ-RTR и BR-RTR.
Динамическая маршрутизация OSPF через FRR (только по туннелю, с паролем).

3. Доступ в интернет
На ISP: IP-forward и NAT (masquerade).
На HQ-RTR и BR-RTR – NAT для локальных сетей в сторону ISP.

4. DHCP (HQ-CLI)
isc-dhcp-server на HQ-RTR.
Исключён адрес роутера, DNS – HQ-SRV, DNS-суффикс – au-team.irpo.

5. DNS
Dnsmasq на HQ-SRV (основной DNS).
Для домена au-team.irpo – пересылка на Samba DC (BR-SRV).
Прямые и обратные записи для устройств (A, PTR).

6. Samba Domain Controller (BR-SRV)
Домен au-team.irpo.
Созданы пользователи hquser1..5 и группа hq.
HQ-CLI введён в домен.
Права sudo для группы hq ограничены (cat, grep, id).

7. Файловое хранилище и NFS
RAID 0 (md0) из двух дисков, монтирование в /raid.
NFS-сервер на HQ-SRV – папка /raid/nfs для сети HQ-CLI.
На HQ-CLI – автомонтирование в /mnt/nfs.

8. Синхронизация времени (Chrony)
ISP – NTP-сервер (стратум 5).
HQ-SRV, HQ-CLI, BR-RTR, BR-SRV – клиенты ISP.

9. Ansible (BR-SRV)
Инвентарь включает HQ-SRV, HQ-CLI, HQ-RTR, BR-RTR.
Проверка ping – pong.

10. Docker-приложение (BR-SRV)
Образы site_latest и mariadb_latest из Additional.iso.
Контейнеры: testapp (web) и db (MariaDB).
БД: testdb, пользователь test/P@ssw0rd.
Доступно на порту 8080.

11. Веб-приложение на Apache (HQ-SRV)
LAMP: Apache + MariaDB + PHP.
Импорт БД webdb из dump.sql, пользователь web/P@ssw0rd.
Файлы приложения (index.php, logo.png) – из Additional.iso.

12. Проброс портов (статический DNAT)
BR-RTR:
8080 → BR-SRV:8080 (Docker)
2026 → BR-SRV:2026 (SSH)

HQ-RTR:
8080 → HQ-SRV:80 (Apache)
2026 → HQ-SRV:2026 (SSH)

13. Обратный прокси (nginx) на ISP
web.au-team.irpo → веб-приложение HQ-SRV
docker.au-team.irpo → Docker-приложение BR-SRV
Web-based аутентификация для web.au-team.irpo (пользователь WEB/P@ssw0rd).

14. Дополнительно
Часовой пояс – Asia/Yekaterinburg.
На HQ-CLI установлен Яндекс Браузер.

> [!WARNING]
> На экзамене присутствует внешняя комиссия. 
> **Используйте на свой страх и риск.** Обязательно изучите код перед использованием - вы должны понимать каждую настройку и уметь объяснить её вручную.

### Как расположены команды 

В репозитории есть файлы в которых расположены команды.
Они разделены на модули(например: module_1), в них входит всё необходимое для решения конкретного модуля.

> [!WARNING]
> Настраиваем последовательно по гайду, иначе возможны проблемы 
>

### Дополнительная информация 

<img width="926" height="295" alt="image" src="https://github.com/user-attachments/assets/2d035611-6663-4833-b5c5-b997d802a5a8" />

<img width="1071" height="245" alt="image" src="https://github.com/user-attachments/assets/a86d4eeb-5efc-4a28-a26a-7052c526d58e" />

<img width="945" height="116" alt="image" src="https://github.com/user-attachments/assets/904f9972-16a5-467e-a4fb-e7dc96c8d3fe" />

<img width="948" height="183" alt="image" src="https://github.com/user-attachments/assets/90e24cad-7842-41dd-bec8-1593c419aaec" />
