### Создание тестового приложения
1. В качестве тестового приложения выбрал NGINX,который по средствам докера подтягивает данные для статического контента с файла index.html
### Установка и настройка CI/CD
В качестве CI/CD инструмента был собран Jenkins.
 - В качестве Workspace где будет отрабатыватся тестовое приложение выбран node1
 - На node1 установлен docker,создан пользователь с необходимыми правами,проброшены ssh ключи от K8S Jenkins,склонирован репозиторий
 - создан следующий скрипт, который выполняет основное требование к тестовому приложению
 ```
 jenkins@node1:~$ cat jenkins_script.sh 
#!/bin/bash
cd netology_diplom
git pull origin main
vers=`sed -n 7p ~/mydiplom/values.yaml | grep -o 1.**`
newvers=$(bc<<<"scale=4;$vers+0.01")
echo $newvers
sed -i 's/'$vers'/'$newvers'/g' ~/mydiplom/values.yaml
sed -n 7p ~/mydiplom/values.yaml
docker build -t f6987c8d6ed5/diplom_netology:$newvers ~/netology_diplom 
docker push f6987c8d6ed5/diplom_netology:$newvers
docker rmi f6987c8d6ed5/diplom_netology:$vers
unset vers
unset newvers
```
 - В Jenkins создана "задача со свободной конфигурацией" с конфигурацией, представленной ниже на слайдах
 - К Jenkins подключен git репозиторий по ssh
![](https://github.com/vk1391/devops-netology/blob/9e44361c9d0bfe608910724ae57447de734ca03a/jenkins1.jpg)
 - Триггер настроен следующим образом:Jenkins раз в минуту проверяет изменения в git репозитории.Если они имеются,то запускается сборка
![](https://github.com/vk1391/devops-netology/blob/9e44361c9d0bfe608910724ae57447de734ca03a/jenkins2.jpg)
- При срабатывании триггера Jenkins запускает скрипт на node1(192.168.15.15 адрес node1 за натом),обновляет helm chart рабочего приложения,запускает проверку приложения с задержкой в 10 секунд для того что бы в успел обновиться деплой
![](https://github.com/vk1391/devops-netology/blob/9e44361c9d0bfe608910724ae57447de734ca03a/jenkins3.jpg)
