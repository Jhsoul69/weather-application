# weather-application
import kivy
from kivy.app import App
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.label import Label
from kivy.uix.textinput import TextInput
from kivy.uix.button import Button
from kivy.uix.image import Image
from kivy.uix.popup import Popup
from kivy.uix.gridlayout import GridLayout
from kivy.uix.scrollview import ScrollView
from kivy.uix.anchorlayout import AnchorLayout
from kivy.core.window import Window
import requests

class WeatherApp(App):
    def build(self):
        Window.clearcolor = (0, 0, 0, 1)  # Black background


        self.api_key = "5225ce26df391f8a25a2433215881d84"  # Replace with your actual API key

        # Main layout
        main_layout = BoxLayout(orientation='vertical', padding=10, spacing=10)

        # Header layout with search and logo
        header_layout = BoxLayout(size_hint=(1, 0.2), padding=(0, 10, 0, 10))

        logo = Image(source='weather_icon.png', size_hint=(0.3, 1))
        header_layout.add_widget(logo)

        self.city_input = TextInput(hint_text='Search city', multiline=False, size_hint=(0.7, 1), font_size=20)
        header_layout.add_widget(self.city_input)

        search_button = Button(text="Search", size_hint=(0.3, 1), font_size=20, background_color=(0.1, 0.6, 0.8, 1))
        search_button.bind(on_press=self.get_weather)
        header_layout.add_widget(search_button)

        main_layout.add_widget(header_layout)

        # Weather info layout
        self.weather_info_layout = GridLayout(cols=1, padding=10, spacing=10, size_hint_y=None)
        self.weather_info_layout.bind(minimum_height=self.weather_info_layout.setter('height'))

        # Scroll view for weather details
        scroll_view = ScrollView(size_hint=(1, 0.8))
        scroll_view.add_widget(self.weather_info_layout)
        main_layout.add_widget(scroll_view)

        # Footer for additional options (future enhancement)
        footer_layout = AnchorLayout(size_hint=(1, 0.1))
        main_layout.add_widget(footer_layout)

        return main_layout

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

                    # Clear previous weather info
                    self.weather_info_layout.clear_widgets()

                    # Adding weather data in a structured format
                    self.add_weather_info("City", city_name.capitalize())
                    self.add_weather_info("Temperature", f"{temperature}°C")
                    self.add_weather_info("Weather", weather_desc.capitalize())
                    self.add_weather_info("Humidity", f"{humidity}%")
                    self.add_weather_info("Wind Speed", f"{wind_speed} m/s")
                    self.add_weather_info("Pressure", f"{pressure} hPa")

                else:
                    self.show_popup("Error", "Could not retrieve weather data. Please try again.")
            else:
                self.show_popup("Error", f"{weather_data['message']} (HTTP {response.status_code})")
        else:
            self.show_popup("Error", "Please enter a city name.")

    def add_weather_info(self, label_text, value_text):
        layout = BoxLayout(orientation='horizontal', size_hint_y=None, height=40)

        label = Label(text=f"[b]{label_text}:[/b]", font_size=20, size_hint_x=0.4, halign="left", markup=True)
        layout.add_widget(label)

        value = Label(text=value_text, font_size=20, size_hint_x=0.6, halign="right")
        layout.add_widget(value)

        self.weather_info_layout.add_widget(layout)

    def show_popup(self, title, message):
        popup_layout = GridLayout(cols=1, padding=10)
        popup_label = Label(text=message, font_size=18)
        close_button = Button(text="Close", size_hint=(1, 0.25), font_size=18)
        popup_layout.add_widget(popup_label)
        popup_layout.add_widget(close_button)

        popup = Popup(title=title, content=popup_layout, size_hint=(0.75, 0.5))
        close_button.bind(on_press=popup.dismiss)
        popup.open()

if __name__ == "__main__":
    WeatherApp().run()
