---
title: 'Git — это просто'
isTranslate: true

author:
  name: Ben Straub
  site: https://github.com/ben
  avatar: https://gravatar.com/avatar/0d410939c9e80980cf0430772567dfb4?r=x&s=160
  github: ben
  twitter: benstraub

origin:
  link: http://jaxenter.com/git-is-simple-on-the-inside-49426.html
  title: 'Git is simple (on the inside)'

permalink: '__/git-is-simple.html'

layout: layouts/post.njk
---

> Иногда нам действительно нужно знать, как это работает.

В основном мы общаемся с компьютером с помощью метафор.

Когда вы используете текстовый редактор, вы не думаете о байтовом представлении каждого символа или об управляющих последовательностях, которые определяют, какие слова имеют курсивное или полужирное начертание.
Программа избавляет вас от этого, предоставляя понятную метафору — нарисованный символ на листе бумаги.
Вы пишете слова, определяетесь со шрифтом и стилем для них, и когда вы готовы к тому, чтобы ваши слова попали на _реальную_ бумагу — отправляете их на принтер.

Когда вы регулируете экспозицию фотографии, вы не думаете о математических выкрутасах, необходимых для смены R, G, B каналов для каждого пикселя, или алгоритмах пересчёта при смене размера или поворота.
Тут метафора для вас — это комната проявки, вы просто регулируете экспозицию и яркость, убираете «красные глаза» и стираете пятно от вина на подвенечном платье.

Со многими системами контроля версий вы тоже не хотите знать, как данные хранятся и извлекаются, или как сортируются байты при передаче по сети.
Вы просто хотите писать свой код, каждый раз сохраняя «снимок» _(англ. snapshot)_ в безопасное место, и метафора обозначена на этом уровне.
Внутренняя структура данных сложна, и вам редко (если вообще когда-нибудь) понадобится знать о ней, потому что пользовательский интерфейс довольно эффективно скрывает эти детали на своём уровне абстракции.

С Git'ом немного другая история.
Метафора представляет из себя направленный ацикличный граф _(англ. directed acyclic graph, DAG)_ с коммитами в качестве его узлов, что говорит нам о том, что нет никакой метафоры — модель данных на самом деле граф.
Если вы попробуете использовать метафоры других систем, то вам не избежать неприятностей.

Хорошая новость в том, что эта модель данных легка для понимания и, разобравшись в ней, вы улучшите свои навыки использования Git'а.

## Объекты

Практически всё в репозитории Git'а — либо **объект**, либо **ссылки** _(англ. reference, ref)_.

Объекты — это то, в чём Git хранит содержимое.
Они хранятся в `.git/objects`, директории, которая иногда называется _объектной базой данных_.
Объекты в ней неизменны, однажды создав их, вы не можете их изменить.
Это потому, что Git использует SHA-1 хэш от их содержимого для их идентификации и поиска, и если вы поменяете содержимое объекта — его хэш изменится.

Объекты представлены в четырёх видах: блобы _(англ. blobs)_, деревья, коммиты и аннотированные теги.
**Блобы** — это куски данных, которые Git не выделяет ни в какую структуру, это то, как Git хранит ваши файлы.
Объекты легко проинспектировать:

```
# Вывести тип объекта
$ git cat-file -t d7abd6
blob

# Вывести первые 5 строк содержимого объекта
$ git cat-file -p d7abd6 | head -n 5
<!DOCTYPE html>
<!--[if IEMobile 7 ]><html class="no-js iem7"><![endif]-->
<!--[if lt IE 9]><html class="no-js lte-ie8"><![endif]-->
<head>
  <body>
```

Обратите внимание, что тут нет имени файла.
Git ожидает, что операция переименования файлов достаточно частая и, если бы имена файлов были объединены с содержимым, вам бы пришлось хранить много копий объектов, разница между которыми была бы лишь в имени.

Вы использовали `git cat-file`, потому что Git оптимизирует хранение объектов.
Они сжимаются gzip'ом, и иногда объединяются в большие упакованные файлы _(англ. pack-files)_ и если вы посмотрите в `.git/objects`, вы можете не увидеть ничего, что смогли бы назвать объектом.

Второй тип объектов называется **дерево** _(англ. tree)_ — это то, как Git хранит структуру директорий проекта.

```
$ git cat-file -t 8f5b65
tree

$ git cat-file -p 8f5b65 | head -n 5
100644 blob 08b8e3400a81a79aeb42878171449b773ab493c0    after_footer.html
100644 blob 11517b315de6d7bc7550cc74ae413f1e6dafce19    archive_post.html
100644 blob 8ad5afd4581caa7458658325aeec9f8de875b988    article.html
040000 tree 5c2166adaa57c909182a45b995dfb750c22c8810    asides
040000 tree 52deb7c58d46aa09208c0b863fbecee81a2e3dad    custom
```

Git ожидает, что все объекты будут иметь довольно конкретный формат, кроме не структурированных блобов.
Каждая строка объекта-дерева содержит флаги доступа к файлу, какой это тип (`blob` это файл, `tree` это поддиректории), SHA-1 хэш от объекта и имя файла.
То есть объект-дерево отвечает за имена и расположение разных вещей, а блобы отвечают за их содержимое.

Третий тип объектов — это **коммит** _(англ. commit)_.
Это то, как Git описывает снимок в истории.

```
 $ git cat-file -t e365b1
commit

 $ git cat-file -p e365b1
tree 58c796e7717809c2ca2217fc5424fdebdbc121b1
parent d4291dfddfae86cfacec789133861098cebc67d4
author Ben Straub <bs@github.com> 1380719530 -0700
committer Ben Straub <bs@github.com> 1380719530 -0700

Fix typo, remove false statement
```

У коммита есть только одна ссылка на объект-дерево, которое является описанием корневой директории коммита.
Он имеет несколько, или не имеет вовсе, родителей, записи о которых — лишь ссылки на другие коммиты, и он содержит некоторую метаинформацию о самом коммите — кто его создал, когда, и о чём он.

Есть ещё один тип объектов, и он используется не часто.
Называется он **аннотированный тег** _(англ. tag annotation)_ и используется для создания тега с комментарием.

```
$ git cat-file -t 849a5e34a
tag

$ git cat-file -p 849a5e34a
object a65fedf39aefe402d3bb6e24df4d4f5fe4547750
type commit
tag hard_tag
tagger Ben Straub <bs@github.com> Fri May 11 11:47:58 2012 -0700

Tag on tag
```

Я расскажу, как это работает, позже, сейчас только отметьте SHA сумму, которая тут хранится.

Это всё!
Вы можете пересчитать типы объектов на пальцах одной руки!
Видите, насколько это просто?

## Ссылки

Ссылки (или _ref'ы_) — не более чем указатели на объекты или другие ссылки.
Информация в них состоит из двух частей: имени ссылки и куда она указывает.
Если ссылка указывает на объект, она называется _прямой ссылкой_, если она указывает на другую ссылку, то _символической ссылкой_.

В основном ссылки прямые.
Чтобы это подтвердить, проверим содержимое чего-либо в директории `.git/refs/heads`. Всё, что там лежит — текстовые файлы, их содержимое — это SHA хэши коммитов, на которые они указывают.

```
$ cat .git/refs/heads/master
2b67270f960563c55dd6c66495517bccc4f7fb17
```

Git хранит и несколько символических ссылок для специальных целей.
Самая используемая — это `HEAD`, обычно указывающая на ветку, с которой вы сейчас работаете:

```
$ cat HEAD
ref: refs/heads/master
```

Теперь мы знаем, как работают ссылки, давайте ещё раз взглянем на объект аннотированного тега, который мы видели ранее.
Помните, что ссылки — это просто имена для обозначения расположения, для них нет комментариев, и вы можете изменять их в любое время.
Аннотированные теги разрешают эти задачи, складывая информацию о ссылке в объектную базу (делая их неизменяемыми и расширяя их дополнительным содержанием), затем делают их доступными, присоединяя к ним обычный тег.
Вся схема выглядит примерно так:

```
tag (ref)  →  tag_ann (odb)  →  commit
```

Отмечу, что это открывает нам целую вселенную новых возможностей: ссылки не обязаны ссылаться на _коммиты_.
Они могут указывать на _любой_ тип объектов, что означает, вы технически можете создать что-то вроде этого (правда, не ясно, зачем вам может такое понадобиться):

```
branch  →  tag  →  tag_ann_a  →  tag_ann_b  →  blob
```

## Три дерева

Объекты-деревья в объектной базе это не все деревья с которыми работает Git.
Во время вашей ежедневной работы, вы имеете дело с тремя другими деревьями: HEAD, индекс и рабочая копия.

**HEAD** — это последний коммит, который был сделан, и он же родитель для следующего.
Технически это символьная ссылка, которая указывает на текущую ветку, которая, в свою очередь, указывает на последний коммит, но для целей этого раздела мы несколько упростим это понятие.

**Индекс** — это то, что предполагается для следующего коммита.
Когда вы делаете чекаут _(англ. checkout)_, Git копирует дерево HEAD в индекс, а когда вы набираете `git commit -m 'foo'`, всё то, что было в индексе, попадает в объектную базу как дерево нового коммита.

**Рабочая копия** — это песочница.
Вы спокойно можете создавать, обновлять и удалять тут файлы, зная что Git сохраняет архивные копии всего.
Именно тут Git делает доступными ваши изменения остальным инструментам.

Есть несколько команд, которые в основном работают с этими деревьями.

- `git checkout` – Копирует содержимое HEAD в индекс и в рабочую копию.
  Она также сначала двигает HEAD.
- `git add` – Копирует содержимое рабочей копии в индекс.
- `git commit` – Копирует содержимое индекса в HEAD.

## Reset теперь проще

Теперь, когда вы кое-что понимаете, немного странные команды Git'а приобретают смысл.
Например `git reset`, одна из наиболее ненавистных и страшных команд из всех.
Reset, как правило выполняет три шага:

1. Подвинуть HEAD (и ветку, на которую HEAD указывает) на другой коммит
1. Обновить содержимое индекса из HEAD
1. Обновить содержимое рабочей копии из индекса

И, с помощью некоторых странно названных параметров командной строки, вы можете выбрать на каком шаге остановиться.

- `git reset --soft` остановится на первом шаге.
  HEAD и текущая ветка подвинется, но на этом всё.
- `git reset --mixed` остановится на втором шаге.
  Рабочая копия не будет затронута, но HEAD и индекс изменятся.
  Это поведение reset'а по умолчанию, параметр `--mixed` опциональный.
- `git reset --hard` выполняет все шаги.
  После первых двух рабочая копия переписывается тем что в индексе.

Если вы используете reset с указанием пути в качестве параметра, Git автоматически пропускает первый шаг, потому что перестановка HEAD — это операция над всем репозиторием.
Остальные два шага работают так же: с `--mixed` обновляется индекс из HEAD'а, а с `--hard` также обновляется и рабочая копия, фактически отменяя любые модификации файлов, которые вы сделали после последнего чекаута.

## Ежедневное использование

Давайте теперь посмотрим на наболее частый случай работы с Git'ом, вооружённые новыми знаниями.

```
$ git checkout -b bug-2345 master
```

Git создаёт новую ветку с именем `bug-2345`, и она указывает на тот же коммит, на который смотрит `master`. После этого двигает HEAD на `bug-2345` и обновляет индекс и рабочую копию в соответствии с HEAD'ом.

Вы делаете какую-то работу, меняете файлы в рабочей копии и готовы сделать коммит.

```
$ git add foo.txt
$ git add -p bar.html
```

Git обновляет индекс контентом из рабочей копии.
Вы даже можете обновить его лишь _некоторыми_ изменениями файла.

```
$ git commit -m 'Update foo and bar'
```

Git преобразует индекс в серию связанных объектов в обектной базе. Блобы и деревья, содержимое которых совпадает — используются повторно, а для файлов и директорий, которые изменились — генерируются новые.
Затем Git создаёт новый коммит, который указывает на новое корневое дерево, и (так как HEAD указывает на ветку) ветка `bug-2345` подвигается на этот коммит.

## Двигайтесь дальше

В основном системы контроля версий поощряют лишь знания пользовательского интерфейса, а детали тщательно скрываются за абстракциями.
Git в этом отличается, его базовая модель данных на достаточно близком к пользователю уровне и её легко понять. Пользовательский интерфейс настолько тонок, что вы обнаружите себя изучающим внутренности, хотите вы того или нет. Эти знания помогут вам во всём, исключая тривиальные случаи.
Я надеюсь, что эта статья убедила вас в том, что тут не так много нужно знать, и вы получите больше возможностей, пройдя через этот процесс.

Ваши новые знания сделали вас могущественнее.
Пожалуйста, используйте вашу силу во имя добра.