from kivy.uix.screenmanager import ScreenManager, Screen
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.floatlayout import FloatLayout
from kivy.uix.button import Button
from kivy.uix.label import Label
from kivy.uix.textinput import TextInput
from kivy.uix.popup import Popup
from kivy.uix.image import Image
from kivy.app import App
import json
import os

USER_DATA_FILE = "users.json"
CART_DATA_FILE = "cart.json"

def load_users():
    if os.path.exists(USER_DATA_FILE):
        with open(USER_DATA_FILE, "r") as f:
            return json.load(f)
    return {}

def save_users(users):
    with open(USER_DATA_FILE, "w") as f:
        json.dump(users, f)

def load_card():
    if os.path.exists(CART_DATA_FILE):
        with open(CART_DATA_FILE, "r") as f:
            return json.load(f)
    return []

def save_card(card):
    with open(CART_DATA_FILE, "w") as f:
        json.dump(card, f)

class LoginScreen(Screen):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        layout = BoxLayout(orientation="vertical", padding=[50, 50, 50, 50], spacing=20)
        
        layout.add_widget(Label(text="Marketplace Login", font_size=24, size_hint=(1, 0.2)))
        
        self.username = TextInput(hint_text="Username", multiline=False, size_hint=(1, 0.15))
        layout.add_widget(self.username)
        
        self.password = TextInput(hint_text="Password", password=True, multiline=False, size_hint=(1, 0.15))
        layout.add_widget(self.password)
        
        login_button = Button(text="Login", size_hint=(1, 0.1))
        login_button.bind(on_press=self.validate_login)
        layout.add_widget(login_button)
        
        signup_button = Button(text="Sign Up", size_hint=(1, 0.1))
        signup_button.bind(on_press=self.go_to_signup)
        layout.add_widget(signup_button)
        self.add_widget(layout)

    def validate_login(self, instance):
        username = self.username.text
        password = self.password.text
        users = load_users()
        if username in users and users[username] == password:
            self.manager.current = "marketplace"
            self.manager.get_screen('profile').username = username  # Pass username to profile screen
        else:
            self.show_popup("Login Failed", "Invalid username or password.")

    def go_to_signup(self, instance):
        self.manager.current = "signup"

    def show_popup(self, title, message):
        popup = Popup(title=title, content=Label(text=message), size_hint=(0.6, 0.4))
        popup.open()

class SignupScreen(Screen):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        layout = BoxLayout(orientation="vertical", padding=[50, 50, 50, 50], spacing=20)
        
        layout.add_widget(Label(text="Create a New Account", font_size=24, size_hint=(1, 0.2)))
        
        self.username = TextInput(hint_text="Choose a Username", multiline=False, size_hint=(1, 0.2))
        layout.add_widget(self.username)
        
        self.password = TextInput(hint_text="Choose a Password", password=True, multiline=False, size_hint=(1, 0.2))
        layout.add_widget(self.password)
        
        self.confirm_password = TextInput(hint_text="Confirm Password", password=True, multiline=False, size_hint=(1, 0.2))
        layout.add_widget(self.confirm_password)
        
        signup_button = Button(text="Sign Up", size_hint=(1, 0.2))
        signup_button.bind(on_press=self.validate_signup)
        layout.add_widget(signup_button)
        
        back_to_login_button = Button(text="Back to Login", size_hint=(1, 0.2))
        back_to_login_button.bind(on_press=self.go_to_login)
        layout.add_widget(back_to_login_button)
        self.add_widget(layout)

    def validate_signup(self, instance):
        username = self.username.text
        password = self.password.text
        confirm_password = self.confirm_password.text
        users = load_users()
        if username in users:
            self.show_popup("Sign-Up Failed", "Username already exists.")
        elif password != confirm_password:
            self.show_popup("Sign-Up Failed", "Passwords do not match.")
        elif len(username) < 3 or len(password) < 6:
            self.show_popup("Sign-Up Failed", "Username or password is too short.")
        else:
            users[username] = password  
            save_users(users)
            self.show_popup("Sign-Up Successful", "Account created successfully!")
            self.manager.current = "marketplace"

    def go_to_login(self, instance):
        self.manager.current = "login"

    def show_popup(self, title, message):
        popup = Popup(title=title, content=Label(text=message), size_hint=(0.6, 0.4))
        popup.open()

class MarketplaceScreen(Screen):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        
        # Use FloatLayout to position elements freely
        float_layout = FloatLayout()
        
        # Menu with buttons for navigation positioned at the top
        menu_layout = BoxLayout(orientation="horizontal", size_hint=(1, 0.1), pos_hint={"top": 1})
        profile_button = Button(text="My Profile", size_hint=(1, 1))
        profile_button.bind(on_press=self.go_to_profile)
        menu_layout.add_widget(profile_button)

        cart_button = Button(text="My Cart", size_hint=(1, 1))
        cart_button.bind(on_press=self.go_to_cart)
        menu_layout.add_widget(cart_button)

        products_button = Button(text="Products", size_hint=(1, 1))
        products_button.bind(on_press=self.go_to_products_screen)
        menu_layout.add_widget(products_button)
        
        # Add menu layout to the main float layout
        float_layout.add_widget(menu_layout)
        
        # Main vertical layout for most of the content
        layout = BoxLayout(orientation="vertical", padding=[50, 50, 50, 50], spacing=20, size_hint=(0.8, 0.8), pos_hint={"center_x": 0.5, "center_y": 0.5})
        
        # Label for products
        layout.add_widget(Label(text="Our Products:", font_size=24, size_hint=(0.2, 0.04)))
        
        # Add image of an iPhone 16 under the product label
        iphone_image = Image(source="iphone_16.png", size_hint=(1, 0.5))
        layout.add_widget(iphone_image)
        
        # Add the main layout to the float layout
        float_layout.add_widget(layout)
        
        # Logout button positioned in the lower left corner
        logout_button = Button(text="Logout", size_hint=(0.2, 0.1), pos_hint={"x": 0, "y": 0})
        logout_button.bind(on_press=self.logout)
        float_layout.add_widget(logout_button)
        
        # Add the float layout to the screen
        self.add_widget(float_layout)
    
    def go_to_cart(self, instance):
        # Logic to navigate to the cart screen
        self.manager.current = "cart"
    
    def go_to_profile(self, instance):
        # Logic to navigate to the profile screen
        self.manager.current = "profile"
    
    def go_to_products_screen(self, instance):
        # Logic to navigate to the products screen
        self.manager.current = "products"
    
    def logout(self, instance):
        self.manager.current = "login"
    
    def show_popup(self, title, message):
        popup = Popup(title=title, content=Label(text=message), size_hint=(0.6, 0.4))
        popup.open()

class CardScreen(Screen):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.layout = BoxLayout(orientation="vertical", padding=[50, 50, 50, 50], spacing=20)
        self.update_card_display()
        
        # Back to marketplace button positioned in the lower left corner
        back_button = Button(text="Back to Marketplace", size_hint=(0.3, 0.15), pos_hint={"left": 1, "y": 0})
        back_button.bind(on_press=self.go_to_marketplace)
        self.add_widget(back_button)
        
        # Checkout button positioned in the lower right corner
        checkout_button = Button(text="Checkout", size_hint=(0.3, 0.15), pos_hint={"right": 1, "y": 0})
        checkout_button.bind(on_press=self.checkout)
        self.add_widget(checkout_button)
        
        # Back to marketplace button positioned in the lower left corner
        back_to_market_button = Button(text="Back to Marketplace", size_hint=(0.3, 0.15), pos_hint={"x": 0, "y": 0})
        back_to_market_button.bind(on_press=self.go_to_marketplace)
        self.add_widget(back_to_market_button)
        self.add_widget(self.layout)

    def update_card_display(self):
        self.layout.clear_widgets()
        card = load_card()
        if card:
            for item in card:
                self.layout.add_widget(Label(text=item, font_size=18, size_hint=(1, 0.2)))
        else:
            self.layout.add_widget(Label(text="Your card is empty.", font_size=18, size_hint=(1, 0.2)))

    def checkout(self, instance):
        save_card([])  # Clear the card
        self.show_popup("Checkout Successful", "Thank you for your purchase!")
        self.update_card_display()

    def go_to_marketplace(self, instance):
        self.manager.current = "marketplace"

    def show_popup(self, title, message):
        popup = Popup(title=title, content=Label(text=message), size_hint=(0.6, 0.4))
        popup.open()

class ProductsScreen(Screen):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        layout = BoxLayout(orientation="vertical", padding=[50, 50, 50, 50], spacing=20)
        
        layout.add_widget(Label(text="Available Products", font_size=24, size_hint=(1, 0.2)))
        
        product_iphone = Button(text="iphone 16 - Add to Cart", size_hint=(1, 0.1))
        product_iphone.bind(on_press=self.add_product_iphone_to_card)
        layout.add_widget(product_iphone)
        
        product_banana = Button(text="banana - Add to Card", size_hint=(1, 0.1))
        product_banana.bind(on_press=self.add_product_banana_to_card)
        layout.add_widget(product_banana)
        
        # Back to marketplace button positioned in the lower right corner
        back_button = Button(text="Back to Marketplace", size_hint=(0.2, 0.1), pos_hint={"right": 1, "y": 0})
        back_button.bind(on_press=self.go_to_marketplace)
        layout.add_widget(back_button)
        
        self.add_widget(layout)
    
    def add_product_iphone_to_card(self, instance):
        card = load_card()
        card.append("iphone 16")
        save_card(card)
        self.show_popup("Product Added", "iphone 16 has been added to your card.")
    
    def add_product_banana_to_card(self, instance):
        card = load_card()
        card.append("banana")
        save_card(card)
        self.show_popup("Product Added", "banana has been added to your card.")
    
    def go_to_marketplace(self, instance):
        self.manager.current = "marketplace"
    
    def show_popup(self, title, message):
        popup = Popup(title=title, content=Label(text=message), size_hint=(0.6, 1))

class profileScreen(Screen):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.username = ""
        self.layout = BoxLayout(orientation="vertical", padding=[50, 50, 50, 50], spacing=20)
        self.update_profile_display()
        
        back_button = Button(text="Back to Marketplace", size_hint=(1, 0.1))
        back_button.bind(on_press=self.go_to_marketplace)
        self.layout.add_widget(back_button)
        self.add_widget(self.layout)

    def update_profile_display(self):
        self.layout.clear_widgets()
        self.layout.add_widget(Label(text=f"Profile Screen", font_size=24, size_hint=(1, 0.2)))
        self.layout.add_widget(Label(text=f"Username: {self.username}", font_size=18, size_hint=(1, 0.1)))
        card = load_card()
        self.layout.add_widget(Label(text=f"Number of Buys: {len(card)}", font_size=18, size_hint=(1, 0.1)))

    def go_to_marketplace(self, instance):
        self.manager.current = "marketplace"

class MarketplaceApp(App):
    def build(self):
        sm = ScreenManager()
        sm.add_widget(LoginScreen(name="login"))
        sm.add_widget(SignupScreen(name="signup"))
        sm.add_widget(MarketplaceScreen(name="marketplace"))
        sm.add_widget(CardScreen(name="cart"))
        sm.add_widget(ProductsScreen(name="products"))
        sm.add_widget(profileScreen(name="profile"))
        return sm

if __name__ == "__main__":
        MarketplaceApp().run()
