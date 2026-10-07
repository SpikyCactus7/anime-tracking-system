# Специфікація ER-моделі

## Критерії прийняття
* Модель має бути згенерована в синтаксисі mermaid erDiagram.
* Усі сутності повинні мати визначені первинні та зовнішні ключі.
* Використовувати зв'язки багато-до-багатьох замість сполучних таблиць.
* Модель має відповідати 3НФ

## Сутності та атрибути
* User:
 - id (UUID, PK),
 - username (String),
 - email (String),
 - register_date (Datetime).

* Anime:
 - id (UUID, PK),
 - title (String),
 - synopsis (Text),
 - release_date (Datetime).

* Review:
 - id (UUID, PK),
 - user_id (FK),
 - anime_id (FK),
 - score (Number),
 - text (Text),
 - created_at (Datetime).

* Genre:
 - id (UUID, PK),
 - name (String).

* UserAnimeStatus:
 - id (UUID, PK),
 - user_id (FK),
 - anime_id (FK),
 - status (String),
 - last_updated (Datetime).

## Зв'язки
* User пише Review (1:N).
* Anime отримує Review (1:N).
* Anime має Genre (M:N).
* User має UserAnimeStatus (1:N).
* Anime зв'язано з UserAnime (1:N).