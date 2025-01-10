import tkinter as tk
from tkinter import ttk
import random
import sys

# -------------------------------------------------------
#                   DATA STRUCTURES
# -------------------------------------------------------

class Card:
    """
    Represents a ship in Space Base, with:
      - name, sector, cost
      - blue_credits, blue_income, blue_vp
      - red_credits, red_income, red_vp
      - special_text (unused in basic example, but you can store triggers here)
    """
    def __init__(self, 
                 name, 
                 sector, 
                 cost, 
                 blue_credits=0, 
                 blue_income=0, 
                 blue_vp=0, 
                 red_credits=0, 
                 red_income=0, 
                 red_vp=0, 
                 special_text=""):
        self.name = name
        self.sector = sector
        self.cost = cost
        self.blue_credits = blue_credits
        self.blue_income = blue_income
        self.blue_vp = blue_vp
        self.red_credits = red_credits
        self.red_income = red_income
        self.red_vp = red_vp
        self.special_text = special_text

    def __str__(self):
        return (f"[{self.name}] S{self.sector} Cost: {self.cost} | "
                f"Blue(C{self.blue_credits},I{self.blue_income},VP{self.blue_vp}) "
                f"Red(C{self.red_credits},I{self.red_income},VP{self.red_vp})")


class Player:
    """
    Represents a player in the game, storing:
    - name (player_id)
    - credits, income, victory points
    - a station: 12 sectors with 'blue' ships
    """
    def __init__(self, player_id):
        self.player_id = player_id
        self.name = f"Player {player_id+1}"
        self.credits = 5
        self.income = 0
        self.vp = 0

        # 12 starter ships
        self.station = []
        for s in range(1, 13):
            # For the base game, each sector s has a starter card with cost=0,
            # and blue_credits = s (some official sets differ for sector 12, but we keep it simple).
            c = Card(
                name=f"Starter {s}",
                sector=s,
                cost=0,
                blue_credits=s,
                blue_income=0,
                blue_vp=0,
                red_credits=0,
                red_income=0,
                red_vp=0,
                special_text="Starter ship"
            )
            self.station.append(c)

    def start_turn_adjustment(self):
        """Ensure the player's credits do not fall below their income at the start of their turn."""
        if self.credits < self.income:
            self.credits = self.income

    def __str__(self):
        return (f"{self.name} | C: {self.credits}, I: {self.income}, VP: {self.vp}")


class GameState:
    """
    Holds the entire game state:
      - players list
      - current_player_idx
      - market (level 1,2,3)
      - dice result
      - game_over, winner
    """
    def __init__(self, num_players=2):
        self.num_players = num_players
        self.players = [Player(pid) for pid in range(num_players)]
        self.current_player_idx = 0
        self.game_over = False
        self.winner = None

        self.dice_result = (0, 0)
        self.last_roll_summed = False

        # Create a representative set of market cards (not complete official distribution).
        self.market_level_1 = [
            Card("Scout MkI", 1, 2, blue_credits=1),
            Card("Transport MkI", 2, 2, blue_income=1),
            Card("Mine Layer MkI", 3, 3, blue_vp=1),
            Card("Patrol Boat MkI", 4, 4, blue_credits=2),
            Card("Torpedo MkI", 5, 5, blue_income=1, blue_credits=1),
            Card("Frigate MkI", 6, 6, blue_vp=1, blue_credits=1),
            Card("Cargo Ship MkI", 7, 7, blue_credits=2),
            Card("Lander MkI", 8, 8, blue_income=1, blue_vp=1),
            Card("Battle Station MkI", 9, 9, blue_credits=1, blue_income=1, blue_vp=1),
            Card("Colony Ship MkI", 10, 5, blue_income=2),
            Card("Rocket MkI", 11, 6, blue_credits=1, blue_vp=1),
            Card("Cruiser MkI", 12, 7, blue_credits=2, blue_income=1),
        ]
        self.market_level_2 = [
            Card("Scout MkII", 1, 8, blue_credits=3),
            Card("Transport MkII", 2, 9, blue_income=2),
            Card("Mine Layer MkII", 3, 10, blue_vp=2),
            Card("Patrol Boat MkII", 4, 11, blue_credits=3),
            Card("Torpedo MkII", 5, 12, blue_income=1, blue_credits=2),
            Card("Frigate MkII", 6, 13, blue_vp=2, blue_credits=2),
            Card("Cargo Ship MkII", 7, 14, blue_credits=3),
            Card("Lander MkII", 8, 15, blue_income=2, blue_vp=1),
            Card("Battle Station MkII", 9, 16, blue_credits=2, blue_income=1, blue_vp=2),
            Card("Colony Ship MkII", 10, 12, blue_income=3),
            Card("Rocket MkII", 11, 13, blue_credits=2, blue_vp=1),
            Card("Cruiser MkII", 12, 14, blue_credits=3, blue_income=1),
        ]
        self.market_level_3 = [
            Card("Scout MkIII", 1, 15, blue_credits=5),
            Card("Transport MkIII", 2, 16, blue_income=3),
            Card("Mine Layer MkIII", 3, 17, blue_vp=3),
            Card("Patrol Boat MkIII", 4, 18, blue_credits=4),
            Card("Torpedo MkIII", 5, 19, blue_income=2, blue_credits=3),
            Card("Frigate MkIII", 6, 20, blue_vp=3, blue_credits=3),
            Card("Cargo Ship MkIII", 7, 21, blue_credits=5),
            Card("Lander MkIII", 8, 22, blue_income=2, blue_vp=2),
            Card("Battle Station MkIII", 9, 23, blue_credits=3, blue_income=2, blue_vp=3),
            Card("Colony Ship MkIII", 10, 20, blue_income=4),
            Card("Rocket MkIII", 11, 22, blue_credits=3, blue_vp=2),
            Card("Cruiser MkIII", 12, 24, blue_credits=4, blue_income=2),
        ]

    def current_player(self):
        return self.players[self.current_player_idx]

    def next_player(self):
        self.current_player_idx = (self.current_player_idx + 1) % self.num_players

    def check_end_game(self):
        """
        Base game typically triggers end when a player reaches 40+ VP,
        then each other player gets one more turn. 
        Here, we'll do an immediate end for simplicity.
        """
        for p in self.players:
            if p.vp >= 40:
                self.game_over = True
                self.winner = p
                return True
        return False


# -------------------------------------------------------
#                 GAME LOGIC FUNCTIONS
# -------------------------------------------------------

def roll_dice():
    """Roll 2 six-sided dice."""
    return (random.randint(1, 6), random.randint(1, 6))

def apply_dice_result(game_state: GameState, dice_result, using_sum):
    """
    Apply dice result to all players:
    - The active player gets BLUE zone effects on the chosen sector(s).
    - All other players get RED zone effects on those sector(s).
    """
    d1, d2 = dice_result
    if using_sum:
        activated_sectors = [d1 + d2]
    else:
        activated_sectors = [d1, d2]

    for idx, player in enumerate(game_state.players):
        for sector in activated_sectors:
            # sector is 1-12
            card = player.station[sector - 1]
            if idx == game_state.current_player_idx:
                # BLUE effect for active player
                player.credits += card.blue_credits
                player.income += card.blue_income
                player.vp += card.blue_vp
            else:
                # RED effect for others
                player.credits += card.red_credits
                player.income += card.red_income
                player.vp += card.red_vp

def buy_card(game_state: GameState, card: Card):
    """
    Attempt to purchase card from the market for the current player.
    Return True if purchase is possible, else False.
    """
    player = game_state.current_player()
    if player.credits < card.cost:
        return False  # Not enough credits
    player.credits -= card.cost
    return True

def place_card_in_sector(player: Player, card: Card, sector_idx: int):
    """
    Place the purchased card into the chosen sector (replacing the old card),
    and flip the old card to its 'red zone' side.
    """
    old_card = player.station[sector_idx]
    # Flip the old card to red zone:
    old_card.red_credits = old_card.blue_credits
    old_card.red_income = old_card.blue_income
    old_card.red_vp = old_card.blue_vp

    # New card is placed
    player.station[sector_idx] = card


# -------------------------------------------------------
#               SCROLLED FRAME UTILITY
# -------------------------------------------------------
class ScrolledFrame(ttk.Frame):
    """
    A reusable scrolled frame where content can exceed the visible area.
    """
    def __init__(self, parent, *args, **kwargs):
        super().__init__(parent, *args, **kwargs)
        
        self.canvas = tk.Canvas(self, bg="#f7f7f7", highlightthickness=0)
        self.scrollbar = ttk.Scrollbar(self, orient="vertical", command=self.canvas.yview)
        self.canvas.configure(yscrollcommand=self.scrollbar.set)

        self.inner_frame = ttk.Frame(self.canvas)
        self.inner_frame_id = None

        self.canvas.pack(side="left", fill="both", expand=True)
        self.scrollbar.pack(side="right", fill="y")

        self.inner_frame.bind("<Configure>", self._on_frame_configure)
        self.inner_frame_id = self.canvas.create_window((0, 0), window=self.inner_frame, anchor="nw")

        # For mouse wheel scrolling on Windows:
        self.canvas.bind_all("<MouseWheel>", self._on_mousewheel)
        # For MacOS (if needed):
        self.canvas.bind_all("<Button-4>", self._on_mousewheel)
        self.canvas.bind_all("<Button-5>", self._on_mousewheel)

    def _on_frame_configure(self, event):
        # Update scroll region
        self.canvas.config(scrollregion=self.canvas.bbox("all"))

    def _on_mousewheel(self, event):
        # Cross-platform approach can vary; on Windows, event.delta is typically in increments of 120
        if event.num == 4:  # macOS scroll up
            self.canvas.yview_scroll(-1, "units")
        elif event.num == 5:  # macOS scroll down
            self.canvas.yview_scroll(1, "units")
        else:
            # On Windows
            direction = -1 if event.delta > 0 else 1
            self.canvas.yview_scroll(direction, "units")


# -------------------------------------------------------
#                          GUI
# -------------------------------------------------------
class SpaceBaseGUI:
    def __init__(self, root):
        self.root = root
        self.root.title("Space Base - Main Menu")
        
        # Start maximized (Windows). On some platforms, "zoomed" might not work.
        try:
            self.root.state('zoomed')
        except:
            # fallback to a large geometry
            self.root.geometry("1400x800")

        self.is_fullscreen = False  # Track fullscreen state
        self.root.bind("<F11>", self.toggle_fullscreen)

        # Add a menu bar
        menubar = tk.Menu(self.root)
        game_menu = tk.Menu(menubar, tearoff=0)
        game_menu.add_command(label="Toggle Fullscreen (F11)", command=self.toggle_fullscreen)
        game_menu.add_separator()
        game_menu.add_command(label="Quit", command=self.root.quit)
        menubar.add_cascade(label="Game", menu=game_menu)
        self.root.config(menu=menubar)

        # Main menu frame
        self.main_menu_frame = ttk.Frame(self.root, padding=20)
        self.main_menu_frame.grid(row=0, column=0, sticky="nsew")

        self.num_players_var = tk.IntVar(value=2)

        # Title
        title_label = ttk.Label(self.main_menu_frame, text="Space Base", font=("Helvetica", 28, "bold"))
        title_label.grid(row=0, column=0, columnspan=2, pady=10)

        # Players label
        players_label = ttk.Label(self.main_menu_frame, text="Number of Players:", font=("Helvetica", 14))
        players_label.grid(row=1, column=0, pady=10, sticky="e")

        # Players dropdown
        players_combo = ttk.Combobox(
            self.main_menu_frame, 
            textvariable=self.num_players_var, 
            values=[2, 3, 4, 5], 
            state="readonly", 
            font=("Helvetica", 14)
        )
        players_combo.grid(row=1, column=1, pady=10, sticky="w")
        players_combo.current(0)

        # Start button
        start_button = ttk.Button(self.main_menu_frame, text="Start Game", command=self.start_game, style="Accent.TButton")
        start_button.grid(row=2, column=0, columnspan=2, pady=20)

        # Configure styles
        style = ttk.Style(self.root)
        style.theme_use("clam")
        style.configure("Accent.TButton", foreground="white", background="#007acc", font=("Helvetica", 12, "bold"))
        style.configure("TLabel", font=("Helvetica", 12))
        style.configure("TButton", font=("Helvetica", 12))
        style.configure("TFrame", background="#f0f0f0")
        style.configure("TLabelFrame", background="#f0f0f0")

        # A frame to hold the actual game once started
        self.game_frame = None
        self.game_state = None

    def toggle_fullscreen(self, event=None):
        self.is_fullscreen = not self.is_fullscreen
        self.root.attributes("-fullscreen", self.is_fullscreen)

    def start_game(self):
        num_players = self.num_players_var.get()
        self.game_state = GameState(num_players=num_players)

        # Destroy main menu
        self.main_menu_frame.destroy()

        # Build the game frame
        self.game_frame = ttk.Frame(self.root, padding=10)
        self.game_frame.grid(row=0, column=0, sticky="nsew")

        # Create subframes: top info, board area, market area, action area
        self.info_frame = ttk.LabelFrame(self.game_frame, text="Game Info")
        self.info_frame.grid(row=0, column=0, columnspan=2, sticky="ew", padx=5, pady=5)

        # Station area (scrollable)
        self.board_frame = ttk.LabelFrame(self.game_frame, text="Stations")
        self.board_frame.grid(row=1, column=0, sticky="nsew", padx=5, pady=5)

        # Market area (scrollable)
        self.market_frame = ttk.LabelFrame(self.game_frame, text="Market")
        self.market_frame.grid(row=1, column=1, sticky="nsew", padx=5, pady=5)

        self.action_frame = ttk.LabelFrame(self.game_frame, text="Actions")
        self.action_frame.grid(row=2, column=0, columnspan=2, sticky="ew", padx=5, pady=5)

        # Expand row/columns
        self.game_frame.rowconfigure(1, weight=1)
        self.game_frame.columnconfigure(1, weight=1)

        # Build sub-components
        self.build_info_panel()
        self.build_board_panel()
        self.build_market_panel()
        self.build_action_panel()

        # Update display
        self.update_display()

    # -----------------------
    #   Info Panel
    # -----------------------
    def build_info_panel(self):
        self.info_label = ttk.Label(self.info_frame, text="Game just started!", font=("Helvetica", 12, "italic"))
        self.info_label.grid(row=0, column=0, sticky="w", padx=10, pady=10)

    # -----------------------
    #   Board Panel (Scroll)
    # -----------------------
    def build_board_panel(self):
        self.board_scrolled = ScrolledFrame(self.board_frame)
        self.board_scrolled.pack(fill="both", expand=True)
        self.player_frames = []

        # We'll create a frame for each player inside board_scrolled.inner_frame
        for pid in range(self.game_state.num_players):
            frame = ttk.LabelFrame(self.board_scrolled.inner_frame, text=f"{self.game_state.players[pid].name}")
            frame.pack(side="top", fill="x", padx=5, pady=5)
            self.player_frames.append(frame)

    def update_board_panel(self):
        # Clear and rebuild each player's station display
        for pid, frame in enumerate(self.player_frames):
            for widget in frame.winfo_children():
                widget.destroy()

            player = self.game_state.players[pid]

            status_label = ttk.Label(frame, text=str(player), font=("Helvetica", 10, "bold"))
            status_label.pack(anchor="w", padx=5, pady=2)

            # Show each sector
            fg_color = "#BF9B30" if (pid == self.game_state.current_player_idx) else "black"
            for sector_idx, card in enumerate(player.station):
                # We'll make a clickable label
                card_text = (
                    f"Sector {sector_idx+1} => {card.name}  "
                    f"(Blue: C{card.blue_credits},I{card.blue_income},VP{card.blue_vp} / "
                    f"Red: C{card.red_credits},I{card.red_income},VP{card.red_vp})"
                )

                lbl = tk.Label(
                    frame, text=card_text, fg=fg_color,
                    font=("Helvetica", 9), justify="left", bg="#f0f0f0"
                )
                lbl.pack(anchor="w", padx=20)

                # Bind click to show card info popup
                lbl.bind("<Button-1>", lambda e, c=card: self.card_info_popup(c))

    # -----------------------
    #   Market Panel (Scroll)
    # -----------------------
    def build_market_panel(self):
        self.market_scrolled = ScrolledFrame(self.market_frame)
        self.market_scrolled.pack(fill="both", expand=True)

    def update_market_panel(self):
        # Clear
        for widget in self.market_scrolled.inner_frame.winfo_children():
            widget.destroy()

        # We'll list level 1, 2, 3 with clickable labels
        def make_card_label(parent, idx, card):
            text_str = f"{idx+1}. {card.name} (Sector {card.sector}, Cost={card.cost})"
            lbl = tk.Label(
                parent, text=text_str, font=("Helvetica", 10), 
                justify="left", bg="#f7f7f7"
            )
            lbl.pack(anchor="w", padx=10)
            lbl.bind("<Button-1>", lambda e, c=card: self.card_info_popup(c))

        # Level 1
        lbl1 = ttk.Label(self.market_scrolled.inner_frame, text="Level 1 Market:", font=("Helvetica", 14, "bold"))
        lbl1.pack(anchor="w", padx=5, pady=5)
        for idx, card in enumerate(self.game_state.market_level_1):
            make_card_label(self.market_scrolled.inner_frame, idx, card)

        ttk.Label(self.market_scrolled.inner_frame, text=" ", font=("Helvetica", 4)).pack()

        # Level 2
        lbl2 = ttk.Label(self.market_scrolled.inner_frame, text="Level 2 Market:", font=("Helvetica", 14, "bold"))
        lbl2.pack(anchor="w", padx=5, pady=5)
        for idx, card in enumerate(self.game_state.market_level_2):
            make_card_label(self.market_scrolled.inner_frame, idx, card)

        ttk.Label(self.market_scrolled.inner_frame, text=" ", font=("Helvetica", 4)).pack()

        # Level 3
        lbl3 = ttk.Label(self.market_scrolled.inner_frame, text="Level 3 Market:", font=("Helvetica", 14, "bold"))
        lbl3.pack(anchor="w", padx=5, pady=5)
        for idx, card in enumerate(self.game_state.market_level_3):
            make_card_label(self.market_scrolled.inner_frame, idx, card)

    # -----------------------
    #   Action Panel
    # -----------------------
    def build_action_panel(self):
        roll_frame = ttk.Frame(self.action_frame)
        roll_frame.pack(side="top", fill="x", padx=5, pady=5)

        self.sum_var = tk.BooleanVar(value=True)
        sum_radio = ttk.Radiobutton(roll_frame, text="Use Sum", variable=self.sum_var, value=True)
        split_radio = ttk.Radiobutton(roll_frame, text="Use Dice Separately", variable=self.sum_var, value=False)
        sum_radio.pack(side="left", padx=5)
        split_radio.pack(side="left", padx=5)

        roll_btn = ttk.Button(roll_frame, text="Roll Dice", command=self.roll_dice_action, style="Accent.TButton")
        roll_btn.pack(side="left", padx=10)

        self.dice_result_label = ttk.Label(roll_frame, text="Dice: ??", font=("Helvetica", 11, "bold"))
        self.dice_result_label.pack(side="left")

        # Buy area
        buy_frame = ttk.Frame(self.action_frame)
        buy_frame.pack(side="top", fill="x", padx=5, pady=5)

        buy_label = ttk.Label(buy_frame, text="Buy (L1, L2, or L3) [#]:")
        buy_label.pack(side="left", padx=5)

        self.market_choice_entry = ttk.Entry(buy_frame, width=10)
        self.market_choice_entry.pack(side="left", padx=5)

        buy_btn = ttk.Button(buy_frame, text="Buy Card", command=self.buy_card_action)
        buy_btn.pack(side="left", padx=5)

        # End turn
        end_turn_btn = ttk.Button(self.action_frame, text="End Turn", command=self.end_turn_action)
        end_turn_btn.pack(side="top", pady=10)

    def roll_dice_action(self):
        if self.game_state.game_over:
            return
        d1, d2 = roll_dice()
        self.game_state.dice_result = (d1, d2)
        using_sum = self.sum_var.get()
        apply_dice_result(self.game_state, (d1, d2), using_sum)

        self.dice_result_label.config(text=f"Dice: {d1}, {d2}")
        self.update_display()

    def buy_card_action(self):
        if self.game_state.game_over:
            return
        choice = self.market_choice_entry.get().strip()
        if not choice:
            return

        parts = choice.split()
        if len(parts) != 2:
            self.info_label.config(text="Invalid format. Use e.g. 'L1 3' or 'L2 2'.")
            return
        level_part, idx_part = parts[0], parts[1]

        level = None
        if level_part.lower() in ["l1", "1"]:
            level = 1
        elif level_part.lower() in ["l2", "2"]:
            level = 2
        elif level_part.lower() in ["l3", "3"]:
            level = 3
        else:
            self.info_label.config(text="Invalid level. Must be 'L1', 'L2', or 'L3'.")
            return

        try:
            card_idx = int(idx_part) - 1
        except ValueError:
            self.info_label.config(text="Invalid card index. Must be an integer.")
            return

        if level == 1:
            market_list = self.game_state.market_level_1
        elif level == 2:
            market_list = self.game_state.market_level_2
        else:
            market_list = self.game_state.market_level_3

        if card_idx < 0 or card_idx >= len(market_list):
            self.info_label.config(text="Invalid card index (out of range).")
            return

        card_to_buy = market_list[card_idx]
        success = buy_card(self.game_state, card_to_buy)
        if not success:
            self.info_label.config(text="Not enough Credits to buy that card.")
            return

        # If buy successful, open sector selection popup
        self.buy_card_popup(card_to_buy, market_list, card_idx)

    def buy_card_popup(self, card_to_buy, market_list, card_idx):
        popup = tk.Toplevel(self.root)
        popup.title("Choose Sector")
        popup.geometry("300x150")
        label = ttk.Label(popup, text=f"Choose a sector (1-12) for {card_to_buy.name}:")
        label.pack(side="top", padx=10, pady=10)

        sector_var = tk.IntVar(value=1)
        sector_entry = ttk.Entry(popup, textvariable=sector_var)
        sector_entry.pack(side="top", padx=10, pady=5)

        def confirm_placement():
            sector_choice = sector_var.get()
            if 1 <= sector_choice <= 12:
                player = self.game_state.current_player()
                place_card_in_sector(player, card_to_buy, sector_choice - 1)

                market_list.pop(card_idx)
                self.update_display()
                popup.destroy()
            else:
                label.config(text="Invalid sector. Must be 1-12.")

        confirm_btn = ttk.Button(popup, text="Confirm", command=confirm_placement)
        confirm_btn.pack(side="top", pady=10)

    def end_turn_action(self):
        if self.game_state.game_over:
            return

        # Current player adjustment
        cp = self.game_state.current_player()
        cp.start_turn_adjustment()

        # Next player
        self.game_state.next_player()

        if self.game_state.check_end_game():
            self.info_label.config(text=f"Game Over! {self.game_state.winner.name} wins with {self.game_state.winner.vp} VP!")
            return

        cp_next = self.game_state.current_player()
        cp_next.start_turn_adjustment()
        self.info_label.config(text=f"{cp_next.name}'s Turn.")
        self.update_display()

    # -----------------------
    #   Card Info Popup
    # -----------------------
    def card_info_popup(self, card: Card):
        """
        Show a popup with full details about the clicked card.
        """
        popup = tk.Toplevel(self.root)
        popup.title(card.name)
        popup.geometry("350x250")

        name_label = ttk.Label(popup, text=card.name, font=("Helvetica", 14, "bold"))
        name_label.pack(pady=5)

        sector_label = ttk.Label(popup, text=f"Sector: {card.sector}")
        sector_label.pack(pady=2)

        cost_label = ttk.Label(popup, text=f"Cost: {card.cost}")
        cost_label.pack(pady=2)

        blue_label = ttk.Label(
            popup, 
            text=f"Blue Effects: +{card.blue_credits}C, +{card.blue_income}I, +{card.blue_vp}VP"
        )
        blue_label.pack(pady=2)

        red_label = ttk.Label(
            popup, 
            text=f"Red Effects: +{card.red_credits}C, +{card.red_income}I, +{card.red_vp}VP"
        )
        red_label.pack(pady=2)

        if card.special_text:
            special_label = ttk.Label(popup, text=f"Special: {card.special_text}", wraplength=300)
            special_label.pack(pady=5)

    # -----------------------
    #   Updating the GUI
    # -----------------------
    def update_display(self):
        self.update_board_panel()
        self.update_market_panel()

        if self.game_state.game_over and self.game_state.winner is not None:
            self.info_label.config(
                text=f"Game Over! {self.game_state.winner.name} wins with {self.game_state.winner.vp} VP!"
            )
        else:
            cp = self.game_state.current_player()
            self.info_label.config(
                text=f"{cp.name}'s Turn | Credits={cp.credits}, Income={cp.income}, VP={cp.vp}"
            )


# -------------------------------------------------------
#                      MAIN
# -------------------------------------------------------
def main():
    root = tk.Tk()
    app = SpaceBaseGUI(root)
    root.mainloop()

if __name__ == "__main__":
    main()

