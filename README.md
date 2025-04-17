# 🐟 Fish Greeting Config (Linux)

Персонализированное приветствие для терминала **Fish Shell**, отображающее:

- Время и имя пользователя
- Курс валют
- Погоду в городе
- Совет дня
- Задачи из Habitica (`dailies` и `todos`)

---

## 🚀 Установка

### 1. Открой конфигурационный файл `config.fish`

```bash
nano ~/.config/fish/config.fish
```

Или выбери другой путь, если ты используешь отдельный файл для `fish_greeting`.

---

### 2. Установи `jq`, если он ещё не установлен:

#### 📦 Arch / Manjaro:

```bash
sudo pacman -S jq
```

#### 🐧 Debian / Ubuntu:

```bash
sudo apt install jq
```

#### 🍎 macOS:

```bash
brew install jq
```

---

### 3. Вставь следующий код в `config.fish`

```fish
function fish_greeting
    set username "YOUR_GITHUB_USERNAME"
    set city "YOUR_CITY"
    set api_user "YOUR_HABITICA_USER_ID"
    set api_token "YOUR_HABITICA_API_TOKEN"
    set api_key "YOUR_OPENWEATHER_API_KEY"

    # Получаем имя с GitHub
    set user_info (curl -s https://api.github.com/users/$username)
    set name (echo $user_info | jq -r '.name')

    set current_time (date '+%H:%M:%S')
    set dollar_to_rub (curl -s https://api.exchangerate-api.com/v4/latest/USD | jq '.rates.RUB')
    set dollar_to_kgs (curl -s https://api.exchangerate-api.com/v4/latest/USD | jq '.rates.KGS')

    set weather (curl -s "http://api.openweathermap.org/data/2.5/weather?q=$city&appid=$api_key&units=metric&lang=ru")
    set temperature (echo $weather | jq '.main.temp')
    set weather_desc (echo $weather | jq -r '.weather[0].description')

    set suggestion (curl -s https://api.adviceslip.com/advice | jq -r '.slip.advice')

    set tasks (curl -s -H "x-api-user: $api_user" -H "x-api-key: $api_token" https://habitica.com/api/v3/tasks/user)

    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo "👋 Привет, Senior 📚 Golang Developer, $name!"
    echo "🕒 Сейчас: $current_time"
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo "💵 1 USD = $dollar_to_rub RUB"
    echo "💶 1 USD = $dollar_to_kgs KGS"
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
        echo "📋 Задачи из Habitica"
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

    # Ежедневные задачи
    set total_daily 0
    set completed_daily 0
    for task in (echo $tasks | jq -c '.data[] | select(.type == "daily")')
        set text (echo $task | jq -r '.text')
        set completed (echo $task | jq '.completed')
        set total_daily (math $total_daily + 1)
        if test $completed = true
            set completed_daily (math $completed_daily + 1)
        end
    end

    if test $total_daily -gt 0
        set daily_percent (math "($completed_daily / $total_daily) * 100")
        echo "🗓 Ежедневные задачи: ✅ $completed_daily из $total_daily ($daily_percent%)"
    else
        echo "🗓 Ежедневные задачи: нет"
    end

    # Вывод самих задач
    for task in (echo $tasks | jq -c '.data[] | select(.type == "daily")')
        set text (echo $task | jq -r '.text')
        set completed (echo $task | jq '.completed')
        if test $completed = true
            echo "  ✅ $text"
        else
            echo "  ❌ $text"
        end
    end

    echo ""

    # Обычные задачи
    set total_todo 0
    set completed_todo 0
    for task in (echo $tasks | jq -c '.data[] | select(.type == "todo")')
        set text (echo $task | jq -r '.text')
        set completed (echo $task | jq '.completed')
        set total_todo (math $total_todo + 1)
        if test $completed = true
            set completed_todo (math $completed_todo + 1)
        end
    end

    if test $total_todo -gt 0
        set todo_percent (math "($completed_todo / $total_todo) * 100")
        echo "🐾 Обычные задачи: ✅ $completed_todo из $total_todo ($todo_percent%)"
    else
        echo "🐾 Обычные задачи: нет"
    end

    for task in (echo $tasks | jq -c '.data[] | select(.type == "todo")')
        set text (echo $task | jq -r '.text')
        set completed (echo $task | jq '.completed')
        if test $completed = true
            echo "  ✅ $text"
        else
            echo "  ❌ $text"
        end
    end

    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo "🌤 Погода в $city: $temperature°C, $weather_desc"
    echo "💡 Совет дня: $suggestion"
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
end
```

---

## 🧠 Дополнительно

- API ключ для погоды можно получить здесь: [OpenWeatherMap](https://openweathermap.org/api)
- API токены Habitica — в [Настройках пользователя](https://habitica.com/user/settings/api)


---
