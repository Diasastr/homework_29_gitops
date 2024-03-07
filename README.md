Налаштування середовища для Kubernetes і ArgoCD на Windows
==========================================================

Цей репозиторій містить інструкції та конфігураційні файли для налаштування середовища розробки з Kubernetes і ArgoCD на Windows, використовуючи Minikube та kubectl для розгортання веб-додатку який було скопійовано з іншої репо (це fork від джерела коду вебзастосунка).

Передумови
----------

Переконайтеся, що у вас встановлено Chocolatey на вашій машині з Windows. І Docker (запущений).

Інсталяція необхідного ПЗ
-------------------------

### Встановлення Minikube і kubectl

Відкрийте PowerShell як адміністратор та виконайте наступні команди:

```powershell

# Встановлення Minikube і встановлення kubectl
choco install minikube
```

### Встановлення ArgoCD CLI

```powershell
# Встановлення Argo-CLI
choco install argo-cli
```
### Запуск Minikube

```powershell
minikube start --driver=docker
```

### Встановлення ArgoCD у Minikube

```powershell
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Налаштування доступу до ArgoCD UI

```powershell
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

### Доступ до ArgoCD через локальний хост

Після встановлення ArgoCD та налаштування вашого Kubernetes середовища, ви можете отримати доступ до ArgoCD UI, використовуючи проброс портів. Це дозволить вам взаємодіяти з ArgoCD через веб-інтерфейс.

1.  Проброс портів для доступу до ArgoCD UI:

    Відкрийте нове вікно або вкладку терміналу та виконайте наступну команду:

    ```bash
    kubectl port-forward svc/argocd-server -n argocd 8080:443
    ```

    Ця команда пробросить порт 443 (HTTPS) сервісу `argocd-server` у вашому локальному оточенні на порт 8080.

2.  Вхід до ArgoCD UI:

    Після успішного пробросу портів, відкрийте веб-браузер та перейдіть за адресою http://localhost:8080. Ви побачите сторінку входу ArgoCD.

    Використовуйте наступні дані для входу:

    -   Ім'я користувача: `admin`

    -   Пароль: виконайте наступну команду в PowerShell, щоб отримати початковий пароль для користувача `admin`:

        ```powershell
        kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | ForEach-Object { [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($_)) }
        ```

    Ця команда виведе пароль, який ви використаєте для входу в систему.

Після входу ви матимете можливість управляти своїми застосунками через ArgoCD UI, включаючи створення нових застосунків, синхронізацію з Git репозиторіями, та моніторинг стану розгортань.

### Документація для Docker-файлів та Docker Compose

Ця документація описує процес створення Docker-файлів для двох частин нашого застосунку: Angular фронтенду та Java бекенду з використанням Spring Boot, а також налаштування Docker Compose для запуску цих сервісів разом із базою даних H2.

#### Структура проєкту

```csharp

проект/
├── angular/
│   ├── Dockerfile
│   ├── e2e/
│   ├── src/
│   ├── .editorconfig
│   ├── angular.json
│   ├── package.json
│   └── ...
└── java/
    ├── basic-web-app-java/
    ├── digitalcrafting-base/
    └── Dockerfile
docker-compose.yml
README.md
```

#### Dockerfile для Angular

Файл `проект/angular/Dockerfile` використовується для створення Docker-образу Angular застосунку.

```Dockerfile
# Stage 1: Build the Angular application
FROM node:14 as build-step

WORKDIR /app

# Copy the package.json file and install dependencies
COPY package.json ./
RUN npm install

# Copy the rest of the Angular application files
COPY . .

# Build the project
RUN npm run build

# Stage 2: Serve the application with Nginx
FROM nginx:alpine

# Copy the build output to replace the default Nginx contents
COPY --from=build-step /app/dist/angular /usr/share/nginx/html

# Expose port 80
EXPOSE 80

# Start Nginx and serve the application
CMD ["nginx", "-g", "daemon off;"]
````

#### Dockerfile для Java

Файл `проект/java/Dockerfile` використовується для збірки Java-проєктів та створення Docker-образу Spring Boot застосунку.

```Dockerfile
# Use Maven to build the project
FROM maven:3.6.3-openjdk-11 as build

# Copy source code and pom files
COPY digitalcrafting-base/pom.xml /build/digitalcrafting-base/pom.xml
COPY basic-web-app-java/pom.xml /build/basic-web-app-java/pom.xml

# Copy the source code of digitalcrafting-base and install it
COPY digitalcrafting-base/src /build/digitalcrafting-base/src
WORKDIR /build/digitalcrafting-base
RUN mvn install

# Copy the source code of basic-web-app-java and package it
COPY basic-web-app-java/src /build/basic-web-app-java/src
WORKDIR /build/basic-web-app-java
RUN mvn package

# Final stage: Create the Docker container with Java runtime
FROM openjdk:11-jre-slim
WORKDIR /app

# Copy the JAR file from the build stage
COPY --from=build /build/basic-web-app-java/target/*.jar /app/app.jar

# Expose the port on which the web app will run
EXPOSE 8080

# Run the jar file
CMD ["java", "-jar", "app.jar"]
```

#### Docker Compose

Файл `docker-compose.yml`, що розташований в кореневій директорії проєкту, визначає та конфігурує всі сервіси, необхідні для запуску застосунку.

```yaml

`version: '3.8'

services:
  # H2 Database Service
  h2database:
    image: oscarfonts/h2
    ports:
      - "1521:1521" # Expose the H2 TCP server port
      - "81:81"     # Expose the H2 Console port
    volumes:
      - h2_data:/opt/h2-data
    environment:
      - H2_OPTIONS=-ifNotExists
      - WEB_ALLOW_OTHERS=true
      - WEB_PORT=81

  # Angular Frontend Service
  angular-frontend:
    build:
      context: ./angular
      dockerfile: Dockerfile
    ports:
      - "80:80"

  # Spring Boot Application Service
  spring-boot-app:
    build:
      context: ./java
      dockerfile: Dockerfile
    environment:
      - SPRING_DATASOURCE_URL=jdbc:h2:tcp://h2database:1521/~/test
      - SPRING_DATASOURCE_DRIVER_CLASS_NAME=org.h2.Driver
      - SPRING_DATASOURCE_USERNAME=sa
      - SPRING_DATASOURCE_PASSWORD=
    depends_on:
      - h2database
    ports:
      - "8080:8080"

volumes:
  h2_data:
  ````

#### Підключення до різних частин застосунку

-   Angular фронтенд: Відкрийте веб-браузер і перейдіть на `http://localhost`. Ви побачите інтерфейс Angular застосунку.
-   Spring Boot бекенд: Для доступу до API, запустіть HTTP-запит на `http://localhost:8080`.
-   Консоль бази даних H2: Для управління базою даних через веб-інтерфейс, перейдіть на `http://localhost:81`.

#### Примітки безпеки

У файлі `docker-compose.yml` ми використовуємо прості значення для імені користувача та пароля бази даних (наприклад, `SPRING_DATASOURCE_USERNAME=sa` і `SPRING_DATASOURCE_PASSWORD=`) для спрощення локального розгортання. Однак, для виробничого середовища необхідно використовувати безпечніші методи зберігання та управління секретами, такі як Docker Secrets або зовнішні менеджери секретів.
