# Drall
Bulk operations for Drupal sites.

## Depedencies
- [amoffat/sh](https://github.com/amoffat/sh)
- [popstas/server-scripts](https://github.com/popstas/server-scripts) (`drupal-get-drupals` and `cp-chown`, `drupal-patch` in examples)
- [popstas/drupal-scripts](https://github.com/popstas/drupal-scripts) (`drs` in examples)
- uses `drupal-get-drupals` command,  you can change variable `get_drupals_command` in script
- search match-tests in `/usr/share/drall/drupal-tests/`, you can change variable `tests_root_path` in script, see `/match-tests-examples`

## Features
- поиск всех папок друпала
- фильтрация папок
- переход в корневую папку друпала
- выполнение заданной команды от имени юзера сайта или root

## Installation
```
git clone https://github.com/popstas/drall.git
cd drall
./install.sh
```


## Usage
Common usage: `drall [options] ["command or command_file_path"]`

```
Options:
  -h, --help            show this help message and exit
  -r, --asroot          Exec as root
  -q, --quiet           don't print status messages to stdout
  -v, --verbose         Verbose
  -n NUMBER             Only NUMBER first sites. 0 = all sites
  --name=ROOT_PATH_MASK
                        only matches by root_path
  -m MATCH_TESTS, --match-test=MATCH_TESTS
                        only matches if command output not empty string
                        commands from/usr/local/bin/drupal-tests/ allowed
                        multiple tests You can use negative matchers, add "-"
                        before command
  --test                Test exec, only first site
  --theme=THEME         Only for sites with theme
```

## Команда:
Команда будет запущена в каждой папке отфильтрованного сайта.  
Можно указать путь к скрипту, а можно написать bash строку, она будет выполнена.  
Чтобы не экранировать строку команды, можно писать ее в одинарных кавычках.  
`drall "echo \$(pwd)"` - правильно  
`drall 'echo $(pwd)'` - правильно  
`drall "echo $(pwd)"` - неправильно, в каждой папке выдаст путь папки, из которой был запущен drall  


## Параметры:
`-q` Не выводить путь к папке сайта перед выполнением команды  
`-v` Подробный вывод работы скрипта  
`-r`, `--asroot` Выполнить команду под root (иначе выполнится от имени владельца папки)  
`-n` Применить только к первым n сайтам  
`-m` --match-test Фильтровать сайты командой  
`--theme` Применять только к сайтам с определенной темой  


## Фильтрация сайтов:
`-m` - главная команда для фильтрации. Работает так:  
В `/usr/local/bin/drupal-tests` ищется одноименный файл и выполняется в корне сайта.  
Если результат не пустой и не равен нулю, сайт проходит фильтр.  
Можно указывать несколько тестов так:  
`drall -m test1 -m test2`  
  
Можно использовать отрицательные тесты, нужно добавить "-" перед командой.  
Например, вывести сайты с "грязным" css:  
`drall -m visitkaplus -m -clean_css`  
  
`--theme` - принимает имя темы, проверяет, лежит ли в sites/all/themes такая папка. После появления match-test не особо полезен.  
  
`-n` - ограничивает выходной список указанным количеством  
  
`--name` - ограничивает выходной список сайтами, содержащими в пути указанную подстроку. Удобно, если нужно проверить drall на конкретном одном сайте  



## Примеры использования:
`drall` - просто выведет список всех друпалов  
`drall -m snormal` - выведет список всех snormal  
`drall -m visitkaplus -m clean_css` - получение списка всех визиток с чистым css  
`drall 'drush dis dblog'` - отключит dblog на всех сайтах  
`drall 'drush ublk user@example.com'` - а также другие drush-команды  
`drall 'drupal-patch http://drupal.org/path/to/patch'` - применение патча ко всем сайтам
`drall -m visitkaplus 'drupal-patch http://example.com/style.css_menu_width.patch'` - пропатчить все готовые не измененные сайты  
`drall -m '-drs vget update_modified' -m visitkaplus` - получить все визитки+ не отключенные от апдейтов  
`drall -v -m all_complete -m clean_all` - получить все готовые не измененные сайты  
`drall --asroot -m snormal -m clean_css -v 'cp-chown /home/from/www/from.example.com $PWD sites/all/themes/theme/css/style.css'` - скопировать style.css из from.example.com в каждый неизмененный snormal  


## Известные проблемы:
1. Команду нужно заключать в кавычки, фактически, команда должна быть одним параметром скрипта
2. Если в папке тестов drupal-tests будет несколько одноименных тестов, команда скорее всего упадет
