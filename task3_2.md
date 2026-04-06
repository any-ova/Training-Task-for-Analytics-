### Задание 3.2: Алгоритм регистрации пользователя

Ниже представлена блок-схема процесса регистрации на стороне бэкенда.

```mermaid
graph TD
    Start([Начало: POST /api/v1/users/register]) --> Req[Десериализация JSON тела запроса]

    %% 1. Проверка структуры
    Req --> CheckFields{Все обязательные поля заполнены?}
    CheckFields -- Нет --> Err400Struct["Return 400: Missing required fields"]
    CheckFields -- Да --> CaptchaVerify[Проверка токена reCAPTCHA]

    %% 2. Проверка капчи (ошибка 403)
    CaptchaVerify --> IsCaptchaValid{Токен валиден?}
    IsCaptchaValid -- Нет --> Err403["Return 403: Please verify reCaptcha to register!"]
    IsCaptchaValid -- Да --> CheckPasswordPolicy{Пароль валиден?}

    %% 3. Проверка пароля (ошибка 400)
    CheckPasswordPolicy -- Нет --> Err400Pass["Return 400: Passwords must have at least..."]
    CheckPasswordPolicy -- Да --> CheckDB[(Проверка в БД)]

    %% 4. Проверка уникальности
    CheckDB --> IsUserExists{Пользователь существует?}
    IsUserExists -- Да --> Err409["Return 409: User exist"]
    IsUserExists -- Нет --> HashPassword[Хеширование пароля]

    %% 5. Сохранение
    HashPassword --> SaveDB[(Сохранение в БД)]
    SaveDB --> Success["Return 200: User Register Successfully"]

    %% Выходы
    Err400Struct --> End([Конец])
    Err403 --> End
    Err400Pass --> End
    Err409 --> End
    Success --> End

    %% Стили
    classDef error fill:#ffcccc,stroke:#cc0000,color:black;
    classDef success fill:#ccffcc,stroke:#009900,color:black;
    classDef decision fill:#ffffcc,stroke:#cccc00,color:black;
    classDef process fill:#e6e6e6,stroke:#666666,color:black;
    classDef db fill:#cce5ff,stroke:#004085,color:black;

    class Err400Struct,Err403,Err400Pass,Err409 error;
    class Success success;
    class CheckFields,IsCaptchaValid,CheckPasswordPolicy,IsUserExists decision;
    class Req,CaptchaVerify,HashPassword process;
    class CheckDB,SaveDB db;
