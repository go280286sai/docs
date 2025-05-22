### Основные команды Git
````
git init               # создать репозиторий
git clone <url>        # склонировать удалённый репозиторий
git status             # текущее состояние
git add <file>         # добавить файл в индекс (staging)
git commit -m "msg"    # сохранить коммит
git log                # история коммитов
git branch             # список веток
git checkout -b new    # создать и перейти в ветку
git switch new         # перейти в ветку (современный способ)
git merge branch       # слить ветку в текущую
git rebase branch      # "переписать" историю поверх ветки
git remote add origin <url>    # связать с удалённым репо
git push -u origin main        # отправить ветку
git pull                       # получить и объединить
git fetch                      # получить без объединения
git remote add origin <url>    # связать с удалённым репо
git push -u origin main        # отправить ветку
git pull                       # получить и объединить
git fetch                      # получить без объединения
git reset --soft HEAD~1     # откатить последний коммит, сохранить изменения
git reset --hard HEAD~1     # откатить и удалить изменения
git checkout <commit>       # перейти на коммит
git revert <commit>         # создать "обратный" коммит
git diff <commit>           # сравнить с коммитом
git diff HEAD~1             # сравнить с предыдущим коммитом
git diff HEAD~1..HEAD       # сравнить с предыдущим и текущим коммитом
git diff HEAD~1..HEAD~2     # сравнить с предыдущим и предыдущим коммитом
git stash             # временно сохранить изменения
git stash pop         # вернуть сохранённое
git tag v1.0           # создать тег
git push origin v1.0   # отправить тег
````
#### git cherry-pick
Применяет один конкретный коммит из одной ветки в другую.
````
git checkout feature
git cherry-pick a1b2c3d
````

#### git bisect
Git перебирает коммиты между ними, спрашивает, хорош ли коммит → тыкаешь git bisect good или git bisect bad.
````
git bisect start
git bisect good <commit>
git bisect bad <commit>
git bisect reset
````
#### git reflog
Показывает всю историю перемещений HEAD, даже те коммиты, которые "потерялись" после reset, rebase и т.д.
````
git reflog
```