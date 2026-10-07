# Специфікація ER-моделі

## Критерії прийняття
* Модель має бути згенерована в синтаксисі mermaid erDiagram
* Усі сутності повинні мати визначені первинні та зовнішні ключі

## Сутності та артрибути
* User: id (UUID, PK), username (String), email (String), register_date (Datetime).
* Anime: id (UUID, PK), title (String), synopsis (Text), release_date (Datetime).
* Review: user_id (FK, PK), anime_id (FK, PK), score (Number), text (Text), user_email (String), created_at (Datetime).
* Genre: id (UUID, PK), name (String).
* AnimeGenres: anime_id (FK), genre_id (FK).

## Зв'язки
* User пише Review (1:N).
* Anime отримує Review (1:N).
* Anime міститься в AnimeGenres (1:N).
* Genre належить до AnimeGenres (1:N).