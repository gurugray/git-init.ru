---
title: "Config «insteadOf»"
date: 2013-01-14
tags: post
permalink: "2013/post/instead-of.html"

layout: layouts/post.njk
---

Многие, _даже начинающие_, пользователи git знают про его возможность настраивать алиасы на команды. Но почему-то не все, _даже более-менее продвинутые_, не знают или забывают про «алиасы» на удалённые репозитории в [.gitconfig](http://git-scm.com/docs/git-config).

Подобная конфигурация может помочь вам при использовании длинных URL к вашим репозиториям, например такой конфиг:

<pre>
vi .gitconfig
…
[url "git@my-enterprize-github-install.example.ru:"]
insteadOf = "ght:"
…
</pre>

Позволит оперировать вашим удалённым репозиторием гораздо проще, например:

<pre>
git clone ght:gurugray/talks.git
…
git remote add ght:partner/talks.git
…
</pre>

Однако и это не всё. Если ваш удалённый сервер позволяет стягивать с себя код анонимно, _например только для чтения_, вы можете ускорить процесс получение новых изменений не используя `ssh` протокол для этого, тем самым немного «разгрузив» сервер, освобождая его от дополнительной работы.

Расширим наш предыдущий конфиг таким образом:

<pre>
vi .gitconfig
…
[url "git@my-enterprize-github-install.example.ru:"]
pushInsteadOf = "ght:"
pushInsteadOf = "git://my-enterprize-github-install.example.ru/"

[url "git://my-enterprize-github-install.example.ru/"]
insteadOf = "ght:"
…
</pre>

Теперь оперируя с удалёнными репозиториями через `ght:` мы можем на самом деле работать с разными протоколами:

<pre>
git clone ght:gurugray/talks.git
…
git remote -v

origin git://my-enterprize-github-install.example.ru/gurugray/talks.git (fetch)
origin git@my-enterprize-github-install.example.ru:gurugray/talks.git (push)
…
</pre>

Дальше всё зависит от вашей фантазии, например можно настроить подобный «алиас» таким образом, что бы забирать изменения из одного репозитория, а выливать в другой, при этом оперируя одним и тем же URL.
