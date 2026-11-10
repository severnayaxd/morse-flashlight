import tkinter as tk

# --- Азбука Морзе ---
MORSE = {
    'А': '.-',    'Б': '-...',  'В': '.--',   'Г': '--.',
    'Д': '-..',   'Е': '.',     'Ж': '...-',  'З': '--..',
    'И': '..',    'Й': '.---',  'К': '-.-',   'Л': '.-..',
    'М': '--',    'Н': '-.',    'О': '---',   'П': '.--.',
    'Р': '.-.',   'С': '...',   'Т': '-',     'У': '..-',
    'Ф': '..-.',  'Х': '....',  'Ц': '-.-.',  'Ч': '---.',
    'Ш': '----',  'Щ': '--.-',  'Ъ': '--.--', 'Ы': '-.--',
    'Ь': '-..-',  'Э': '..-..', 'Ю': '..--',  'Я': '.-.-',
    '0': '-----', '1': '.----', '2': '..---', '3': '...--',
    '4': '....-', '5': '.....', '6': '-....', '7': '--...',
    '8': '---..', '9': '----.',
    ' ': ' '
}

DOT_TIME  = 0.2
DASH_TIME = DOT_TIME * 3
SYMBOL_GAP = DOT_TIME
LETTER_GAP = DOT_TIME * 3
WORD_GAP   = DOT_TIME * 7


class MorseFlashlight:
    def __init__(self, root):
        self.root = root
        self.root.title("Азбука Морзе — фонарик")
        self.root.configure(bg="black")
        self.root.geometry("600x700")

        self.entry = tk.Entry(root, font=("Arial", 18), justify="center",
                              bg="#222", fg="white", insertbackground="white")
        self.entry.pack(pady=20, padx=40, fill="x")
        self.entry.insert(0, "ПРИВЕТ")

        self.btn = tk.Button(root, text="Мигнуть!", font=("Arial", 14),
                             command=self.start_morse, bg="#444", fg="white",
                             activebackground="#666")
        self.btn.pack(pady=10)

        self.canvas = tk.Canvas(root, bg="black", highlightthickness=0)
        self.canvas.pack(fill="both", expand=True)

        self.status = tk.Label(root, text="", font=("Arial", 16),
                               bg="black", fg="yellow")
        self.status.pack(pady=10)

        self.is_playing = False
        self.draw_flashlight(light_on=False)

    def draw_flashlight(self, light_on=False):
        self.canvas.delete("all")
        w = self.canvas.winfo_width()
        h = self.canvas.winfo_height()
        
        # Если окно еще не отрисовалось, ставим дефолтные значения
        if w <= 1 or h <= 1:
            w, h = 600, 500

        # === ЦЕНТР ЭКРАНА ===
        cx = w // 2
        cy = h // 2  # Ищем середину по вертикали

        # Размеры фонарика
        body_w = 70
        body_h = 90
        
        # Смещаем фонарик так, чтобы его ЛИНЗА была ровно по центру (cy)
        body_bottom = cy - 10      # Низ корпуса чуть выше центра
        body_top = body_bottom - body_h
        handle_top = body_top - 40 # Ручка идет вверх

        # --- Рисуем фонарик ---
        # Ручка
        self.canvas.create_rectangle(cx - 12, handle_top, cx + 12, body_top,
                                     fill="#666", outline="#333")
        # Корпус
        body_color = "#FFD700" if light_on else "#8B8B00"
        self.canvas.create_rectangle(cx - body_w // 2, body_top,
                                     cx + body_w // 2, body_bottom,
                                     fill=body_color, outline="#FFA500", width=3)
        # Линза
        lens_color = "#FFFFAA" if light_on else "#555500"
        self.canvas.create_oval(cx - body_w // 2 + 10, body_bottom - 15,
                                cx + body_w // 2 - 10, body_bottom + 5,
                                fill=lens_color, outline="")

        # --- Луч вниз ---
        if light_on:
            # Луч начинается от линзы и расширяется вниз
            beam_top = body_bottom + 5
            beam_bottom = h # Доходит до самого низа экрана
            
            # Внешний (полупрозрачный на вид) конус
            self.canvas.create_polygon(
                cx - 30, beam_top,
                cx + 30, beam_top,
                cx + 250, beam_bottom,
                cx - 250, beam_bottom,
                fill="#FFF8B0", outline=""
            )
            # Внутренний (яркий) конус
            self.canvas.create_polygon(
                cx - 15, beam_top,
                cx + 15, beam_top,
                cx + 100, beam_bottom,
                cx - 100, beam_bottom,
                fill="#FFFFE0", outline=""
            )

    def start_morse(self):
        if self.is_playing:
            return
        text = self.entry.get().strip().upper()
        if not text:
            return
        self.is_playing = True
        self.btn.config(state="disabled")
        self.play_sequence(text)

    def play_sequence(self, text):
        steps = []
        for i, ch in enumerate(text):
            code = MORSE.get(ch)
            if code is None:
                continue
            if code == ' ':
                steps.append(('pause', WORD_GAP))
                continue
            for j, sym in enumerate(code):
                if sym == '.':
                    steps.append(('on', DOT_TIME))
                else:
                    steps.append(('on', DASH_TIME))
                if j < len(code) - 1:
                    steps.append(('off', SYMBOL_GAP))
            if i < len(text) - 1:
                steps.append(('off', LETTER_GAP))

        self._run_steps(steps, 0)

    def _run_steps(self, steps, idx):
        if idx >= len(steps):
            self.draw_flashlight(light_on=False)
            self.status.config(text="")
            self.is_playing = False
            self.btn.config(state="normal")
            return

        action, duration = steps[idx]
        if action == 'on':
            self.draw_flashlight(light_on=True)
            self.status.config(text="● СВЕТ")
        else:
            self.draw_flashlight(light_on=False)
            self.status.config(text="")

        self.root.after(int(duration * 1000),
                        lambda: self._run_steps(steps, idx + 1))


if __name__ == "__main__":
    root = tk.Tk()
    app = MorseFlashlight(root)
    
    # Перерисовка при изменении размера окна
    def on_resize(event):
        # Небольшая задержка, чтобы Canvas успел обновить свои размеры
        root.after(10, lambda: app.draw_flashlight(light_on=app.is_playing and app.status.cget("text") == "● СВЕТ"))
        
    root.bind("<Configure>", on_resize)
    root.mainloop()