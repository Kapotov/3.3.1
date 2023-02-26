# Домашнее задание к занятию «Операционные системы. Лекция 1»

### Цель задания

В результате выполнения задания вы:

* познакомитесь с инструментом strace, который помогает отслеживать системные вызовы процессов и является необходимым для отладки и расследований при возникновении ошибок в работе программ;
* рассмотрите различные режимы работы скриптов, настраиваемые командой set. Один и тот же код в скриптах в разных режимах работы ведёт себя по-разному.

### Чеклист готовности к домашнему заданию

1. Убедитесь, что у вас установлен инструмент `strace`, выполнив команду `strace -V` для проверки версии. В Ubuntu 20.04 strace установлен, но в других дистрибутивах его может не быть в коплекте «из коробки». Обратитесь к документации дистрибутива, чтобы понять, как установить инструмент strace.
![2023-02-26_21-37-18](https://user-images.githubusercontent.com/123774335/221429891-d95d0d74-4816-4508-b3dc-88a299572d2f.png)

2. Убедитесь, что у вас установлен пакет `bpfcc-tools`, информация по установке [по ссылке](https://github.com/iovisor/bcc/blob/master/INSTALL.md).

![2023-02-26_21-42-22](https://user-images.githubusercontent.com/123774335/221430167-d0a6f81e-d430-4300-a930-982c67c9f34c.png)

### Дополнительные материалы для выполнения задания

1. Изучите документацию lsof — `man lsof`. Та же информация есть [в сети](https://linux.die.net/man/8/lsof).
2. Документация по режимам работы bash находится в `help set` или [в сети](https://www.gnu.org/software/bash/manual/html_node/The-Set-Builtin.html).

------

## Задание

1. Какой системный вызов делает команда `cd`? 

    В прошлом ДЗ вы выяснили, что `cd` не является самостоятельной  программой. Это `shell builtin`, поэтому запустить `strace` непосредственно на `cd` не получится. Вы можете запустить `strace` на `/bin/bash -c 'cd /tmp'`. В этом случае увидите полный список системных вызовов, которые делает сам `bash` при старте. 

    Вам нужно найти тот единственный, который относится именно к `cd`. Обратите внимание, что `strace` выдаёт результат своей работы в поток stderr, а не в stdout.
    
# Ответ: chdir
![2023-02-26_22-02-58](https://user-images.githubusercontent.com/123774335/221431211-72ced195-bc40-42f3-a08d-238b8bf14eaa.png)


2. Попробуйте использовать команду `file` на объекты разных типов в файловой системе. Например:

    ```bash
    vagrant@netology1:~$ file /dev/tty
    /dev/tty: character special (5/0)
    vagrant@netology1:~$ file /dev/sda
    /dev/sda: block special (8/0)
    vagrant@netology1:~$ file /bin/bash
    /bin/bash: ELF 64-bit LSB shared object, x86-64
    ```
    
    Используя `strace`, выясните, где находится база данных `file`, на основании которой она делает свои догадки.
    
# Ответ: "/usr/share/misc/magic.mgc" - БД

![2023-02-26_21-55-26](https://user-images.githubusercontent.com/123774335/221430876-35a0b40b-4442-4aec-a203-334be9cdd608.png)

3. Предположим, приложение пишет лог в текстовый файл. Этот файл оказался удалён (deleted в lsof), но сказать сигналом приложению переоткрыть файлы или просто перезапустить приложение возможности нет. Так как приложение продолжает писать в удалённый файл, место на диске постепенно заканчивается. Основываясь на знаниях о перенаправлении потоков, предложите способ обнуления открытого удалённого файла, чтобы освободить место на файловой системе.

# Ответ: cat /dev/null > /proc/PID/fd/number

4. Занимают ли зомби-процессы ресурсы в ОС (CPU, RAM, IO)?
 
# Ответ: зомби-процессы не занимают CPU, RAM но могут заблокировать IO программы и она не будет запускаться.

5. В IO Visor BCC есть утилита `opensnoop`:

    ```bash
    root@vagrant:~# dpkg -L bpfcc-tools | grep sbin/opensnoop
    /usr/sbin/opensnoop-bpfcc
    ```
    
    На какие файлы вы увидели вызовы группы `open` за первую секунду работы утилиты? Воспользуйтесь пакетом `bpfcc-tools` для Ubuntu 20.04. Дополнительные сведения по установке [по ссылке](https://github.com/iovisor/bcc/blob/master/INSTALL.md).
    # Ответ: 
  
```
![image](https://user-images.githubusercontent.com/123774335/221432008-e3133daa-46da-4901-92e9-199337fa417d.png)
   ```
  

6. Какой системный вызов использует `uname -a`? Приведите цитату из man по этому системному вызову, где описывается альтернативное местоположение в `/proc` и где можно узнать версию ядра и релиз ОС.
 # Ответ:  
```
системный вызов uname

Part of the utsname information is also accessible via  
       /proc/sys/kernel/{ostype, hostname, osrelease, version,  
       domainname}. (c)
```

7. Чем отличается последовательность команд через ; и через && в bash? Например: 
```
oot@netology1:~# test -d /tmp/some_dir; echo Hi
Hi
root@netology1:~# test -d /tmp/some_dir && echo Hi
root@netology1:~#
```
# Ответ:

```
'&&' - это логический оператор "И". Используется для объединения команд,  
таким образом, что следующая команда после "&&" выполнится в случае если  
предыдущая завершится с кодом возврата 0.    
с помощью ';' можно разделить команды и записать в одну строку
```
Есть ли смысл использовать в bash &&, если применить set -e?
# Ответ: не имеет, так как set -e завершит работу конвеера, если команда завершится с кодом возврата отличного от 0

8. Из каких опций состоит режим ```bash set -euxo pipefail``` и почему его хорошо было бы использовать в сценариях?
# Ответ:

```
-e Немедленный выход, если команда завершается с ненулевым статусом.
-u Рассматривать неустановленные переменные как ошибку, с выводом в stderr
-x Печатать команды и их аргументы по мере их выполнения.
-o pipefail Возвращает результат работы всего конвеера по результат работы  
последней команды завершенной с ненулевым статусом, или ноль — если работа всех команд завершена успешно.

Использование этих оцпий посволяет понимать работу конвеера, и на каком этапе могут  
вознинуть ошибки
```
9. Используя -o stat для ps, определите, какой наиболее часто встречающийся статус у процессов в системе. В man ps ознакомьтесь (/PROCESS STATE CODES) что значат дополнительные к основной заглавной буквы статуса процессов. Его можно не учитывать при расчете (считать S, Ss или Ssl равнозначными).
# Ответ:

```
root@vagrant:~# ps -o stat
+ ps -o stat
STAT
S - Прерываемый сон (ожидание завершения события) 
S
R+ работающий или работающий (в очереди выполнения) ('+' находится в группе процессов переднего плана)
```

### Правила приёма домашнего задания

В личном кабинете отправлена ссылка на .md-файл в вашем репозитории.


### Критерии оценки

Зачёт:

* выполнены все задания;
* ответы даны в развёрнутой форме;
* приложены соответствующие скриншоты и файлы проекта;
* в выполненных заданиях нет противоречий и нарушения логики.

На доработку:

* задание выполнено частично или не выполнено вообще;
* в логике выполнения заданий есть противоречия и существенные недостатки.  
