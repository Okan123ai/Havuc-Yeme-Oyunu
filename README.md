CODE:

PASTE VİSUAL STUDİO CODE


import math
import random
import tkinter as tk


WIDTH, HEIGHT = 960, 640
LEVEL_TARGETS = (8, 12, 16, 20, 25)
FRAME_MS = 35


class CarrotGarden:
    def __init__(self, root):
        self.root = root
        self.root.title("Havuç Bahçesi | Tavşanın Lezzet Macerası")
        self.root.geometry(f"{WIDTH}x{HEIGHT}")
        self.root.resizable(False, False)
        self.root.configure(bg="#182d27")

        self.canvas = tk.Canvas(
            root,
            width=WIDTH,
            height=HEIGHT,
            background="#8bc9c2",
            highlightthickness=0,
        )
        self.canvas.pack()

        self.screen = "menu"
        self.mode = "endless"
        self.level = 1
        self.eaten = 0
        self.total_eaten = 0
        self.bite_animation = 0.0
        self.chew_animation = 0.0
        self.transition_timer = 0.0
        self.elapsed = 0.0
        self.last_frame = 0.0
        self.tick_id = None
        self.buttons = {}

        self.root.bind("<KeyPress>", self.on_key)
        self.canvas.bind("<Button-1>", self.on_click)
        self.draw_menu()

    def oval(self, x1, y1, x2, y2, color, outline="", width=1, **options):
        return self.canvas.create_oval(
            x1, y1, x2, y2, fill=color, outline=outline, width=width, **options
        )

    def polygon(self, points, color, outline="", width=1, **options):
        return self.canvas.create_polygon(
            points, fill=color, outline=outline, width=width, **options
        )

    def text(self, x, y, value, size, color, weight="normal", **options):
        return self.canvas.create_text(
            x,
            y,
            text=value,
            fill=color,
            font=("Segoe UI", size, weight),
            **options,
        )

    def rounded_panel(self, x1, y1, x2, y2, fill, outline="#ffffff"):
        self.canvas.create_rectangle(
            x1 + 9,
            y1,
            x2 - 9,
            y2,
            fill=fill,
            outline="",
        )
        self.canvas.create_rectangle(
            x1,
            y1 + 9,
            x2,
            y2 - 9,
            fill=fill,
            outline="",
        )
        for x, y in (
            (x1 + 9, y1 + 9),
            (x2 - 9, y1 + 9),
            (x1 + 9, y2 - 9),
            (x2 - 9, y2 - 9),
        ):
            self.oval(x - 9, y - 9, x + 9, y + 9, fill)
        if outline:
            self.canvas.create_rectangle(
                x1 + 9,
                y1,
                x2 - 9,
                y2,
                outline=outline,
                width=1,
            )
            self.canvas.create_rectangle(
                x1,
                y1 + 9,
                x2,
                y2 - 9,
                outline=outline,
                width=1,
            )

    def draw_scene(self, clock):
        canvas = self.canvas
        canvas.delete("all")

        # A soft sky gradient keeps the distant hills bright and atmospheric.
        sky = ("#79b9ba", "#91c8be", "#b6d6b7", "#d9e2b9")
        band_height = 90
        for index, color in enumerate(sky):
            canvas.create_rectangle(
                0,
                index * band_height,
                WIDTH,
                (index + 1) * band_height + 1,
                fill=color,
                outline="",
            )

        self.oval(727, 52, 813, 138, "#f7e6a8")
        self.oval(740, 64, 800, 124, "#fff3c8")

        for x, y, scale in ((170, 123, 1.0), (435, 81, 0.7), (850, 172, 0.82)):
            self.draw_cloud(x, y, scale)

        self.polygon(
            [0, 248, 125, 185, 228, 249, 354, 164, 477, 250, 618, 183,
             741, 253, 857, 178, 960, 238, 960, 397, 0, 397],
            "#829e7e",
        )
        self.polygon(
            [0, 290, 115, 228, 219, 286, 333, 217, 449, 295, 573, 220,
             693, 290, 832, 225, 960, 286, 960, 426, 0, 426],
            "#719473",
        )
        self.polygon(
            [0, 326, 132, 273, 268, 326, 382, 263, 523, 327, 677, 269,
             792, 327, 960, 275, 960, 480, 0, 480],
            "#527d61",
        )

        # The converging garden rows create depth without external 3D assets.
        self.polygon([0, 374, 960, 374, 960, 640, 0, 640], "#507954")
        self.polygon(
            [432, 317, 529, 317, 802, 555, 158, 555],
            "#a99870",
        )
        self.polygon([462, 321, 477, 321, 393, 555, 360, 555], "#c7b78d")
        self.polygon([485, 321, 493, 321, 522, 555, 501, 555], "#8f815f")
        self.polygon([0, 424, 380, 467, 0, 505], "#669060")
        self.polygon([960, 418, 591, 466, 960, 509], "#628a5e")

        # Small rows of plants shrink toward the horizon for a 2.5D perspective.
        for row in range(7):
            depth = row / 6
            y = 357 + depth * 137
            spread = 48 + depth * 190
            scale = 0.34 + depth * 0.85
            for side in (-1, 1):
                for plant in range(7):
                    fraction = plant / 6
                    x = WIDTH / 2 + side * (
                        spread + fraction * (85 + depth * 125)
                    )
                    height = (8 + ((plant * 7 + row * 3) % 8)) * scale
                    self.draw_plant(x, y, scale, height)

        self.draw_ambient_grass(clock)
        self.draw_tree(57, 326, 1.02)
        self.draw_tree(899, 310, 0.84)
        self.draw_bunny(clock)

    def draw_cloud(self, x, y, scale):
        color = "#dce9d2"
        self.oval(x, y, x + 78 * scale, y + 24 * scale, color)
        self.oval(x + 15 * scale, y - 14 * scale,
                  x + 48 * scale, y + 22 * scale, color)
        self.oval(x + 42 * scale, y - 8 * scale,
                  x + 72 * scale, y + 22 * scale, color)

    def draw_tree(self, x, y, scale):
        self.oval(x - 37 * scale, y + 74 * scale,
                  x + 44 * scale, y + 91 * scale, "#466f52")
        self.polygon(
            [x - 6 * scale, y + 61 * scale, x + 10 * scale, y + 61 * scale,
             x + 12 * scale, y + 93 * scale, x - 8 * scale, y + 93 * scale],
            "#705b40",
        )
        for dx, dy, size, color in (
            (-18, 19, 34, "#355f48"),
            (12, 15, 41, "#477953"),
            (-4, -6, 43, "#557f54"),
            (22, -9, 29, "#67905c"),
        ):
            self.oval(
                x + (dx - size / 2) * scale,
                y + (dy - size / 2) * scale,
                x + (dx + size / 2) * scale,
                y + (dy + size / 2) * scale,
                color,
            )
        self.oval(
            x - 10 * scale,
            y - 23 * scale,
            x + 5 * scale,
            y - 8 * scale,
            "#83a767",
        )

    def draw_plant(self, x, y, scale, height):
        self.oval(x - 5 * scale, y - 2 * scale,
                  x + 6 * scale, y + 3 * scale, "#d0b482")
        self.polygon(
            [x, y + 1 * scale, x - 2 * scale, y - height,
             x + 1 * scale, y - height * 0.37],
            "#497445",
        )
        self.polygon(
            [x, y - height * 0.48,
             x - height * 0.53, y - height * 0.78,
             x - height * 0.08, y - height * 0.34],
            "#628b4d",
        )
        self.polygon(
            [x, y - height * 0.6,
             x + height * 0.50, y - height * 0.87,
             x + height * 0.12, y - height * 0.42],
            "#739451",
        )

    def draw_ambient_grass(self, clock):
        rng = random.Random(29)
        for index in range(64):
            x = rng.randint(8, WIDTH - 8)
            y = rng.randint(449, HEIGHT + 3)
            height = rng.randint(5, 17)
            sway = math.sin(clock * 1.5 + index) * 2.2
            shade = ("#668f5b", "#72965d", "#466f4f")[index % 3]
            self.polygon(
                [x - 2, y, x + sway, y - height, x + 3, y],
                shade,
            )

    def draw_bunny(self, clock):
        chew = math.sin(clock * 19) * 2 if self.chew_animation > 0 else 0
        breath = math.sin(clock * 2.1) * 2

        self.oval(371, 493, 595, 534, "#385d46")
        self.oval(411, 450 + breath, 537, 524 + breath, "#e4d7b6")
        self.oval(425, 463 + breath, 533, 521 + breath, "#f5ebd3")
        self.oval(395, 485, 429, 513, "#eee3cb")
        self.oval(390, 492, 424, 526, "#f6ead3")

        self.oval(412, 497, 471, 531, "#efe3ca")
        self.oval(478, 498, 541, 532, "#e9ddc3")
        self.oval(420, 502, 461, 525, "#c6b99f")
        self.oval(487, 502, 532, 526, "#c4b79e")

        ear_sway = math.sin(clock * 2.0) * 3
        self.oval(416, 330 + ear_sway, 453, 440 + ear_sway, "#dfd2b7")
        self.oval(422, 338 + ear_sway, 447, 428 + ear_sway, "#d18b83")
        self.oval(448, 319 - ear_sway, 487, 432 - ear_sway, "#f0e5cf")
        self.oval(455, 328 - ear_sway, 480, 418 - ear_sway, "#d99b92")

        self.oval(412, 379 + chew * 0.3, 538, 498 + chew, "#d5c7aa")
        self.oval(420, 378 + chew * 0.3, 532, 490 + chew, "#f4ead5")
        self.oval(435, 385, 464, 418, "#fff8e6")
        self.oval(486, 387, 516, 420, "#fdf4df")
        self.oval(437, 387, 462, 415, "#fcf4e3")
        self.oval(488, 389, 514, 418, "#f7edda")

        self.oval(447, 402, 457, 415, "#302d2b")
        self.oval(495, 403, 505, 416, "#302d2b")
        self.oval(449, 403, 452, 407, "#ffffff")
        self.oval(497, 404, 500, 408, "#ffffff")

        self.oval(461, 426, 480, 439, "#d7837d")
        self.oval(464, 428, 468, 431, "#f5c2ad")
        self.polygon([469, 438, 477, 438, 473, 444], "#81625d")
        self.canvas.create_line(473, 443, 473, 450, fill="#876d62", width=2)
        mouth_height = 451 + chew
        self.canvas.create_arc(
            460,
            443,
            474,
            mouth_height + 9,
            start=200,
            extent=140,
            style=tk.ARC,
            outline="#795e55",
            width=2,
        )
        self.canvas.create_arc(
            473,
            443,
            487,
            mouth_height + 9,
            start=200,
            extent=140,
            style=tk.ARC,
            outline="#795e55",
            width=2,
        )

        for direction in (-1, 1):
            self.oval(
                420 if direction < 0 else 508,
                428,
                451 if direction < 0 else 528,
                454,
                "#efb4a1",
            )
        for index in range(3):
            y = 432 + index * 6
            self.canvas.create_line(
                453, y, 427, y - 5 + index * 4,
                fill="#ab9981", width=1,
            )
            self.canvas.create_line(
                493, y, 520, y - 5 + index * 4,
                fill="#ab9981", width=1,
            )

        paw_lift = math.sin(clock * 7) * 3 if self.chew_animation > 0 else 0
        self.oval(491, 451 - paw_lift, 526, 476 - paw_lift, "#e6dac1")
        self.oval(494, 452 - paw_lift, 519, 470 - paw_lift, "#faf0dd")
        self.draw_carrot(535, 438, self.bite_animation)

        self.oval(440, 384, 456, 389, "#fff8e6")
        self.oval(490, 383, 506, 388, "#fff8e6")

    def draw_carrot(self, x, y, bite):
        remaining = max(0.08, 1.0 - bite * 0.94)
        height = 68 * remaining
        width = 25 * remaining
        self.oval(x - 7, y + 12, x + 28, y + 26, "#6a6541")
        self.polygon(
            [
                x - 4, y,
                x + 22, y + 2,
                x + 17, y + height * 0.42,
                x + 6, y + height,
            ],
            "#bd532b",
        )
        self.polygon(
            [
                x - 1, y + 2,
                x + 17, y + 4,
                x + 13, y + height * 0.43,
                x + 5, y + height * 0.87,
            ],
            "#ed8134",
        )
        self.polygon(
            [x + 1, y + 5, x + 5, y + 7, x + 8, y + height * 0.73,
             x + 5, y + height * 0.80],
            "#f6a14a",
        )
        for fraction in (0.25, 0.46, 0.66):
            if fraction < remaining:
                stripe_y = y + height * fraction
                self.canvas.create_line(
                    x + 12 - width * 0.28,
                    stripe_y,
                    x + 15 + width * 0.13,
                    stripe_y + 3 * remaining,
                    fill="#bd5b2b",
                    width=2,
                )
        leaf_top = y - 13
        self.polygon(
            [x + 5, y + 5, x - 3, leaf_top, x + 5, y - 2, x + 11, leaf_top - 5],
            "#4f8246",
        )
        self.polygon(
            [x + 8, y + 2, x + 16, leaf_top - 3, x + 15, y + 5],
            "#76a451",
        )

    def draw_hud(self):
        self.rounded_panel(22, 18, WIDTH - 22, 84, "#234538", "")
        self.polygon([47, 34, 61, 38, 56, 55, 50, 59], "#ed8134")
        self.polygon([51, 35, 47, 29, 54, 35], "#6e9850")
        self.polygon([55, 37, 61, 31, 59, 39], "#80a65b")
        self.text(86, 40, "YENEN HAVUÇ", 11, "#c3d7bd", "bold", anchor="w")
        self.text(87, 63, str(self.eaten), 20, "#fff3d4", "bold", anchor="w")

        if self.mode == "level":
            target = LEVEL_TARGETS[self.level - 1]
            self.text(
                WIDTH / 2,
                38,
                f"{self.level}. BÖLÜM",
                13,
                "#f2cb77",
                "bold",
            )
            self.text(
                WIDTH / 2,
                62,
                f"{self.eaten} / {target} havuç",
                15,
                "#ffffff",
                "bold",
            )
            bar_x1, bar_x2 = 390, 570
            self.canvas.create_rectangle(
                bar_x1, 92, bar_x2, 102, fill="#d9dec2", outline=""
            )
            if self.eaten:
                self.canvas.create_rectangle(
                    bar_x1,
                    92,
                    bar_x1 + (bar_x2 - bar_x1) * min(self.eaten / target, 1),
                    102,
                    fill="#f1b85b",
                    outline="",
                )
        else:
            self.text(WIDTH / 2, 42, "SONSUZ BAHÇE", 14, "#f2cb77", "bold")
            self.text(
                WIDTH / 2,
                65,
                "Canın istediği kadar havuç ye!",
                12,
                "#e3ecda",
            )

        self.rounded_panel(WIDTH - 170, 29, WIDTH - 40, 72, "#315745", "")
        self.text(WIDTH - 105, 50, "TOPLAM", 10, "#c3d7bd", "bold")
        self.text(
            WIDTH - 105,
            68,
            f"{self.total_eaten} havuç",
            14,
            "#fff3d4",
            "bold",
        )

        self.rounded_panel(375, 545, 585, 601, "#f1ad49", "")
        self.text(480, 573, "SPACE   •   HAVUÇ YE", 13, "#49361e", "bold")

        self.rounded_panel(811, 551, 937, 600, "#294c3c", "")
        self.text(874, 575, "MENÜ   •   ESC", 11, "#ffffff", "bold")

    def draw_menu(self):
        self.screen = "menu"
        self.buttons = {
            "endless": (72, 437, 396, 510),
            "level": (564, 437, 888, 510),
            "exit": (394, 548, 566, 594),
        }
        self.draw_scene(0)
        self.rounded_panel(117, 40, 843, 314, "#234538")
        self.text(480, 92, "HAVUÇ BAHÇESİ", 40, "#fff1ce", "bold")
        self.text(480, 139, "TAVŞANIN LEZZET MACERASI", 15, "#e8d59a", "bold")
        self.rounded_panel(154, 189, 806, 267, "#315745")
        self.text(
            480,
            214,
            "Havuç bahçesine hoş geldin!",
            17,
            "#ffffff",
            "bold",
        )
        self.text(
            480,
            241,
            "Bir mod seç, SPACE tuşuna bas ya da havuç düğmesine tıkla.",
            12,
            "#d9e6d3",
        )
        for bounds, fill, title, subtitle in (
            (
                self.buttons["endless"],
                "#c66b37",
                "SONSUZ HAVUÇ",
                "Durmadan ye, rekorunu kır!",
            ),
            (
                self.buttons["level"],
                "#e1a843",
                "BÖLÜM BÖLÜM",
                "5 bölümü tamamla!",
            ),
        ):
            x1, y1, x2, y2 = bounds
            self.rounded_panel(x1, y1, x2, y2, fill, "#ffe4ab")
            self.text((x1 + x2) / 2, y1 + 25, title, 17, "#fffaf0", "bold")
            self.text((x1 + x2) / 2, y1 + 51, subtitle, 11, "#fff0d3")
        self.rounded_panel(394, 548, 566, 594, "#315745", "")
        self.text(480, 571, "ÇIKIŞ", 12, "#ffffff", "bold")
        self.text(
            480,
            619,
            "Yumuşacık kulaklar, taptaze havuçlar ve güzel bir bahçe",
            11,
            "#f2ead0",
        )

    def start_game(self, mode):
        if self.tick_id is not None:
            self.root.after_cancel(self.tick_id)
            self.tick_id = None
        self.mode = mode
        self.level = 1
        self.eaten = 0
        self.screen = "playing"
        self.bite_animation = 0.0
        self.chew_animation = 0.0
        self.transition_timer = 0.0
        self.elapsed = 0.0
        self.last_frame = 0.0
        self.tick()

    def eat_carrot(self):
        if self.screen != "playing" or self.bite_animation > 0:
            return
        self.eaten += 1
        self.total_eaten += 1
        self.bite_animation = 0.92
        self.chew_animation = 0.42

        if self.mode == "level" and self.eaten >= LEVEL_TARGETS[self.level - 1]:
            self.screen = "level_complete"
            self.transition_timer = 1.65
        self.draw_game()

    def tick(self):
        self.tick_id = None
        if self.screen == "menu":
            return
        now = self.root.tk.call("clock", "milliseconds") / 1000
        delta = min(now - self.last_frame, 0.1) if self.last_frame else 0.035
        self.last_frame = now
        self.elapsed += delta
        self.bite_animation = max(0.0, self.bite_animation - delta * 2.0)
        self.chew_animation = max(0.0, self.chew_animation - delta)

        if self.screen == "level_complete":
            self.transition_timer -= delta
            if self.transition_timer <= 0:
                if self.level >= len(LEVEL_TARGETS):
                    self.screen = "finished"
                else:
                    self.level += 1
                    self.eaten = 0
                    self.screen = "playing"

        self.draw_game()
        if self.screen != "menu":
            self.tick_id = self.root.after(FRAME_MS, self.tick)

    def draw_game(self):
        self.draw_scene(self.elapsed)
        self.draw_hud()

        if self.screen == "level_complete":
            self.rounded_panel(275, 216, 685, 379, "#264b3a")
            self.text(480, 258, "HARİKA İŞ!", 27, "#f8d78e", "bold")
            self.text(
                480,
                302,
                f"{self.level}. bölümü tamamladın!",
                17,
                "#ffffff",
                "bold",
            )
            self.text(480, 340, "Yeni bölüm hazırlanıyor…", 12, "#d5e4d3")
        elif self.screen == "finished":
            self.rounded_panel(259, 203, 701, 392, "#264b3a")
            self.text(480, 252, "BAHÇENİN YILDIZI!", 26, "#f8d78e", "bold")
            self.text(480, 301, "5 bölümün hepsini bitirdin!", 17, "#ffffff", "bold")
            self.rounded_panel(391, 333, 569, 374, "#e0a544", "")
            self.text(480, 354, "TEKRAR OYNA", 11, "#45351d", "bold")

    def hit(self, bounds, x, y):
        x1, y1, x2, y2 = bounds
        return x1 <= x <= x2 and y1 <= y <= y2

    def on_click(self, event):
        x, y = event.x, event.y
        if self.screen == "menu":
            if self.hit(self.buttons["endless"], x, y):
                self.start_game("endless")
            elif self.hit(self.buttons["level"], x, y):
                self.start_game("level")
            elif self.hit(self.buttons["exit"], x, y):
                self.root.destroy()
        elif self.screen == "playing":
            if 800 <= x <= 950 and 540 <= y <= 612:
                self.draw_menu()
            else:
                self.eat_carrot()
        elif self.screen == "finished" and 380 <= x <= 580 and 325 <= y <= 389:
            self.start_game("level")

    def on_key(self, event):
        key = event.keysym.lower()
        if key == "escape":
            if self.screen == "menu":
                self.root.destroy()
            else:
                self.draw_menu()
        elif key in ("space", "return"):
            if self.screen == "menu":
                self.start_game("endless")
            elif self.screen == "playing":
                self.eat_carrot()
            elif self.screen == "finished":
                self.start_game("level")


def main():
    root = tk.Tk()
    CarrotGarden(root)
    root.mainloop()


if __name__ == "__main__":
    main()
