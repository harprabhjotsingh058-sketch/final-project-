import requests
import pandas as pd
from datetime import datetime
import os

# ==============================
# OpenWeatherMap API Key
# ==============================
API_KEY = "2ba5a1b4a30ad67dce1ad2821211d8aa"

BASE_URL = "https://api.openweathermap.org/data/2.5/weather"

# store history next to this script to avoid path problems
CSV_FILE = os.path.join(os.path.dirname(__file__), "weather_history.csv")


def get_weather(city):
    params = {
        "q": city,
        "appid": API_KEY,
        "units": "metric"
    }

    try:
        response = requests.get(BASE_URL, params=params, timeout=10)
        response.raise_for_status()
    except requests.exceptions.RequestException as e:
        print(f"\nNetwork/API error: {e}\n")
        return None

    try:
        data = response.json()
    except ValueError:
        print("\nInvalid response from API.\n")
        return None

    # API may return JSON with 'cod' != 200 and message
    if str(data.get("cod")) != "200":
        msg = data.get("message", "Unknown API error")
        print(f"\nAPI error: {msg}\n")
        return None

    return data


def display_weather(data):
    # use safe accessors with defaults
    city = data.get("name", "N/A")
    country = data.get("sys", {}).get("country", "N/A")

    temperature = data.get("main", {}).get("temp", "N/A")
    feels_like = data.get("main", {}).get("feels_like", "N/A")
    humidity = data.get("main", {}).get("humidity", "N/A")
    pressure = data.get("main", {}).get("pressure", "N/A")

    weather = "N/A"
    try:
        weather = data.get("weather", [{}])[0].get("description", "N/A").title()
    except Exception:
        pass

    wind_speed = data.get("wind", {}).get("speed", "N/A")

    print("\n==============================")
    print("      WEATHER REPORT")
    print("==============================")
    print(f"City        : {city}, {country}")
    print(f"Temperature : {temperature} °C")
    print(f"Feels Like  : {feels_like} °C")
    print(f"Humidity    : {humidity}%")
    print(f"Pressure    : {pressure} hPa")
    print(f"Weather     : {weather}")
    print(f"Wind Speed  : {wind_speed} m/s")
    print("==============================\n")


def save_history(data):
    try:
        record = {
            "Date & Time": datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
            "City": data.get("name", "N/A"),
            "Country": data.get("sys", {}).get("country", "N/A"),
            "Temperature": data.get("main", {}).get("temp", "N/A"),
            "Feels Like": data.get("main", {}).get("feels_like", "N/A"),
            "Humidity": data.get("main", {}).get("humidity", "N/A"),
            "Pressure": data.get("main", {}).get("pressure", "N/A"),
            "Weather": data.get("weather", [{}])[0].get("description", "N/A"),
            "Wind Speed": data.get("wind", {}).get("speed", "N/A")
        }

        df = pd.DataFrame([record])

        if os.path.exists(CSV_FILE):
            df.to_csv(CSV_FILE, mode='a', index=False, header=False)
        else:
            df.to_csv(CSV_FILE, index=False)
    except Exception as e:
        print(f"\nFailed to save history: {e}\n")


def show_history():
    if not os.path.exists(CSV_FILE):
        print("\nNo history found.\n")
        return

    try:
        df = pd.read_csv(CSV_FILE)
        if df.empty:
            print("\nHistory is empty.\n")
            return
        print("\n========= WEATHER HISTORY =========")
        print(df)
        print("===================================\n")
    except pd.errors.EmptyDataError:
        print("\nHistory file is empty or corrupted.\n")
    except Exception as e:
        print(f"\nFailed to read history: {e}\n")


def main():
    while True:
        print("========== LIVE WEATHER DASHBOARD ==========")
        print("1. Check Weather")
        print("2. View Weather History")
        print("3. Exit")

        choice = input("\nEnter your choice: ").strip()

        if choice == "1":
            city = input("Enter City Name: ").strip()
            if not city:
                print("\nPlease enter a city name.\n")
                continue

            data = get_weather(city)

            if data:
                display_weather(data)
                save_history(data)
            else:
                print("\nCould not retrieve weather for that city.\n")

        elif choice == "2":
            show_history()

        elif choice == "3":
            print("\nThank you!\n")
            break

        else:
            print("\nInvalid Choice!\n")


if __name__ == "__main__":
    main()
