# fish-greetings-config for Linux

## Usage
1. 
wrtie in terminal
```
gnome-text-editor ~/.config/fish/functions/fish_greeting.fish
```
or choose any your fish_greetings file path
2.
and paste this code below

```fish
function fish_greeting
    # Получаем имя с GitHub
    set_color magenta
    set username "YOUR_USER_NAME"
    set user_info (curl -s https://api.github.com/users/$username)
    set name (echo $user_info | jq -r '.name')

    echo "👋 Привет Senior 📚 Golang Developer, $name!"
    set_color normal
    echo "🕒 The time is "(date)

    # Получаем курсы доллара
    set dollar_to_rub (curl -s https://api.exchangerate-api.com/v4/latest/USD | jq '.rates.RUB')
    set dollar_to_kgs (curl -s https://api.exchangerate-api.com/v4/latest/USD | jq '.rates.KGS')

    echo "💵 The current exchange rate of USD to RUB is $dollar_to_rub"
    echo "💶 The current exchange rate of USD to KGS is $dollar_to_kgs"

    # Получаем задачи из Habitica
    set api_user "YOUR_USER_ID"  # Замените на ваш API User ID
    set api_token "YOUR_USER_API_TOKEN"     # Замените на ваш API токен

    # Получаем список задач
    set tasks (curl -s -H "x-api-user: $api_user" -H "x-api-key: $api_token" https://habitica.com/api/v3/tasks/user)

    echo -e "\n--- Ваши задачи Habitica ---\n"

    # Отображаем ежедневные дела
    echo "🗓️ Ежедневные дела:"
    echo $tasks | jq -r '.data[] | select(.type == "daily") | "\(.text) - Status: \(.completed | if . then "✅" else "❌" end)"' | column -t -s '-'

    # Отображаем обычные задачи
    echo -e "\n🐾 Обычные задачи:"
    echo $tasks | jq -r '.data[] | select(.type == "todo") | "\(.text) - Status: \(.completed | if . then "✅" else "❌" end)"' | column -t -s '-'

    echo -e "\n---------------------------\n"
    # Получаем погоду (замените CITY и API_KEY на свои данные)
    set city "Bishkek"  # Замените на ваш город
    set api_key "YOUR_USER_WEATHER_API"  # Замените на ваш OpenWeather API ключ
    set weather (curl -s "http://api.openweathermap.org/data/2.5/weather?q=$city&appid=$api_key&units=metric&lang=ru")
    set temperature (echo $weather | jq '.main.temp')
    set weather_desc (echo $weather | jq -r '.weather[0].description')
    
    echo "🌤️ Текущая погода в $city: $temperature°C, $weather_desc"
    
    # Предложение по саморазвитию
    set suggestion (curl -s https://api.adviceslip.com/advice | jq -r '.slip.advice')
    echo "💡 Предложение по саморазвитию: $suggestion"
end
```
