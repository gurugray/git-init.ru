---
title: "Git это просто"

author: Ben Straub
authorLink: https://github.com/ben
authorAvatar: https://gravatar.com/avatar/0d410939c9e80980cf0430772567dfb4?r=x&s=160
authorGithub: ben
authorTwitter: benstraub

origin: http://jaxenter.com/git-is-simple-on-the-inside-49426.html
originTitle: "Git is simple (on the inside)"

category: translation
permalink: /import/git-is-simple.html

layout: translate
---
> Иногда нам действительно нужно знать как это работает.

В основном мы используем компьютеры с помощью метафор.

Когда вы используете текстовый редактор, вы не думаете о байтовом представлении каждого символа, или об управляющих последовательностях, которые определяют какие слова имеют курсивное или полужирное начертание.
Программа избавляет вас от этого, предоставляя понятную метафору — нарисованный символ на листе бумаги.
Вы пишете слова, определяетесь со шрифтом и стилем для них, и когда вы готовы к тому, чтобы ваши слова попали на *реальную* бумагу — отправляете их на принтер.

Когда вы регулируете экспозицию фотографии, вы не думаете о математических выкрутасах необходимых для смены R, G, B каналов для каждого пикселя, или алгоритмов пересчёта при смене размера или поворота.
Тут метафора для вас это комната проявки, вы просто регулируете экспозицию и яркость, убираете «красные глаза» и стираете пятно от вина на подвенечном платье.

Со многими системами контроля версий вы тоже не хотите знать как данные хранятся и извлекаются, или как сортируются байты при передаче по сети.
Вы просто хотите писать свой код, каждый раз сохраняя «снимок» _(англ. snapshot)_ в безопасное место, и метафора обозначена на таком уровне.
Нижележащая структура данных сложна и вам редко (если вообще кода-нибудь) понадобиться знать насколько, потому что пользовательский интерфейс довольно эффективно скрывает эти детали на своём уровне абстракции.

С Git'ом немного другая история.
Метафора представляет из себя направленный ацикличный граф _(англ. directed acyclic graph, DAG)_ с коммитами в качестве его узлов, что говорит нам о том, что нет никакой метафоры — модель данных на самом деле граф.
Если вы попробуете использовать метафоры других систем, то вам не избежать неприятностей.

Хорошая новость в том, что эта модель данных легка для понимания и выяснив это вы улучшите свои навыки в использовании Git'а.

## Объекты

Практически всё в репозитории Git'а либо **объект** либо **ссылки** _(анг. reference, ref)_.

Объекты, это то в чём Git хранит содержимое.
Они хранятся в `.git/objects`, директории, которая иногда называется *объектной базой данных*.
Объекты в ней неизменны, однажды создав их, вы не можете их поменять.
Это потому что Git использует SHA-1 хэш от их содержимого для их идентификации и поиска, и если вы поменяете содержимое объекта — его хэш изменится.

Объекты представлены в четырёх видах: blob'ы _(англ. blobs)_, деревья, коммиты и аннотированные теги.
**Blobs'ы** это куски данных, которые Git никаким образом не интерпретирует, это то как Git хранит ваши файлы.
Объекты легко проинспектировать:

```
# вывести тип объекта
$ git cat-file -t d7abd6
blob

# Вывести первые 5 строк содержимого объекта
$ git cat-file -p d7abd6 | head -n 5
<!DOCTYPE html>
<!--[if IEMobile 7 ]><html class="no-js iem7"><![endif]-->
<!--[if lt IE 9]><html class="no-js lte-ie8"><![endif]-->
<!--[if (gt IE 8)|(gt IEMobile 7)|!(IEMobile)|!(IE)]><!--><html class="no-js" lang="en"><!--<![endif]-->
<head>
```

Обратите внимания, что тут нет имени файла.
Git ожидает, что операция переименование файлов достаточно частая и, если бы имена файлов были объедены с содержимым, вам бы пришлось хранить много копий объектов, разница между которыми была бы лишь в имени файла.

Вы использовали `git cat-file` потому что Git оптимизирует хранение объектов.
Они сжимаются gzip'ом, и иногда объединяются в большие упакованные файлы _(англ. pack-files)_ и если вы посмотрите в `.git/objects`, вы можете не увидеть ничего, что смогли бы назвать объектом.

Второй тип объектов называется **дерево** _(англ. tree)_ и это то как Git хранит структуру директорий проекта.

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

Только лишь blob'ы не структурированы, Git ожидает довольно строгого формата от всех остальных объектов.
Каждая строка объекта «дерево» содержит флаги доступа к файлу, какой это тип (`blob` это файл, `tree` это поддиректории), SHA-1 хэш от объекта и имя файла.
То есть объект «дерево» отвечает за имена и расположение разных вещей, а blob'ы отвечают за их содержимое.

Третий тип объектов это **коммит** _(англ. commit)_.
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

У коммита есть только одна ссылка на объект дерева, которое является описанием корневой директории коммита.
Он имеет несколько, или не имеет вовсе, родителей, записи о которых лишь ссылки на другие коммиты и он содержит некоторую мета-информацию о самом коммите — кто его создал, когда и о чём он.

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

Я расскажу как это работает позже, сейчас же отметьте SHA сумму, которая тут хранится.

Это всё!
Вы можете пересчитать типы объектов на пальцах одной руки!
Видите насколько это просто?

## Ссылки

Ссылки  (или *ref'ы*) не более чем указатели на объекты или другие ссылки.
Информация в них состоит из двух частей: имени ссылки и куда она указывает.
Если ссылка указывает на объект, она называется *прямой ссылкой*, если она указывает на другую ссылку, то *символической ссылкой*.

В основном ссылки прямые.
Что бы это подтвердить проверим содержимое чего либо в директории `.git/refs/heads`, всё что там лежит — текстовые файлы, их содержимое это SHA хэши коммитов на которые они указывают.

```
$ cat .git/refs/heads/master
2b67270f960563c55dd6c66495517bccc4f7fb17
```

Git хранит и несколько символических ссылок для специальных целей.
Самая используемая это `HEAD`, указывающая на ветку с которой вы сейчас работаете:

```
$ cat HEAD
ref: refs/heads/master
```

Теперь мы знаем как работают ссылки, давайте ещё раз взглянем на объект аннотированного тега, который мы видели ранее.
Помните, что ссылки это просто имена для обозначения расположения, для них нет комментариев и вы можете изменять их в любое время.
Аннотированные теги разрешают эти задачи складывая информацию о ссылке в объектную базу (делая их неизменяемыми и расширяя их дополнительным содержанием) затем делают их доступными, присоединяя обычный тег к ним.
Вся схема выглядит примерно так:

```
tag (ref)  →  tag_ann (odb)  →  commit
```

Омечу, что это открывает нам целую вселенную новых возможностей: ссылки не обязаны ссылаться на *коммиты*.
Они могут указывать на *любой* тип объектов, что означает вы технически может создать что-то вроде этого (правда не ясно зачем вам может такое понадобиться):

```
branch  →  tag  →  tag_ann_a  →  tag_ann_b  →  blob
```

## Three Trees

Tree-type objects in the ODB aren't the only tree that Git likes to think about.
During your day-to-day work, you'll deal with three of them: HEAD, the index, and the work tree.

**HEAD** is the last commit that was made, and it's the parent of your next commit.
Technically, it's a symbolic ref that points to the branch you're on, which points to the last commit, but for the purposes of this section we'll simplify a bit.

The **index** is a proposal for the next commit.
When you do a checkout, Git copies the HEAD tree to the index, and when you type `git commit -m 'foo'`, it's going to take what's in the index and store it in the ODB as the new commit's tree.

The **work tree** is a sandbox.
You can safely create, update, and delete files at will in this area, knowing that Git has backups of everything.
This is how Git makes your content available to the rest of the universe of tools.

There are a few commands whose job is mainly to deal with these three trees.

* `git checkout` – Copies content from HEAD into the index and work tree.
  It can also move HEAD first.
* `git add` – Copies content from the work tree to the index.
* `git commit` – Copies content from the index to HEAD.



## Reset is Easier Now

Now that we know a few things, some of Git's seemingly-strange commands start to make sense.
One example is `git reset`, one of the most hated and feared commands in all of Git.
Reset generally performs three steps:

1. Move HEAD (and the branch it points to) to point to a different commit
1. Update the index to have the contents from HEAD
1. Update the work tree to have the contents from the index

And, through some oddly-named command-line options, you can choose where it stops.

* `git reset --soft` will stop after step 1.
  HEAD and the checked-out branch are moved, but that's all.
* `git reset --mixed` stops after step 2.
  The work tree isn't touched at all, but HEAD and the index are.
  This is actually reset's default; the `--mixed` argument is optional.
* `git reset --hard` performs all three steps.
  After the first two, the work tree is overwritten with what's in the index.

If you use reset with a path, Git actually skips the first step, since moving HEAD is a whole-repository operation.
The other two steps apply, though; `--mixed` will update the index with HEAD's version of the file, and `--hard` also updates the working directory, effectively trashing any modifications you've made to the file since it was checked out.


## The Everyday Pattern

Let's take a look at a really common usage pattern with our new Git X-ray goggles on.

```
$ git checkout -b bug-2345 master
```

Git creates a new branch called `bug-2345`, and points it at the same commit `master` points to. Then it moves HEAD to point to `bug-2345`, and updates the index and work tree to match HEAD.

You do some work, changing the files in the work tree, and now you're ready to make a commit.

```
$ git add foo.txt
$ git add -p bar.html
```

Git updates the index to match the contents of the work tree.
You can even update it with only *some* of the changes from a file.

```
$ git commit -m 'Update foo and bar'
```

Git converts the index into a series of linked objects in the ODB. Blobs and trees whose contents match are re-used, and the files and directories that changed have new blobs and trees generated for them.
Git then creates a new commit which points to the new root tree, and (since HEAD points to a branch) `bug-2345` is moved to point to the new commit.


## Go Forth

With most version-control systems, you're encouraged to just get to know the UI layer, and the details will be safely abstracted away.
Git is different; its basic data model is at a high enough level that it's pretty easy to understand, and its UI layer is so thin, that you'll find yourself learning the internals whether you want to or not – you'll need to for everything except the bare minimum of usage.
I hope this article has convinced you that there isn't really that much to know, and that you earn many new abilities by going through this process.

Your new understanding has made you powerful.
Please use your new powers for good.
