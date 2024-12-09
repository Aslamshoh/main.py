from kivy.app import App
from kivy.uix.widget import Widget
from kivy.uix.button import Button
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.label import Label
from kivy.uix.gridlayout import GridLayout
from kivy.graphics import Rectangle, Color
import random

GRID_SIZE = 10
CELL_SIZE = 40

class BattleshipGame(Widget):
    def init(self, **kwargs):
        super().init(**kwargs)
        self.ships = self.place_ships()
        self.player_moves = set()
        self.enemy_moves = set()
        self.turn = "player"
        self.player_score = 0
        self.enemy_score = 0
        self.difficulty = "easy"
        self.bind(pos=self.update_canvas, size=self.update_canvas)

    def place_ships(self):
        ship_sizes = [2, 3, 3, 4, 5]
        ships = []
        for size in ship_sizes:
            while True:
                x = random.randint(0, GRID_SIZE - 1)
                y = random.randint(0, GRID_SIZE - 1)
                orientation = random.choice(["horizontal", "vertical"])
                if self.can_place_ship(x, y, size, orientation, ships):
                    ships.append((x, y, size, orientation))
                    break
        return ships

    def can_place_ship(self, x, y, size, orientation, ships):
        coordinates = []
        for i in range(size):
            nx, ny = (x + i, y) if orientation == "horizontal" else (x, y + i)
            if nx >= GRID_SIZE or ny >= GRID_SIZE:
                return False
            coordinates.append((nx, ny))
        for nx, ny in coordinates:
            for dx in [-1, 0, 1]:
                for dy in [-1, 0, 1]:
                    if (nx + dx, ny + dy) in [coord for ship in ships for coord in self.get_ship_coordinates(ship)]:
                        return False
        return True

    def get_ship_coordinates(self, ship):
        x, y, size, orientation = ship
        return [(x + i, y) if orientation == "horizontal" else (x, y + i) for i in range(size)]

    def on_touch_down(self, touch):
        if self.turn != "player":
            return

        x, y = int(touch.x // CELL_SIZE), int(touch.y // CELL_SIZE)
        if (x, y) in self.player_moves:
            return
        self.player_moves.add((x, y))

        hit = any((x, y) in self.get_ship_coordinates(ship) for ship in self.ships)
        color = (1, 0, 0, 1) if hit else (1, 1, 1, 1)
        with self.canvas:
            Color(*color)
            Rectangle(pos=(x * CELL_SIZE, y * CELL_SIZE), size=(CELL_SIZE, CELL_SIZE))

        if hit:
            self.player_score += 1
            self.update_score()

        self.turn = "enemy"
        self.enemy_turn()

    def enemy_turn(self):
        if self.turn != "enemy":
            return
        while True:
            x, y = random.randint(0, GRID_SIZE - 1), random.randint(0, GRID_SIZE - 1)
            if (x, y) not in self.enemy_moves:
                self.enemy_moves.add((x, y))
                break
        hit = any((x, y) in self.get_ship_coordinates(ship) for ship in self.ships)
        color = (1, 0, 0, 1) if hit else (1, 1, 1, 1)
        with self.canvas:
            Color(*color)
            Rectangle(pos=(x * CELL_SIZE, y * CELL_SIZE), size=(CELL_SIZE, CELL_SIZE))

        if hit:
            self.enemy_score += 1
            self.update_score()

        self.turn = "player"

    def update_canvas(self, *args):
        self.canvas.clear()
        self.draw_grid()

    def draw_grid(self):
        with self.canvas:
            for x in range(0, GRID_SIZE * CELL_SIZE, CELL_SIZE):
                for y in range(0, GRID_SIZE * CELL_SIZE, CELL_SIZE):
                    Color(0, 0, 0, 1)
                    Rectangle(pos=(x, y), size=(CELL_SIZE, CELL_SIZE))

    def update_score(self):
        pass  # Реализуйте метод обновления счета (например, с помощью Kivy Label)
class BattleshipApp(App):
    def build(self):
        root = BoxLayout(orientation='vertical')
        self.game = BattleshipGame()
        root.add_widget(self.game)

        button_layout = BoxLayout(size_hint=(1, 0.2))
        start_button = Button(text='Начать игру', on_press=self.start_game)
        auto_button = Button(text='Авторегистрация', on_press=self.auto_register)
        difficulty_button = Button(text='Уровень игры', on_press=self.change_difficulty)
        player_button = Button(text='Игрок', on_press=self.switch_to_player)
        ai_button = Button(text='ИИ', on_press=self.switch_to_ai)

        button_layout.add_widget(start_button)
        button_layout.add_widget(auto_button)
        button_layout.add_widget(difficulty_button)
        button_layout.add_widget(player_button)
        button_layout.add_widget(ai_button)

        root.add_widget(button_layout)
        return root

    def start_game(self, instance):
        self.game.ships = self.game.place_ships()
        self.game.player_moves = set()
        self.game.enemy_moves = set()
        self.game.player_score = 0
        self.game.enemy_score = 0
        self.game.turn = "player"
        self.game.update_canvas()

    def auto_register(self, instance):
        self.game.ships = self.game.place_ships()
        self.game.update_canvas()

    def change_difficulty(self, instance):
        if self.game.difficulty == "easy":
            self.game.difficulty = "medium"
        elif self.game.difficulty == "medium":
            self.game.difficulty = "hard"
        else:
            self.game.difficulty = "easy"

    def switch_to_player(self, instance):
        self.game.turn = "player"

    def switch_to_ai(self, instance):
        self.game.turn = "enemy"
        self.game.enemy_turn()

if __name__ == 'main':
    BattleshipApp().run()
