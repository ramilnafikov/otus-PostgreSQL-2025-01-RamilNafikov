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
