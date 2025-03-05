# Установка и настройка PostgreSQL в контейнере Docker
* Устанавливаем Ubuntu в Windows Subsystem Linux (WSL)
  ```
  wsl --install Ubuntu
  ```
  ![image](https://github.com/user-attachments/assets/9a84ab1e-1c02-4e53-a5a6-3376d6a2630d)
* Открываем терминал WSL
  
  ![image](https://github.com/user-attachments/assets/c5b91453-7272-4fba-ae59-3b9ee9b5352d)
* Устанавливаем Docker
  * Обновляем компоненты системы
  ```
  sudo apt update && sudo apt upgrade -y
  ```
  * Устанавливаем зависимости для Docker
  ```
  sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
  ```
  * Добавляем GPG ключ Docker
  ```
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
  ```
  * Добавляем репозиторий Docker
  ```
  echo "deb [signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  ```
  * Обновляем репозиторий и устанавливаем Docker Engine
  ```
  sudo apt update
  sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
  ```
  * Добавляем пользователя в Docker Group
  ```
  sudo usermod -aG docker student
  ```
  * Проверяем, установился ли Docker
  ```
  docker --version
  ```
  ![image](https://github.com/user-attachments/assets/21050efc-a5be-4e3a-8d81-912da2a88c5f)
* Создаем каталог /var/lib/postgres
  ```
  mkdir /var/lib/postgres
  ```
  ![image](https://github.com/user-attachments/assets/18ba54a2-ba7f-4299-b59f-d65712af8bb6)
* Скачиваем образ Postgres 15
  ```
  docker pull postgres:15
  ```
* Проверяем образы docker
  ```
  docker images
  ```
  ![image](https://github.com/user-attachments/assets/c67decf4-ddca-422b-bc66-72058466bbb8)
* Создаем docker-сеть
  ```
  sudo docker network create pg-net
  ```
* Разворачиваем контейнер с PostgreSQL 15, смонтировав в него /var/lib/postgresql
  ```
  sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:15
  ```
* Открываем второй терминал WSL и запускаем отдельный контейнер с клиентом PostgreSQL в общей сети
  ```
  sudo docker run -it --rm --network pg-net --name pg-client postgres:15 psql -h pg-server -U postgres
  ```
* В первом терминале проверяем что мы подключились к БД Postgres через отдельный контейнер
  ![image](https://github.com/user-attachments/assets/4cbfdf37-5313-4e70-8a11-c2e8e83f8894)
* В контейнере с клиентом Postgres проверяем, какие базы есть на текущий момент
  ![image](https://github.com/user-attachments/assets/bc8bd496-8021-425f-aa70-fbfdeae90f05)
* Создаем новую базу данных otus и подсоединяемся к ней
  ![image](https://github.com/user-attachments/assets/55712d5a-ddcd-4ce5-82da-913d489b45c8)
* Создаем таблицу books и заполняем парой строк
  ![image](https://github.com/user-attachments/assets/ee3bd2d1-c94c-41f5-8817-2ab012168a8b)
  
  Проверяем, есть ли записи в таблице books
  
  ![image](https://github.com/user-attachments/assets/e2ae9ca5-c200-4fa1-b2e1-e308adbf3aa9)
* Смотрим имя контейнера с сервером Postgres
    ![image](https://github.com/user-attachments/assets/eed5e114-da36-4a7b-ae34-8d52426ecbeb)

    Останавливаем контейнер и удаляем его
    ```
    docker stop 7b926dbb0e0f
    docker rm 7b926dbb0e0f
    ```
    Запущенных контейнеров нет
  
    ![image](https://github.com/user-attachments/assets/362e316f-96a5-4f66-8b70-852cba1401a6)
* Снова создаем контейнер с сервером PostgreSQL 15, смонтировав в него /var/lib/postgresql
  ```
  sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:15
  ```
  Подключаемся из контейнера с клиентом Postgres к нашему серверу
  ```
  sudo docker run -it --rm --network pg-net --name pg-client postgres:15 psql -h pg-server -U postgres
  ```
* Получаем список баз данных
  ![image](https://github.com/user-attachments/assets/b65dce85-808c-433c-82b6-b32384dd412f)

  Снова подключаемся к базе otus и смотрим, есть ли таблица books и есть ли в ней записи
  ![image](https://github.com/user-attachments/assets/6b7d3d67-8c77-4446-a91a-0b332dea9341)

  Данные остались на месте.

  

