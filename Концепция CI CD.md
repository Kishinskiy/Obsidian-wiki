
В компании по большому счету 4 типа проектов:

Бэкенды на Golang

Бэкенды на C# (.Net)

Шареные либы на C#

Фронтенды на Angular

Проекты необходимо собирать либо в кросс-платформенные артефакты, либо собирать под целевую платформу, на которой они будут запускаться.

При этом есть определенное количество вызовов:

Не везде разработка идет стандартизированно, например Golang репозиторий armaux не соответстует стандарту project-layot. В C# репозиториях вроде бы такой проблемы нет. Также нет стантартов использования тех или иных библиотек.

Артефакты разработки необходимо запускать на разных окружениях, где-то это будет Astra Linux, где-то другой дистрибутив.

И разработка, и DevOps имеют желание контейнеризировать все и переместить процессы в Kubernetes.

Необходимо поддерживать semver релизы и желательно - conventional коммиты.

Отсуствует инвенторизация текущих окружений - где какая ВМ, где что запущено.

Есть монорепозитории

Исходя из текущих данных, можно смело заявить, что CI часть пайплайнов можно настроить с легкой руки. CI/CD можно организовать в централизованном репозитории и договориться с разработчиками всей компании о единообразном процессе.

Контейнеризация выглядит несложной задачей, но есть очевидные риски: C# проект, протестированный в alpine контейнере, может повести себя аномально при запуске в виде systemd службы на Astra Linux.

Вывод № 1: Контейнеры + Kubernetes с огромной вероятностью сильно ускорить разработку продуктов, но если идти по этому пути, то имеет смысл сделать preprod окружения максимально подобными окружениям компаний-заказчиков. Таких окружений может быть несколько.
CD часть пайплайнов должна быть гибкой - нужно поддерживать доставку релиза на разные окружнения, где могут быть разные ОС, разные версии ядра и т.п. Например, нужно собирать агенты с разными параметрами GOOS, GOARCH. Некоторые компании могут быть не против запуска продукта в Docker/K8s.

Вывод № 2: Напрашивается подключение разных Джоб деплоя на разные окружения.
CI/CD общая концепция

Правило 1. Фича-флаги для Джоб CI/CD

Я предлагаю реализовать такой CI/CD, где разработчик конечного проекта сам сможет определить, какие Джобы ему нужно подключать. Для этого он сможет использовать фича-флаги:

Разработчик создает файл .gitlab-ci.yml и инклюдит общий CI/CD:

include:
  - project: 'KazbekT/ci-cd-test'
    file: '/go-cicd.yml'

В этом же файле разработчик подключает Джобы через фича-флаги:

include:
  - project: 'KazbekT/ci-cd-test'
    file: '/go-cicd.yml'

variables:
  BUILD_TYPES: "build-docker,build-astra"
  TEST_JOBS: "test"
  DEPLOY_TARGETS: "deploy-k8s-dev,deploy-k8s-stage"

При этом часть Джоб будет обязательной для всей компнаии, например линтер.

В рамках CI/CD запускается только то, что запросил разработчик через фича-флаги, например так:
image.png

или так:
image.png

Правило 2. Ветка != Окружение

Это был очень хороший паттерн, пока не устарел.

Сейчас принято готовить релиз, и готовый релиз перекатывать по разным окружениям.

Один из многих доводов за этот подход: если мы работаем не с веткой, а с готовым релизом, то у нас нет сомнения, что в production ушли те фичи и коммиты, которые мы тестировали в dev-окружении.

Правило 3. Флоу

В Мире существует большое количество разных флоу: Git Flow, Gitlab Flow, Github Flow, TBD.

В России и СНГ часто приживается Git Flow, и векти Git Flow привязывают к окружениям, например develop-ветка = dev окружение.

Если мы признаем правило 2, то мы можем не мучать разработку сложным Git Flow и можем предложить всем по дефолтку использовать Github Flow, а по необзодимости поменять на любой другой флоу, который нравится/больше подходит для проекта.

Правило 4. Conventional commits

Разработчикам необходимо будет использовать в работе Conventional commits, например:

feat: Добавил новый метод 

fix: Исправил обработку ошибок

chore: Увеличил покрытие тестами

А DevOps реализуют 2 вещи:

проверку коммитов на соответсвие правилам

автоматическую сборку Changelog на базе коммитов

Как будет выглядеть процесс разработки

Когда разработчику надо сделать новую фичу, он делает git checkout с main-ветки в ветку feature/JIRA-213 или feature/JIRA-213-new-method
image.png

Разработчик делает коммиты в ветку feature/JIRA-213 в формате conventional-коммитов:
image.png
На каждый такой коммит в Гитлабе запускается CI-часть папйплайна:

сборка без пуша (но при желании по кнопочке можно запушить)
юнит-тесты
линт
Когда фича готова, разработчик делает MR в main-ветку. Влить фичу в main-ветку можно только после ревью
image.png

Когда в main-ветке скапливается одна или несоклько фич, выпускается релизный тег
image.png

Для релизного тега автоматически создается пайплайн:

сборка
юнит-тесты
пуш
деплой
Деплой мануальный, по нажатию кнопки можно деплоить релизные артефакты на разные окружения.
image.png

Вызовы которые предстоит решить:

Скорее всего нужно будет продумать ролевую модель - кто, куда может деплоить
 

CI. Инструменты и подходы

Подход 1: Логика Джоб внутри Dockerfile

Предлагаю реализовать CI часть пайплайна через Dockerfile'ы, например для Golang:

Такой Dockerfile позволяет реализовать разные Джобы CI/CD через --target флаг во время сборки:

docker build . --target build
docker build . --target lint
docker build . --target test
docker build . --target runtime

Только вместо Docker, в качестве сбрщика нужно использовать Kaniko или BuildKit, но принцип такой же.

Плюсы такого подхода:

CI/CD будет единообразен для всех, вся кастомная логика, которая зависит от фреймворка или особенностей проекта, будет заключена в Dockerfile

Разработчики смогут делать локальные сборки и настраивать локальные стенды в Docker Compose/k3s/minikube

Вызовы которые предстоит решить:

В Dockerfile нужно будет добавить сборки под разные окружения, например еще один stage для сборки под MacOS:

# Build
FROM dependency AS build
ENV CGO_ENABLED=0
ARG PROJECT_NAME
ARG APP_VERSION
WORKDIR /app
RUN go build -ldflags="-X 'main.serviceName= ${PROJECT_NAME}' -w -s -X 'main.serviceVersion=${APP_VERSION}' -w -s" -o application ./main.go

# Build for Mac OS
FROM dependency AS build-macos

ENV CGO_ENABLED=0
ENV GOOS=darwin
ENV GOARCH=amd64 #

ENV CGO_ENABLED=0
ARG PROJECT_NAME
ARG APP_VERSION
WORKDIR /app
RUN go build -ldflags="-X 'main.serviceName= ${PROJECT_NAME}' -w -s -X 'main.serviceVersion=${APP_VERSION}' -w -s" -o application ./main.go

Нужно будет придумать, как публиковать артефакты в Nexus. Кстати, в .Net и придумывать ничего не надо, там есть dotnet push. Думаю для Go и Angular быстро найдется решение.

Джобы gitlab-ci будут выглядеть примерно так (см. --target):

Меняется только таргет, все остальное определяется внутри Dockerfile.

Подход 2: Kubernetes executor

Предлагаю осущестлять запуск всех пайплайнов в Gitlab Runner в Kubernetes. Во-первых, не надо бояться K8s, во-вторых, будет проще с масштабированием, мониторингом и т.п.

Подход 3: Kaniko

Все сборки необходимо будет делать в Kaniko или Buildkit из базовых соображений безопасности. Плюс эти инструменты здорово умеют кэшировать слои. Учитывая объем сущесвтующих наработок (из прошлых компаний), предлагаю Kaniko.

Вызовы которые предстоит решить:

Рано или поздно встанет вопрос e2e тестов, которые обычно запускают в тетс-контейнерах. То есть придется либо делать это на изолированных ВМ, либо играть с проектами, типо dind-rootless.
 

CD. Инструменты и подходы

Эта часть намного сложенее. У продуктов разное количество окружений и сред, куда надо доставлять артефакты.

Также, Всеволодом Петровым идет разработка инсталлятора для проектов, типо Агентов ЦУГИ.

Для начала разделим CD на CDL (delivery) и CDP (deployment).

CDL. Инструменты и подходы

Подход 1: Nexus

Я считаю, что нужно доставлять все виды артефактов и версионировать их исключительно в Nexus. Docker образы, rpm/deb-пакеты, архивы, nuget - Nexus поддерживает все. И хотя у него есть огромные проблемы с масштабированием, пока что у нас нет вариантов лучше.

Подход 2: CDL для DevOps

Помимо CI/CDL пайплайнов для разработки, необходимо будет сделать пайплайны для доставки продуктов DevOps: Docker-образы, Helm чарты и т.п.

CDP. Инструменты и подходы

Подход 1: Для K8s использовать Argo CD

Все, что делпоится в Kubernetes, предлагаю доставлять через Argo CD - у разработчиков будет удобный Argo UI для работы с их проектами.

Также предлагаю строить инфраструктуру DevOps, используя Argo CD, чтобы инфраструктура была надежнее: меньше отказов - меньше замедлений в процессах разработки.

Подход 2: Для ВМ использовать Ansible

Хотелось бы использовать инсталлятор Всеволода, но т.к. он пока не готов, думаю, лучше будет сделать Ansible.
