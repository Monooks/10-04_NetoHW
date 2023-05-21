# Домашнее задание к занятию 10.4 "`Резервное копирование`" - `Емельянов Михаил`

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

Желаем успехов в выполнении домашнего задания!

---

### Задание 1

В чём разница между:

- полным резервным копированием,
- дифференциальным резервным копированием,
- инкрементным резервным копированием.

*Приведите ответ в свободной форме.*

#### ОТВЕТ:

Полное копирование (Full backup) – копирование данных в полном объеме. Самый надежный способ копирования. В случае выхода из строя свежей копии данные можно восстановить из предыдущих копий. Эффективный и быстрый метод восстановления. Недостаток – требует носителей большого объема и длительного времени выполнения.
Дифференциальное копирование (Differential backup) — копируются файлы, изменившиеся после последнего Full backup. Данные копируются «нарастающим итогом», так что последняя копия всегда будет содержать все изменения с момента последнего Full backup. Выполняется быстрее чем Full backup, и повреждение одной из копий не приводит к потере всех данных за последующий и предыдущий период (при наличии живого Full backup). Так или иначе требуется регулярный Full backup и бывает, что последняя копия (при длительной работе) по размеру изменений приближается к Full backup.
Инкрементное копирование (Incremental backup) — выполняется копирование только информации, измененной после выполнения предыдущего Incremental backup. Это самый быстрый метод резервирования и занимает меньше всего объема, но и самый ненадежный метод. В случае повреждения одной из копий все последующие становятся бесполезны. И соответственно при повреждении Full backup все становиться негодным. Восстановление данных занимает продолжительное время.

---

### Задание 2

Установите программное обеспечении Bacula, настройте bacula-dir, bacula-sd,  bacula-fd. Протестируйте работу сервисов.

*Пришлите:*
*- конфигурационные файлы для bacula-dir, bacula-sd,  bacula-fd;*
*- скриншот, подтверждающий успешное прохождение резервного копирования.*

#### ОТВЕТ:

![bacula-dir](https://github.com/Monooks/10-04_NetoHW/blob/main/img/bacula-dir.conf)
![bacula-sd](https://github.com/Monooks/10-04_NetoHW/blob/main/img/bacula-sd.conf)
![bacula-fd](https://github.com/Monooks/10-04_NetoHW/blob/main/img/bacula-fd.conf)
![Скриншот-1](https://github.com/Monooks/10-04_NetoHW/blob/main/img/10.04_1.png)

---

### Задание 3

Установите программное обеспечении Rsync. Настройте синхронизацию на двух нодах. Протестируйте работу сервиса.

*Пришлите рабочую конфигурацию сервера и клиента Rsync блоком кода в вашем md-файле.*

#### ОТВЕТ:

Нода 1 (клиент)

1. /etc/rsyncd.conf
```
pid file = /var/run/rsyncd.pid
log file = /var/log/rsyncd.log
transfer logging = true
munge symlinks = yes
[data]
path = /root/
uid = root
read only = yes
list = yes
comment = Data backup Dir ][a][a][a
auth users = backup
secrets file = /etc/rsyncd.scrt
[new]
path = /etc/
uid = root
read only = yes
list = yes
comment = System fileset
auth users = backup
secrets file = /etc/rsyncd.scrt
```
2. /etc/rsyncd.scrt
```
backup:12345
```
Нода 2 (сервер)
1. /root/scripts/backup-node1.sh
``` bash
srv_user=backup
# Ресурс на сервере для бэкапа
srv_dir=data
echo "Start backup ${srv_name}"
# Создаем папку для инкрементных бэкапов
mkdir -p ${syst_dir}${srv_name}/increment/
/usr/bin/rsync -avz --progress --delete --password-file=/etc/rsyncd.scrt ${srv_user}@${srv_ip}::${srv_dir} ${syst_dir}${srv_name}/current/ --backup --backup-dir=${syst_dir}${srv_name}/increment/`date +%Y-%m-%d`/
/usr/bin/find ${syst_dir}${srv_name}/increment/ -maxdepth 1 -type d -mtime +30 -exec rm -rf {} \;
date
echo "Finish backup ${srv_name}"

srv_dir=new
echo "Start backup ${srv_name}"
# Создаем папку для инкрементных бэкапов
mkdir -p ${syst_dir}${srv_name}/increment/
/usr/bin/rsync -avz --progress --delete --password-file=/etc/rsyncd.scrt ${srv_user}@${srv_ip}::${srv_dir} ${syst_dir}${srv_name}/current/ --backup --backup-dir=${syst_dir}${srv_name}/increment/`date +%Y-%m-%d`/
/usr/bin/find ${syst_dir}${srv_name}/increment/ -maxdepth 1 -type d -mtime +30 -exec rm -rf {} \;
date
echo "Finish backup ${srv_name}"
```
2. /etc/rsyncd.scrt
```
12345
```
---

### Задание со звёздочкой*
Это задание дополнительное. Его можно не выполнять. На зачёт это не повлияет. Вы можете его выполнить, если хотите глубже разобраться в материале.

---

### Задание 4*

Настройте резервное копирование двумя или более методами, используя одну из рассмотренных команд для папки /etc/default. Проверьте резервное копирование.

*Пришлите рабочую конфигурацию выбранного сервиса по поставленной задаче.*