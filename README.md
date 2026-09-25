# lab1
My university projects and laboratory works.

import tkinter as tk
from tkinter import ttk
from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg
import matplotlib.pyplot as plt

class PicnicApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Теорія ризику — Задача про пікнік")
        self.root.geometry("900x580")
        self.root.configure(bg="#1e1e2e")

        self.u_vkraj_pohano = tk.DoubleVar(value=0)
        self.u_pohano = tk.DoubleVar(value=2)
        self.u_poseredno = tk.DoubleVar(value=5)
        self.u_chudovo = tk.DoubleVar(value=8)
        self.p_rain = tk.DoubleVar(value=0.64)

        self.style = ttk.Style()
        self.style.theme_use('clam')
        self._configure_styles()

        self._setup_ui()
        self.update_calculations()

    def _configure_styles(self):
        self.style.configure(".", background="#1e1e2e", foreground="#cdd6f4", font=("Segoe UI", 10))
        self.style.configure("TLabelframe", background="#181825", borderwidth=1, relief="solid")
        self.style.configure("TLabelframe.Label", background="#181825", foreground="#cba6f7", font=("Segoe UI", 11, "bold"))
        self.style.configure("TLabel", background="#181825", foreground="#cdd6f4")
        self.style.configure("Header.TLabel", font=("Segoe UI", 10, "bold"), foreground="#89b4fa")
        self.style.configure("TScale", background="#181825", troughcolor="#313244", sliderthickness=15)
        self.style.configure("TSeparator", background="#45475a")

    def _setup_ui(self):
        main_frame = ttk.Frame(self.root, padding=15) 
        main_frame.pack(fill=tk.BOTH, expand=True)

        left_frame = ttk.LabelFrame(main_frame, text=" Параметри та Вихідні дані ", padding=15)
        left_frame.pack(side=tk.LEFT, fill=tk.Y, padx=(0, 15))

        ttk.Label(left_frame, text="Ймовірність дощу (p):", style="Header.TLabel").pack(anchor=tk.W, pady=(0, 5))
        p_frame = ttk.Frame(left_frame)#okremii konteiner dlia povzunka i znachennia
        p_frame.pack(fill=tk.X, pady=(0, 15))
        #stvorennia povzunka dlia 
        slider = ttk.Scale(p_frame, from_=0.0, to=1.0, variable=self.p_rain, command=lambda e: self.update_calculations())
        slider.pack(side=tk.LEFT, fill=tk.X, expand=True, padx=(0, 10))

        self.p_val_lbl = tk.Label(p_frame, text="0.64", bg="#313244", fg="#a6e3a1", font=("Consolas", 11, "bold"), width=6, relief="flat")
        self.p_val_lbl.pack(side=tk.RIGHT)

        ttk.Label(left_frame, text="Корисність результатів:", style="Header.TLabel").pack(anchor=tk.W, pady=(5, 5))
        
        utils = [
            ("Вкрай погано", self.u_vkraj_pohano),
            ("Погано", self.u_pohano),
            ("Посередньо", self.u_poseredno),
            ("Чудово", self.u_chudovo)
        ]
        
        for label_text, var in utils:
            row = ttk.Frame(left_frame)
            row.pack(fill=tk.X, pady=3)
            ttk.Label(row, text=label_text, width=14).pack(side=tk.LEFT)
            
            entry = tk.Entry(row, textvariable=var, width=8, bg="#313244", fg="#cdd6f4", insertbackground="white", relief="flat", justify="center", font=("Consolas", 10))
            entry.pack(side=tk.RIGHT)
            #koli vvedene znachennia zminyuiut'sia, avtomatychno perevynno vykonuiut'sia rozrakhunky
            entry.bind("<KeyRelease>", lambda e: self.update_calculations())
        
        ttk.Separator(left_frame, orient='horizontal').pack(fill='x', pady=15)

        ttk.Label(left_frame, text="Очікувана корисність:", style="Header.TLabel").pack(anchor=tk.W, pady=(0, 5))
        
        res_forest_frame = ttk.Frame(left_frame)
        res_forest_frame.pack(fill=tk.X, pady=2)
        ttk.Label(res_forest_frame, text="🌲  Пікнік у лісі:").pack(side=tk.LEFT)
        self.u_forest_lbl = ttk.Label(res_forest_frame, text="0.00", font=("Consolas", 11, "bold"))
        self.u_forest_lbl.pack(side=tk.RIGHT)

        res_home_frame = ttk.Frame(left_frame)
        res_home_frame.pack(fill=tk.X, pady=2)
        ttk.Label(res_home_frame, text="🏠  Залишитися вдома:").pack(side=tk.LEFT)
        self.u_home_lbl = ttk.Label(res_home_frame, text="0.00", font=("Consolas", 11, "bold"))
        self.u_home_lbl.pack(side=tk.RIGHT)

        ttk.Separator(left_frame, orient='horizontal').pack(fill='x', pady=15)
        ttk.Label(left_frame, text="Рекомендоване рішення:", style="Header.TLabel").pack(anchor=tk.W, pady=(0, 5))
        
        self.card = tk.Frame(left_frame, background="#313244", padx=10, pady=10)
        self.card.pack(fill=tk.X, pady=5)
        self.decision_lbl = tk.Label(self.card, text="", font=("Segoe UI", 11, "bold"), background="#313244")
        self.decision_lbl.pack(expand=True)

        #prava chastina
        right_frame = ttk.Frame(main_frame)
        right_frame.pack(side=tk.RIGHT, fill=tk.BOTH, expand=True)
        #nastal'noi chastyni stvoriuemo grafik (matplotlib)
        plt.style.use('dark_background')
        self.fig, self.ax = plt.subplots(figsize=(5, 4), dpi=100)
        self.fig.patch.set_facecolor('#181825')
        self.ax.set_facecolor('#181825')
         #pridnannia canvas dlia vstavky grafika v tkinter
        self.canvas = FigureCanvasTkAgg(self.fig, master=right_frame)
        self.canvas.get_tk_widget().pack(fill=tk.BOTH, expand=True)
#rozrahunok
    def calculate_utilities(self, p):
        try:
            u_vp = self.u_vkraj_pohano.get()
            u_p = self.u_pohano.get()
            u_pos = self.u_poseredno.get()
            u_ch = self.u_chudovo.get()
        except tk.TclError:
            return 0, 0

        u_forest = p * u_vp + (1 - p) * u_ch
        u_home = p * u_p + (1 - p) * u_pos
        return u_forest, u_home

    def update_calculations(self):
        p = round(self.p_rain.get(), 2)
        self.p_val_lbl.config(text=f"{p:.2f}")

        u_forest, u_home = self.calculate_utilities(p)

        self.u_forest_lbl.config(text=f"{u_forest:.2f}")
        self.u_home_lbl.config(text=f"{u_home:.2f}")

        if u_forest > u_home:
            bg_color = "#1e3a29"
            self.decision_lbl.config(text="🌲 Їдемо в ліс!", foreground="#a6e3a1", background=bg_color)
            self.card.config(background=bg_color)
        elif u_home > u_forest:
            bg_color = "#1e293b"
            self.decision_lbl.config(text="🏠 Сидимо вдома", foreground="#89b4fa", background=bg_color)
            self.card.config(background=bg_color)
        else:
            bg_color = "#3b331e"
            self.decision_lbl.config(text="⚖️ Рішення рівноцінні", foreground="#f9e2af", background=bg_color)
            self.card.config(background=bg_color)

        self.plot_graph()

    def plot_graph(self):
        self.ax.clear()

        p_values = [i / 100.0 for i in range(101)]
        forest_values = [self.calculate_utilities(p)[0] for p in p_values]
        home_values = [self.calculate_utilities(p)[1] for p in p_values]

        self.ax.plot(p_values, forest_values, label='Ліс', color='#f38ba8', linewidth=2.5)
        self.ax.plot(p_values, home_values, label='Дім', color='#89b4fa', linewidth=2.5)

        current_p = self.p_rain.get()
        self.ax.axvline(x=current_p, color='#a6adc8', linestyle='--', alpha=0.7, label=f'p = {current_p:.2f}')

        self.ax.set_title("Залежність корисності від ймовірності дощу", color="#cdd6f4", fontsize=11, pad=12)
        self.ax.set_xlabel("Ймовірність дощу (p)", color="#a6adc8", fontsize=9)
        self.ax.set_ylabel("Корисність", color="#a6adc8", fontsize=9)
        self.ax.set_xlim(0, 1)
        self.ax.tick_params(colors='#a6adc8', labelsize=8)
        self.ax.grid(True, linestyle=':', alpha=0.3, color='#45475a')
        
        legend = self.ax.legend(loc='upper right', facecolor='#1e1e2e', edgecolor='#45475a')
        plt.setp(legend.get_texts(), color='#cdd6f4')

        self.fig.tight_layout()
        self.canvas.draw()

if __name__ == "__main__":
    root = tk.Tk()
    app = PicnicApp(root)
    root.mainloop()
    <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/6cf23c6c-25f8-444f-afc4-9662ebd4b39e" />
