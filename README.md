```markdown
# 🐟 Fish Greeting Config (Linux)

Персонализированное приветствие для терминала **Fish Shell**, которое отображает:

- Время и имя пользователя
- Курс валют
- Погоду в городе
- Совет дня
- Стрик с GitHub (если есть)
- Стрик с Duolingo
- Задачи из Habitica (`dailies` и `todos`)
- в конце минимальный конфиг для скорости открытия терминала

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



```

function fish_greeting
    set day (date +%j)        # Номер дня в году
    set total 365             # Всего дней
    set left (math "$total - $day")  

    set_color yellow
    echo "📅 Сегодня $day-й день из $total. Осталось $left дней."

    set_color brcyan
    echo "🧠 Цель: Стать Senior за 5 лет"

    set_color blue
    # Массив мотивационных цитат
    set -l quotes \
        "봄이 되면 날씨가 따뜻해지고 나무에 꽃이 피기 시작합니다. Весной погода становится тёплой, и на деревьях начинают распускаться цветы."\
        "여름에는 해가 길고 기온이 매우 높아서 사람들이 자주 바닷가로 갑니다. Летом долгие дни и высокая температура, поэтому люди часто ездят на море."\
        "가을이 되면 나뭇잎이 노랗고 빨갛게 물들고 시원한 바람이 불기 시작합니다. Осенью листья окрашиваются в жёлтые и красные цвета, и начинает дуть прохладный ветер."\
        "겨울에는 기온이 영하로 떨어지고 눈이 내려서 거리와 집이 하얗게 변합니다. Зимой температура опускается ниже нуля, идёт снег, и всё вокруг становится белым."\
        "오늘은 맑고 햇빛이 강해서 사람들이 공원에서 산책을 많이 합니다. Сегодня ясно и яркое солнце, и поэтому многие гуляют в парке."\
        "어제는 하루 종일 비가 와서 외출하기가 매우 불편했습니다. Вчера целый день шёл дождь, и было очень неудобно выходить из дома."\
        "이번 주말에는 날씨가 흐리고 비가 올 예정이라서 실내 활동을 계획하고 있습니다. В эти выходные ожидается пасмурная погода и дождь, поэтому я планирую провести время в помещении."\
        "겨울 방학 동안에는 눈이 많이 와서 친구들과 눈싸움을 하며 즐거운 시간을 보냈습니다. Во время зимних каникул шло много снега, и мы с друзьями весело играли в снежки."\
        "가을 저녁에는 하늘이 붉게 물들고 공기가 선선해서 산책하기에 아주 좋습니다. Осенними вечерами небо окрашивается в красный цвет, и прохладный воздух делает прогулки особенно приятными."\
        "봄에는 새로운 시작을 의미해서 많은 사람들이 기분이 좋아집니다. Весна символизирует новое начало, поэтому у многих поднимается настроение."\
        "저는 부모님과 형, 그리고 강아지 한 마리와 함께 살고 있습니다. Я живу с родителями, старшим братом и одной собакой."\
        "주말마다 우리 가족은 함께 저녁을 먹으며 하루 동안 있었던 일에 대해 이야기합니다. Каждые выходные мы ужинаем всей семьёй и обсуждаем, как прошёл день."\
        "내 친구 민수는 어릴 때부터 알고 지낸 사이로, 지금까지도 자주 만나고 연락합니다. Мой друг Минсу — я знаю его с детства, и мы до сих пор часто встречаемся и переписываемся."\
        "가족은 언제나 나를 지지해주고 힘들 때 옆에 있어줘서 큰 힘이 됩니다. Семья всегда поддерживает меня и остаётся рядом в трудные времена, что даёт мне силы."\
        "명절이 되면 친척들이 모두 모여서 함께 맛있는 음식을 먹고 즐거운 시간을 보냅니다. На праздники собираются все родственники, мы едим вкусную еду и приятно проводим время."\
        "친구들과 함께 여행을 가면 새로운 추억이 생기고 더 친해지는 계기가 됩니다. Совместные путешествия с друзьями создают новые воспоминания и делают нас ближе."\
        "어머니는 매일 아침 따뜻한 아침밥을 차려주시고, 저는 항상 감사한 마음을 가지고 있습니다. Мама каждый день готовит мне тёплый завтрак, и я всегда благодарен ей за это."\
        "아버지는 내가 어려울 때 조용히 조언을 해주시는 분이라서 존경합니다. Папа всегда спокойно даёт советы, когда мне трудно, и я уважаю его за это."\
        "친한 친구와 솔직하게 대화를 나누면 마음이 편해지고 스트레스도 줄어듭니다. Когда честно разговариваешь с близким другом, становится легче на душе и меньше стресса."\
        "가족은 내 삶에서 가장 소중한 존재이며, 함께할 수 있는 시간이 늘 감사하게 느껴집니다. Семья — самое ценное в моей жизни, и я благодарен за каждое мгновение вместе."\
        "제 여동생은 긴 갈색 머리와 큰 눈을 가지고 있어서 사람들한테 자주 예쁘다는 말을 들어요. У моей младшей сестры длинные каштановые волосы и большие глаза, поэтому люди часто говорят, что она красивая."\
        "나는 외모보다는 성격을 더 중요하게 생각하지만, 깔끔한 인상도 첫인상에서는 무시할 수 없다고 생각해요. Я считаю, что характер важнее внешности, но ухоженный вид тоже играет большую роль при первом впечатлении."\
        "그 사람은 키가 크고 말랐지만, 항상 옷을 잘 입어서 멋져 보여요. Он высокий и худощавый, но всегда стильно одевается, поэтому выглядит классно."\
        "제 친구는 머리를 자주 염색하고 새로운 스타일을 시도해서 언제나 개성이 넘쳐요. Мой друг часто красит волосы и пробует новые стили, поэтому у него всегда яркая внешность."\
        "거울을 볼 때마다 피부가 좋아지도록 노력해야겠다는 생각이 들어요. Каждый раз, глядя в зеркало, я думаю, что нужно больше заботиться о своей коже."\
        "외모에 너무 집착하는 건 좋지 않지만, 자기 관리는 자신감에도 영향을 준다고 생각해요. Слишком зацикливаться на внешности — плохо, но забота о себе повышает уверенность."\
        "그녀는 자연스럽고 단정한 스타일을 선호해서 항상 단아하게 보여요. Она предпочитает естественный и аккуратный стиль, поэтому всегда выглядит элегантно."\
        "어렸을 때는 외모에 신경을 많이 안 썼는데, 지금은 첫인상이 중요하다는 걸 느껴요. В детстве я мало обращал внимание на внешность, но теперь понимаю, насколько важны первые впечатления."\
        "그 사람의 눈웃음이 정말 매력적이라서 처음 만났을 때 바로 기억에 남았어요. Его улыбка глазами была настолько обаятельной, что я сразу его запомнил при первой встрече."\
        "운동을 꾸준히 하면서 몸이 건강해지고 외모에도 긍정적인 변화가 생겼어요. С тех пор как я регулярно занимаюсь спортом, моё тело стало здоровее и внешний вид заметно улучшился."\
    # Случайный индекс цитаты
    set -l rand (math (random) % (count $quotes) + 1)
    echo "🧩 Мотивация дня: $quotes[$rand]"

    set_color normal
end



```
