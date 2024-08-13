# weather-application
import kivy
from kivy.app import App
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.label import Label
from kivy.uix.textinput import TextInput
from kivy.uix.button import Button
import requests

class WeatherApp(App):
    def build(self):
        self.api_key = "5225ce26df391f8a25a2433215881d84"


        layout = BoxLayout(orientation='vertical')

        self.city_input = TextInput(hint_text='Enter city name', multiline=False)
        layout.add_widget(self.city_input)

        self.result_label = Label(text='Weather details will appear here')
        layout.add_widget(self.result_label)

        get_weather_button = Button(text="Get Weather")
        get_weather_button.bind(on_press=self.get_weather)
        layout.add_widget(get_weather_button)

        return layout

    def get_weather(self, instance):
        city_name = self.city_input.text
        if city_name:
            base_url = "http://api.openweathermap.org/data/2.5/weather?"
            complete_url = f"{base_url}q={city_name}&appid={self.api_key}&units=metric"
            
            response = requests.get(complete_url)
            weather_data = response.json()
            
            if response.status_code == 200:
                if "main" in weather_data:
                    main_data = weather_data["main"]
                    weather_desc = weather_data["weather"][0]["description"]
                    wind_data = weather_data["wind"]
                    
                    temperature = main_data["temp"]
                    pressure = main_data["pressure"]
                    humidity = main_data["humidity"]
                    wind_speed = wind_data["speed"]
                    
                    self.result_label.text = (f"Temperature: {temperature}°C\n"
                                              f"Weather: {weather_desc.capitalize()}\n"
                                              f"Humidity: {humidity}%\n"
                                              f"Wind Speed: {wind_speed} m/s\n"
                                              f"Pressure: {pressure} hPa")
                else:
                    self.result_label.text = "Could not retrieve weather data. Please try again."
            else:
                self.result_label.text = f"Error: {weather_data['message']} (HTTP {response.status_code})"
        else:
            self.result_label.text = "Please enter a city name."

if __name__ == "__main__":
    WeatherApp().run()
