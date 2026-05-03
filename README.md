# tutor-finder-docs

Документация web-приложения для онлайн-подбора репетиторов

TutorFinder — web-приложение для онлайн-подбора репетиторов, ориентированное на школьников и студентов, с функциями специально адаптированными под учебный процесс: подбор репетиторов, фильтрация по предметам, отправка заявок, управление заявками.

## Функции приложения

- Регистрация и вход в аккаунт
- Два типа пользователей — ученик и репетитор
- Заполнение профиля с личными данными
- Поиск репетиторов с фильтрацией по предметам
- Отправка заявки репетитору с указанием предмета
- Принятие и отклонение заявок репетитором
- Обмен контактами после принятия заявки
- Редактирование профиля с подтверждением паролем
- Обращение в техподдержку с отправкой письма на почту

Приложение реализуется с использованием клиент-серверной архитектуры:

- Клиентская часть — web-приложение на HTML + JavaScript, репозиторий [tutor-finder-front](https://github.com/Chirkov-Arhip/tutor-finder-front).
- Серверная часть — REST API, написанное на Python с использованием FastAPI, репозиторий [tutor-finder-api](https://github.com/Chirkov-Arhip/tutor-finder-api).

## Оглавление

- [Архитектура приложения](#архитектура-приложения)
- [Схема базы данных](#схема-базы-данных)
- [Диаграммы](#диаграммы)
- [Работа с диаграммами](#работа-с-диаграммами)

## Архитектура приложения

Приложение построено по клиент-серверной архитектуре:

```
Браузер (HTML + JavaScript)
        ↕ HTTP запросы (JSON)
FastAPI сервер (Python)
        ↕ SQLAlchemy ORM
PostgreSQL база данных
```

**Клиентская часть** отвечает за отображение интерфейса и отправку запросов к API. Вся логика взаимодействия с сервером реализована через Fetch API в JavaScript.

**Серверная часть** обрабатывает запросы, работает с базой данных и возвращает данные в формате JSON.

## Схема базы данных

Приложение использует следующие таблицы:

- **users** — все пользователи системы (email, пароль, роль)
- **student_profiles** — профили учеников (ФИО, возраст, телефон, о себе)
- **tutor_profiles** — профили репетиторов (ФИО, возраст, опыт, телефон, о себе)
- **subjects** — справочник предметов
- **tutor_subjects** — связь репетиторов и предметов (многие-ко-многим)
- **applications** — заявки от учеников к репетиторам

## Диаграммы

Диаграммы проекта созданы с использованием библиотеки [Mermaid](https://mermaid.js.org/).

### Диаграмма базы данных

```mermaid
erDiagram
    users {
        int id PK
        string email
        string password_hash
        string role
        datetime created_at
    }
    student_profiles {
        int id PK
        int user_id FK
        string last_name
        string first_name
        string middle_name
        int age
        string phone
        text about
    }
    tutor_profiles {
        int id PK
        int user_id FK
        string last_name
        string first_name
        string middle_name
        int age
        int experience
        string phone
        text about
    }
    subjects {
        int id PK
        string name
    }
    tutor_subjects {
        int tutor_id FK
        int subject_id FK
    }
    applications {
        int id PK
        int student_id FK
        int tutor_id FK
        int subject_id FK
        string custom_subject
        string status
        datetime created_at
    }

    users ||--o| student_profiles : "имеет"
    users ||--o| tutor_profiles : "имеет"
    tutor_profiles ||--o{ tutor_subjects : "преподаёт"
    subjects ||--o{ tutor_subjects : "преподаётся"
    student_profiles ||--o{ applications : "отправляет"
    tutor_profiles ||--o{ applications : "получает"
    subjects ||--o{ applications : "указывается в"
```

### Диаграмма последовательности — регистрация ученика

```mermaid
sequenceDiagram
    actor Ученик
    participant Браузер
    participant API
    participant БД

    Ученик->>Браузер: Заполняет форму регистрации
    Браузер->>API: POST /api/auth/register
    API->>БД: Создать пользователя
    БД-->>API: OK
    API-->>Браузер: {id, email, role}
    Браузер->>Браузер: Сохранить user_id в localStorage
    Браузер->>Ученик: Перенаправить на заполнение профиля
```

### Диаграмма последовательности — отправка заявки

```mermaid
sequenceDiagram
    actor Ученик
    participant Браузер
    participant API
    participant БД

    Ученик->>Браузер: Выбирает репетитора
    Браузер->>API: GET /api/students/tutor-subjects/{tutor_id}
    API->>БД: Получить предметы репетитора
    БД-->>API: Список предметов
    API-->>Браузер: [{id, name}]
    Ученик->>Браузер: Выбирает предмет и отправляет заявку
    Браузер->>API: POST /api/students/apply
    API->>БД: Создать заявку
    БД-->>API: OK
    API-->>Браузер: {message, id}
    Браузер->>Ученик: Заявка отправлена
```

### Диаграмма последовательности — принятие заявки

```mermaid
sequenceDiagram
    actor Репетитор
    participant Браузер
    participant API
    participant БД

    Репетитор->>Браузер: Открывает кабинет
    Браузер->>API: GET /api/tutors/applications/{tutor_id}
    API->>БД: Получить заявки
    БД-->>API: Список заявок
    API-->>Браузер: [{id, student, subject, status}]
    Репетитор->>Браузер: Нажимает "Принять"
    Браузер->>API: PATCH /api/tutors/applications/{app_id}
    API->>БД: Обновить статус заявки
    БД-->>API: OK
    API-->>Браузер: {status: accepted}
    Браузер->>Репетитор: Показать телефон ученика
```

## Работа с диаграммами

Для разработки диаграмм можно использовать библиотеку [Mermaid](https://mermaid.js.org/) и [онлайн-редактор диаграмм](https://mermaid.live/edit).

Для работы с диаграммами в VS Code рекомендуется установить расширения:

- **Markdown Preview Mermaid Support** — предпросмотр диаграмм в Markdown
- **Code Spell Checker** — проверка орфографии
- **Russian - Code Spell Checker** — проверка орфографии на русском языке
