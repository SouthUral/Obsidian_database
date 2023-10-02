[[Git]]
Для того чтобы работать над проектом в нескольких репозиториях, например, работа в домашнем репозитории и в рабочем, если нет доступа к рабочему репозиторию вне организации, можно добавить репозиторий через консоль:
```shell
git remote add name_repository https://github.com/username/
```

ссылка `https://github.com/username/` берется из git clone

Команда для проверки, был ли добавлен репозиторий:
```shell
git remote -v
```

Пример вывода:
```shell
git_lab git@spbgit.polymetal.ru:polyna/algorithm/shoveltrucksarrival_v2.git (fetch)
git_lab git@spbgit.polymetal.ru:polyna/algorithm/shoveltrucksarrival_v2.git (push)
origin  git@github.com:SouthUral/shovelTrucksArrival.git (fetch)
origin  git@github.com:SouthUral/shovelTrucksArrival.git (push)
```

Теперь можно закидывать изменения в оба репозитория, работая, по факту,  над одним проектом.
```shell
git push origin develop
git push git_lab develop
```