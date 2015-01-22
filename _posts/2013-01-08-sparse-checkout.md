---
title: "sparse checkout"
date: 2013-01-08
tags: tips, external
category: post

layout: post
---

Странно, но я мало встречаю упоминаний этой возможности в различных заметках и документации. Однако **частичный _(sparse)_ чекаут** часто необходим пользователям переходящим с svn и использующим svn externals в своих проектах.

В git архитектуре заложено, что локально клонируется весь удалённый репозиторий, но пользователям svn externals часто нужна лишь часть проекта. С версии 1.7 в git доступна схожая функциональность, которую назвали [sparse checkout](http://www.kernel.org/pub/software/scm/git/docs/git-read-tree.html#_sparse_checkout), с её помощью вы можете оставить в своей рабочей копии проекта лишь те файлы и директории, которые вам необходимы. 

Вот как это работает:<ol>
    <li>клонируем весь репозиторий
        <pre><code>git clone &lt;repo_url&gt; &lt;directory&gt;</code></pre>
    </li>
    <li>переходим в директорию проекта
        <pre><code>cd &lt;directory&gt;</code></pre>
    </li>
    <li>включаем опцию частичного чекаута
        <pre><code>git config core.sparsecheckout true</code></pre>
    </li>
    <li>добавляем директории и файлы, которые хотим видеть в подключённом проекте (в моём случае я хочу отфильтровать не только директории, но и конткретные технологии, например только файлы JavaScript)
        <pre><code>echo "/blocks-desktop/**/*.js" &gt; .git/info/sparse-checkout</code></pre>
    </li>
    <li>перечитываем рабочую копию проекта
        <pre><code>git read-tree -m -u HEAD</code></pre>
    </li>
</ol>

Видно, что синтаксис файла `.git/info/sparse-checkout` аналогичен синтаксису `.gitignore`, но объявляющий *наличие* попадающих под маску сущностей, а следовательно чуть более гибкий, чем svn externals.
