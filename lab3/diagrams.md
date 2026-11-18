# Sequence-діаграми UC-2: Оновити статус перегляду

Діаграми деталізують взаємодію для зареєстрованого користувача та гостя.

**Припущення:** перевірка активної сесії виконується через зовнішній механізм `SessionProvider`, якого немає в ER-моделі. Для пари `user_id` + `anime_id` існує не більше одного запису `USER_ANIME_STATUS`. Перед зміною статусу перевіряється наявність запису `ANIME`.

## Сценарій 1: Зареєстрований користувач задає статус аніме

```mermaid
sequenceDiagram
    actor User
    participant WebUI
    participant StatusService
    participant AuthService
    participant SessionProvider as SessionProvider (зовнішній)
    participant Database

    User->>WebUI: chooseStatus(animeId, status)
    activate WebUI
    WebUI->>StatusService: updateStatus(sessionId, animeId, status)
    activate StatusService
    StatusService->>AuthService: validateSession(sessionId)
    activate AuthService
    AuthService->>SessionProvider: validateSession(sessionId)
    activate SessionProvider
    SessionProvider-->>AuthService: activeSession(userId)
    deactivate SessionProvider
    AuthService->>Database: findUserById(userId)
    activate Database
    Database-->>AuthService: USER
    deactivate Database
    AuthService-->>StatusService: authorized(userId)
    deactivate AuthService

    StatusService->>Database: findAnimeById(animeId)
    activate Database
    Database-->>StatusService: ANIME або not found
    deactivate Database
    alt ANIME існує
        StatusService->>Database: findStatus(userId, animeId)
        activate Database
        Database-->>StatusService: USER_ANIME_STATUS або not found
        deactivate Database
        alt Запис USER_ANIME_STATUS існує
            StatusService->>Database: updateStatus(id, status, last_updated)
            activate Database
            Database-->>StatusService: updatedStatus
            deactivate Database
        else Запис USER_ANIME_STATUS відсутній
            StatusService->>Database: insertStatus(userId, animeId, status, last_updated)
            activate Database
            Database-->>StatusService: createdStatus
            deactivate Database
        end
        StatusService-->>WebUI: statusUpdated(status)
    else ANIME не існує
        StatusService-->>WebUI: animeNotFound()
    end
    deactivate StatusService
    WebUI-->>User: showResult()
    deactivate WebUI
```

## Сценарій 2: Гість намагається змінити статус перегляду

```mermaid
sequenceDiagram
    actor Guest
    participant WebUI
    participant StatusService
    participant AuthService
    participant SessionProvider as SessionProvider (зовнішній)
    participant Database

    Guest->>WebUI: chooseStatus(animeId, status)
    activate WebUI
    WebUI->>StatusService: updateStatus(sessionId, animeId, status)
    activate StatusService
    StatusService->>AuthService: validateSession(sessionId)
    activate AuthService
    AuthService->>SessionProvider: validateSession(sessionId)
    activate SessionProvider
    SessionProvider-->>AuthService: activeSession(userId) або inactiveSession
    deactivate SessionProvider
    alt Активної сесії немає
        AuthService-->>StatusService: unauthorized
        deactivate AuthService
        StatusService-->>WebUI: statusChangeRejected()
        Note right of StatusService: USER_ANIME_STATUS не читається і не змінюється
        WebUI-->>Guest: promptSignInOrRegister()
    else Сесія активна
        AuthService->>Database: findUserById(userId)
        activate Database
        Database-->>AuthService: USER або not found
        deactivate Database
        alt Запис USER існує
            AuthService-->>StatusService: authorized(userId)
            deactivate AuthService
            Note over StatusService,Database: Це авторизований користувач; продовження відповідає сценарію 1
            StatusService-->>WebUI: continueAsAuthenticatedUser()
        else Запис USER відсутній
            AuthService-->>StatusService: unauthorized
            deactivate AuthService
            StatusService-->>WebUI: statusChangeRejected()
            Note right of StatusService: USER_ANIME_STATUS не читається і не змінюється
            WebUI-->>Guest: promptSignInOrRegister()
        end
    end
    deactivate StatusService
    deactivate WebUI
```
