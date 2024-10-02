# Вот пошаговое руководство по выполнению всех этапов вашего задания, включая создание docker-compose.yml, развертывание стека, тестирование отказоустойчивости и другие запросы.

Шаг 1: Подготовка окружения
Создайте файл docker-compose.yml с следующим содержимым:

yaml
Копировать код
```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    deploy:
      replicas: 3
      placement:
        constraints:
          - node.role == worker
    ports:
      - "80:80"

  db:
    image: postgres:latest
    environment:
      POSTGRES_USER: example
      POSTGRES_PASSWORD: example
      POSTGRES_DB: example_db
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.hostname == <ваше_имя_ноды>  # Укажите имя ноды, где хотите развернуть БД
```

Замените <ваше_имя_ноды> на имя конкретной ноды, где вы хотите, чтобы база данных развернулась.
Шаг 2: Развертывание стека
Запустите Docker Swarm на главном менеджере:

```bash
docker swarm init
```

Затем разверните стек с помощью команды:

```bash
docker stack deploy -c docker-compose.yml my_stack
```

Шаг 3: Тестирование отказоустойчивости
Отключите одну из нод с веб-сервером (например, 192.168.xx.12):

```bash
ssh <user>@192.168.xx.12
sudo shutdown now
```

Проверьте доступность веб-сервиса по IP отключенной ноды:

Откройте браузер и перейдите по адресу <http://192.168.xx.12>.

Зафиксируйте поведение системы, вы можете использовать команду:

```bash
docker service ls
```

Шаг 4: Закрепление базы данных на конкретной ноде
Убедитесь, что в конфигурации docker-compose.yml указано имя ноды, на которой должна работать база данных, как показано выше.

Шаг 5: Масштабирование веб-сервера
Для увеличения количества реплик веб-сервера до пяти выполните команду:

```bash
docker service scale my_stack_web=5
```

Шаг 6: Мониторинг состояния
Для отображения текущего состояния стека и каждого сервиса используйте команды:

```bash
docker stack ps my_stack
docker service ls
```

Подсказки
Логи: Для просмотра логов сервиса используйте команду:

```bash
docker service logs my_stack_web
```

Доступ к контейнерам: Чтобы подключиться к конкретному контейнеру, выполните:

```bash
docker exec -it <container_id> /bin/*bash*
```