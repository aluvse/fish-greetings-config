```markdown
# 🐟 Fish Greeting Config (Linux)

Персонализированное приветствие для терминала **Fish Shell**, которое отображает:

- Время и имя пользователя
- Курс валют
- Погоду в городе
- Совет дня
- Стрик с GitHub (если есть)
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
    set duolingo_user "YOUR_DUOLINGO_USERNAME"

    # Получаем имя с GitHub
    set user_info (curl -s https://api.github.com/users/$username)
    set name (echo $user_info | jq -r '.name')

    # Получаем события с GitHub (публичные)
    set events (curl -s "https://api.github.com/users/$username/events/public")
    set streak_days 0
    set last_commit_date ""

    # Подсчитываем стрик на основе событий
    for event in (echo $events | jq -c '.[] | select(.type == "PushEvent")')
        set commit_date (echo $event | jq -r '.created_at')
        
        # Если нет последнего коммита, начинаем с текущего
        if test -z $last_commit_date
            set last_commit_date $commit_date
            set streak_days 1
        else
            set prev_date (date -d $last_commit_date '+%Y-%m-%d')
            set current_date (date -d $commit_date '+%Y-%m-%d')

            # Проверяем разницу в днях между коммитами
            set diff_seconds (math "(($(date -d $commit_date +%s) - $(date -d $last_commit_date +%s)) / 86400)")
            if test $diff_seconds -eq 1
                set streak_days (math $streak_days + 1)
            else
                break
            end
            set last_commit_date $commit_date
        end
    end

    # Получаем стрик Duolingo (исправленная версия)
    set duolingo_streak 0
    if test -n "$duolingo_user"
        set duolingo_data (curl -s -A "Mozilla/5.0" "https://www.duolingo.com/2017-06-30/users?username=$duolingo_user")
        if test $status -eq 0
            set duolingo_streak (echo $duolingo_data | jq -r '.users[0].streak // 0' 2>/dev/null || echo 0)
        else
            set duolingo_streak "API error"
        end
    end

    # Текущее время
    set current_time (date '+%H:%M:%S')

    # Курсы валют
    set dollar_to_rub (curl -s https://api.exchangerate-api.com/v4/latest/USD | jq '.rates.RUB')
    set dollar_to_kgs (curl -s https://api.exchangerate-api.com/v4/latest/USD | jq '.rates.KGS')
    set dollar_to_krw (curl -s https://api.exchangerate-api.com/v4/latest/USD | jq '.rates.KRW')

    # Погода
    set weather (curl -s "http://api.openweathermap.org/data/2.5/weather?q=$city&appid=$api_key&units=metric&lang=ru")
    set temperature (echo $weather | jq '.main.temp')
    set weather_desc (echo $weather | jq -r '.weather[0].description')

    # Совет дня
    set suggestion (curl -s https://api.adviceslip.com/advice | jq -r '.slip.advice')

    # Задачи Habitica
    set tasks (curl -s -H "x-api-user: $api_user" -H "x-api-key: $api_token" https://habitica.com/api/v3/tasks/user)

    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo "👋 Hi Flutter, Golang Developer, $name!"
    echo "🕒 Now: $current_time"
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo "🇷🇺 1 USD = $dollar_to_rub RUB"
    echo "🇰🇬 1 USD = $dollar_to_kgs KGS"
    echo "🇰🇷 1 USD = $dollar_to_krw KRW" 
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo "🍏 Duolingo Streak: $duolingo_streak days"
    echo "🔥 GitHub Streak: $streak_days days"
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo "📋 Habitica habits"
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
        echo "🥶 Habits: ✅ $completed_daily из $total_daily ($daily_percent%)"
    else
        echo "🥶 Habits: empty"
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
        echo "🐾 Goals: ✅ $completed_todo из $total_todo ($todo_percent%)"
    else
        echo "🐾 Goals: empty"
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
    echo "🌀 Weather in $city: $temperature°C, $weather_desc"
    echo "🕊️ Tips of the day: $suggestion"
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
end
```

---

## 🧠 Дополнительно

- API ключ для погоды можно получить здесь: [OpenWeatherMap](https://openweathermap.org/api)
- API токены Habitica — в [Настройках пользователя](https://habitica.com/user/settings/api)

--- 
