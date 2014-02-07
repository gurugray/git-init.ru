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

## Objects

Just about everything in a Git repository is either an **object**  or a **ref**.

Objects are how Git stores content.
They're stored in `.git/objects`, which is sometimes called the *object database* or *ODB*.
Objects in the ODB are immutable; once you create one, you can't change it.
This is because Git uses an object's SHA-1 hash to identify and find it, and if you were to change the content of an object, its hash would change.

Objects come in four flavors: blobs, trees, commits, and tag annotations.
**Blobs** are chunks of data that Git doesn't interpret with any kind of structure, and it's how Git stores your files.
Objects are actually pretty easy to inspect:

```
# Print the object's type
$ git cat-file -t d7abd6
blob

# Print the first 5 lines of the object's content
$ git cat-file -p d7abd6 | head -n 5
<!DOCTYPE html>
<!--[if IEMobile 7 ]><html class="no-js iem7"><![endif]-->
<!--[if lt IE 9]><html class="no-js lte-ie8"><![endif]-->
<!--[if (gt IE 8)|(gt IEMobile 7)|!(IEMobile)|!(IE)]><!--><html class="no-js" lang="en"><!--<![endif]-->
<head>
```

Note that there's no filename here.
Git expects file renames to be fairly common, and if the filename were embedded with the content, you'd have to keep lots of copies of that object where the only difference is the filename.

You use `git cat-file` because Git has optimized the storage of objects.
They're gzip-compressed, and sometimes strings lots of them together into big pack-files, so if you look in `.git/objects`, you may not see anything you recognize as an object.

The second type of object is called a **tree**, and it's how Git stores directory structures.

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

Only blobs are unstructured; Git expects a fairly specific format for all the others.
Each line of a tree object contains the file's permission flags, what type it is (`blob`s are files, `tree`s are subdirectories), the SHA-1 hash of the object, and a filename.
So the tree type is responsible for the names and locations of things, and the blob type is responsible for their contents.

The third type of object is a **commit**.
This is how Git represents a snapshot in history.

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

A commit has exactly one tree reference, which is the root directory of the commit.
It has zero or more parents, which are references to other commits, and it has some metadata about the commit – who made it, when it was made, and what it's about.

There's just one more type of object, and it's not used very often.
It's called a **tag annotation**, and it's used to make a tag with comments.

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

I'll say more about how these work later; for now, just note the object SHA that's stored in there.

That's it!
You can count the number of object types on one hand!
See how simple this is?

## Refs

References (or *refs*) are nothing more than pointers to objects or other refs.
They consist of two pieces of information: the name of the ref, and where it points.
If a ref points to an object, it's called a *direct ref*; if it points to another ref, it's called a *symbolic ref*.

Most refs are direct.
To confirm this, check the contents of anything under `.git/refs/heads`; they're all plain-text files whose content is the SHA hash of the commit they point to.

```
$ cat .git/refs/heads/master
2b67270f960563c55dd6c66495517bccc4f7fb17
```

Git also keeps around a few symbolic refs for specific purposes.
The most commonly useful one is `HEAD`, which usually points to the branch you have checked out:

```
$ cat HEAD
ref: refs/heads/master
```

Now that we know how refs work, let's take another stab at that tag annotation object we saw earlier.
Remember that refs are basically just names for locations; there's no commentary associated with them, and you can change them at any time.
Tag annotations solve both of these issues by putting ref-like information into the ODB (making it immutable, and allowing it to have more content), then making it findable by attaching a regular tag ref to it.
The whole scheme looks like this:

```
tag (ref)  -->  tag_ann (odb)  -->  commit
```

Note that this opens up a whole new universe of possibility: refs don't have to point to *commits*.
They can point to *any* kind of object, which means you could technically set up something like this (though it's not clear why you'd want to):

```
branch --> tag --> tag_ann_a --> tag_ann_b --> blob
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
