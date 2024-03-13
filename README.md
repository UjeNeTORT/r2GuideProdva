# Быстрый вкат в radare2
----
Если вы здесь, значит вы хардкорный линуксоид и дед не смог переманить вас на винду ради IDA. Ну или вы просто крутой чел.
В любом случае r2 не супер сложный ~~хахха~~.

## Как скачать
> Склонировать их репозиторий.

![страничка откуда копировать команды](image.png)
1. https://rada.re/n/radare2.html
2. тык radare 2
3. в консоль вставляте команды из окошка

Вставляйте команды по одной и не забудьте про
```sys/install.sh``` - это отдельная команда, а не комментарий. Точка с запятой - обманка, не ведитесь.

---

Чтобы проверить, что все ок:
```
netort@thinkpad-x1:~$ r2
Usage: r2 [-ACdfjLMnNqStuvwzX] [-P patch] [-p prj] [-a arch] [-b bits] [-c cmd]
          [-s addr] [-B baddr] [-m maddr] [-i script] [-e k=v] file|pid|-|--|=

```
## Дизассемблируем первый файл

```
netort@thinkpad-x1:~$ r2 <путь_до_исполняемого_файла>
WARN: Relocs has not been applied. Please use `-e bin.relocs.apply=true` or `-e bin.cache=true` next time
 -- rip and tear until it compiles
[0x00401040]>
```
- WARN - я всегда его игнорил. вроде ничего критического
- все что после ```--``` - шутка (тут они смешные кста)
- ```[0x00401040]>``` - так вы будете общаться с r2 в момент входа

Вписываем
```[0x00401040]> V``` - (регистр важен!) - и у нас открывается hexdump view
![hexdumpview](image-1.png)
![disasmview](image-2.png)
![dbg-view](image-3.png)
Нажимая на ```p``` мы можем переключаться между видами нашей проги в таком порядке:   
**hexdump** -> **disassemble** -> **debug** -> entropy -> strings -> и так по кругу...

Жирным выделено то, чем вы реально будете пользоваться.

## База по навигации

Теперь освоим азы перемещения по этим окошкам.

Управление как в vim:
стрелочками тоже можно, но я бы рекомендовал уже приучаться к 'h', 'j', 'k', 'l' - это в миллион раз удобнее.

*Курсор:*
```c``` - чтобы двигаться по отдельным байтам вашей проги. Это может быть полезно, когда вам надо будет пропатчить бинарник.

*Комментарии*:
```;``` - в режиме курсора так вы можете оставлять комментарии к отдельным байтам или целым строчкам.

![alt text](image-4.png)

## Как патчить бинарник

> Чтобы пропатчить файл, его нужно открыть в режиме редактирования: 
```r2 -w <path>```   
 **НЕ ЗАБУДЬТЕ СДЕЛАТЬ РЕЗЕРВНУЮ КОПИЮ ФАЙЛА**

- режим курсора ```c```
- ```hjkl``` - подкрадываетесь к нужному байту
-  ```i``` - insert
- меняете байт, победа!

## Как дебажить прогу
> Чтобы поймать багулину файл открываем в дебаг режиме: ```r2 -d <path>``` 

### Вы знаете где точка входа (вы явно указали ее в ассемблерном файле)
В этом случае вас загрузит прямо в ней.
### Вы не знаете где точка входа (например, она в функции main в си файле)
Пишем ```aaa``` - эта команда выполнит анализ бинарника.   
```[0x7b232ed29290]> aaa
INFO: Analyze all flags starting with sym. and entry0 (aa)
INFO: Analyze imports (af@@@i)
INFO: Analyze entrypoint (af@ entry0)
INFO: Analyze symbols (af@@@s)
INFO: Recovering variables
INFO: Analyze all functions arguments/locals (afva@@@F)
INFO: Analyze function calls (aac)
INFO: Analyze len bytes of instructions for references (aar)
INFO: Finding and parsing C++ vtables (avrr)
INFO: Analyzing methods
INFO: Recovering local variables (afva)
INFO: Skipping type matching analysis in debugger mode (aaft)
INFO: Propagate noreturn information (aanr)
INFO: Use -AA or aaaa to perform additional experimental analysis
```
Потом ```afl``` - *analyze function list* - так радар распознает ваши функции.
```
0x00401020    1     37 entry0
0x00401060    4     31 sym.deregister_tm_clones
0x00401090    4     49 sym.register_tm_clones
0x004010d0    3     32 sym.__do_global_dtors_aux
0x00401100    1      6 sym.frame_dummy
0x00401380    1     13 sym._fini
0x00401050    1      5 sym._dl_relocate_static_pie
0x00401314    1    105 main
0x00401110    1     18 loc.myprintf
0x00401122    9     86 loc.myprintf_cdecl
0x00401000    3     27 sym._init
```

Наконец, команда ```db <address_or_funcname>``` - поставить точку останова. (```db main``` в нашем случае).

Finally, команда ```dc``` - выполнять код до точки останова.
```
[0x7b232ed29290]> dc
INFO: hit breakpoint at: 0x401314
```
## Как дебажить то??
Теперь можно перейти в визуальный режим: ```V``` и переключиться на debug view ```p```x2
> В визуальном режиме (```V```) у вас не будет консоли, но к ней можно получить доступ, нажав ```:```: все команды дальше вы будете вводить через эту кнопку.
![alt text](image-5.png)

- ```F7``` и ```F8``` - step into и step out - как в turbo debugger досбокса, удобно!   
- Если вы пропустили нужное место, вы можете быстро перезагрузить дебаггер прямо из радара: ```ood```. В этом случае вас загрузит по стандартному адресу и вам придется заново выставлять точку останова на ```main``` и после этого ```dc``` на нее.   
 *(вообще инфа по точкам останова сохраняется и ```db``` писать не обязательно, можно сразу ```dc```, если вы не путаетесь)*

## Кастомизация

Для себя я настроил довольно прикольную тему.