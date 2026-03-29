import tkinter as tk
from tkinter import ttk
import math
import colorsys
import random
import re
import webbrowser
from datetime import datetime
from fractions import Fraction

class OmegaUltraV17:
    def __init__(self, root):
        self.root = root
        self.root.title("HalexPlay Win11 OMEGA ULTRA v17.0 Enhanced")
        self.root.geometry("8500x980") # Окно адаптируется под контент
        
        self.themes = {
            "white": {"bg": "#F3F3F3", "btn": "#FFFFFF", "fg": "#000000", "acc": "#0067C0", "disp": "#FFFFFF"},
            "dark": {"bg": "#202020", "btn": "#333333", "fg": "#FFFFFF", "acc": "#4CC2FF", "disp": "#2D2D2D"},
            "orange": {"bg": "#FF9800", "btn": "#FFF3E0", "fg": "#E65100", "acc": "#F57C00", "disp": "#FFFFFF"},
            "green": {"bg": "#2E7D32", "btn": "#E8F5E9", "fg": "#1B5E20", "acc": "#4CAF50", "disp": "#FFFFFF"},
            "red": {"bg": "#C62828", "btn": "#FFEBEE", "fg": "#B71C1C", "acc": "#F44336", "disp": "#FFFFFF"},
            "rgb": {"btn": "#000000", "fg": "#FFFFFF", "acc": "#FFFFFF", "disp": "#111111"},
            "glitch": {"bg": "#FFFFFF", "btn": "#F9F9F9", "fg": "#000000", "acc": "#000000", "disp": "#FFFFFF"},
            "cloud": {"btn": "#111111", "fg": "#FFFFFF", "acc": "#FFFFFF", "disp": "#000000"}
        }
        
        self.current_theme = "dark"
        self.hue = 0.0
        self.is_animated = False
        self.bg_widgets, self.btn_widgets, self.text_widgets, self.entry_widgets = [], [], [], []

        self.setup_ui()
        self.update_clock()
        self.apply_theme("dark")

    def setup_ui(self):
        top = tk.Frame(self.root); top.pack(fill="x", padx=15, pady=10); self.bg_widgets.append(top)
        self.lbl_clock = tk.Label(top, font=("Segoe UI", 11, "bold")); self.lbl_clock.pack(side="left"); self.text_widgets.append(self.lbl_clock)
        
        t_frame = tk.Frame(top); t_frame.pack(side="right"); self.bg_widgets.append(t_frame)
        th_list = [("white", "⬜"), ("dark", "⬛"), ("orange", "🟧"), ("green", "🟩"), ("red", "🟥"), ("rgb", "🌈"), ("glitch", "👾"), ("cloud", "☁️")]
        for t_id, icon in th_list:
            b = tk.Button(t_frame, text=icon, command=lambda x=t_id: self.apply_theme(x), relief="flat", font=12, padx=5)
            b.pack(side="left", padx=2); self.btn_widgets.append(b)

        self.nb = ttk.Notebook(self.root); self.nb.pack(expand=True, fill="both", padx=10, pady=5)
        self.tab_eng = tk.Frame(self.nb); self.nb.add(self.tab_eng, text="Инженерный")
        self.tab_frac = tk.Frame(self.nb); self.nb.add(self.tab_frac, text="Дроби (3x)")
        self.tab_conv = tk.Frame(self.nb); self.nb.add(self.tab_conv, text="Конвертер")
        self.tab_solve = tk.Frame(self.nb); self.nb.add(self.tab_solve, text="Уравнение X")
        self.tab_about = tk.Frame(self.nb); self.nb.add(self.tab_about, text="О нас")
        
        self.bg_widgets.extend([self.tab_eng, self.tab_frac, self.tab_conv, self.tab_solve, self.tab_about, self.root])
        self.init_eng(); self.init_frac(); self.init_conv(); self.init_solve(); self.init_about()

    def update_clock(self):
        self.lbl_clock.config(text=datetime.now().strftime("%d.%m.%Y  %H:%M:%S"))
        self.root.after(1000, self.update_clock)

    def init_about(self):
        tk.Label(self.tab_about, text="HalexPlay OMEGA ULTRA", font=("Segoe UI", 24, "bold")).pack(pady=(50, 10))
        tk.Label(self.tab_about, text="Версия: 17.0 (Stable)", font=("Segoe UI", 14)).pack()
        
        btn_vk = tk.Button(self.tab_about, text="ОТКРЫТЬ VK: halexplay", font=("Segoe UI", 14, "bold"), 
                            fg="white", bg="#0077FF", padx=20, pady=10, command=lambda: webbrowser.open("https://vk.com/halexplay"))
        btn_vk.pack(pady=30)
        
        # Твоя надпись внизу
        lbl_footer = tk.Label(self.tab_about, text="Покачто это последняя версия (Далее будут обновления)!", 
                              font=("Segoe UI", 11, "italic"), fg="gray")
        lbl_footer.pack(side="bottom", pady=20)
        self.text_widgets.append(lbl_footer)

    def init_solve(self):
        tk.Label(self.tab_solve, text="Примеры: (10-x)*5=60 или 5+5-x=10", font=("Segoe UI", 11)).pack(pady=10)
        self.solve_in = tk.Entry(self.tab_solve, font=("Segoe UI", 22), justify="center", width=35)
        self.solve_in.pack(pady=10); self.entry_widgets.append(self.solve_in)
        tk.Button(self.tab_solve, text="РЕШИТЬ ПО ШАГАМ", font=("Segoe UI", 12, "bold"), command=self.do_solve).pack(pady=10)
        self.solve_res = tk.Text(self.tab_solve, font=("Segoe UI", 12), height=15, width=75, state="disabled", relief="flat")
        self.solve_res.pack(pady=10, padx=20)
        
        lbl_beta = tk.Label(self.tab_solve, text="Эта вкладка \"Уравнения X\" находится в бета-тестировании!", 
                            font=("Segoe UI", 9, "italic"), fg="gray")
        lbl_beta.pack(side="bottom", pady=10); self.text_widgets.append(lbl_beta)

    def do_solve(self):
        raw = self.solve_in.get().replace(" ", "").lower()
        self.solve_res.config(state="normal"); self.solve_res.delete("1.0", tk.END)
        try:
            if '=' not in raw: raise ValueError
            left, right = raw.split('=')
            res = float(right)
            curr = left
            step = 1
            for _ in range(3):
                m_simp = re.search(r"(\d+[\+\-\*\/]\d+)", curr)
                if m_simp and 'x' not in m_simp.group(0):
                    val = eval(m_simp.group(0))
                    new_curr = curr.replace(m_simp.group(0), str(val), 1)
                    self.solve_res.insert(tk.END, f"{step}. Упрощаем {m_simp.group(0)}: {new_curr} = {res}\n")
                    curr = new_curr
                    step += 1
            while 'x' in curr and curr != 'x':
                m_br = re.match(r"\((.+)\)([\*\/\+\-])(\d+\.?\d*)", curr)
                m_br_inv = re.match(r"(\d+\.?\d*)([\*\/\+\-])\((.+)\)", curr)
                if m_br:
                    inner, op, val = m_br.group(1), m_br.group(2), float(m_br.group(3))
                    if op == '*': res /= val
                    elif op == '/': res *= val
                    elif op == '+': res -= val
                    elif op == '-': res += val
                    self.solve_res.insert(tk.END, f"{step}. Убираем скобки ({op} {val}): {inner} = {res}\n")
                    curr = inner
                elif m_br_inv:
                    val, op, inner = float(m_br_inv.group(1)), m_br_inv.group(2), m_br_inv.group(3)
                    if op == '*': res /= val
                    elif op == '+': res -= val
                    elif op == '-': res = val - res
                    elif op == '/': res = val / res
                    self.solve_res.insert(tk.END, f"{step}. Убираем скобки ({val} {op}): {inner} = {res}\n")
                    curr = inner
                else:
                    match = re.search(r"([\+\-\*\/])", curr)
                    if match:
                        op = match.group(0)
                        parts = curr.split(op, 1)
                        if 'x' in parts[1]:
                            val = eval(parts[0])
                            if op == '+': res -= val
                            elif op == '-': res = val - res
                            elif op == '*': res /= val
                            elif op == '/': res = val / res
                            curr = parts[1]
                        else:
                            val = eval(parts[1])
                            if op == '+': res -= val
                            elif op == '-': res += val
                            elif op == '*': res /= val
                            elif op == '/': res *= val
                            curr = parts[0]
                        self.solve_res.insert(tk.END, f"{step}. Переносим {val}: x = {res}\n")
                    else: break
                step += 1
                if step > 15: break
            self.solve_res.insert(tk.END, f"\nИТОГ: x = {res}")
        except:
            self.solve_res.insert(tk.END, "ОШИБКА! Пример: (10-x)*5=60")
        self.solve_res.config(state="disabled")

    def init_eng(self):
        self.e_disp = tk.Entry(self.tab_eng, font=("Segoe UI Semibold", 25), justify="right", bd=10, relief="flat")
        self.e_disp.pack(fill="x", padx=20, pady=20); self.e_disp.insert(0, "0"); self.entry_widgets.append(self.e_disp)
        g = tk.Frame(self.tab_eng); g.pack(expand=True, fill="both", padx=20); self.bg_widgets.append(g)
        btns = [['sin','cos','log','ln','exp'],['sqrt','(',')','[',']'],['{','}','^','pi','C'],['7','8','9','/','DEL'],['4','5','6','*','-'],['1','2','3','-','+'],['+/-','0','.','=','']]
        for r, row in enumerate(btns):
            for c, txt in enumerate(row):
                btn = tk.Button(g, text=txt, font=("Segoe UI", 11, "bold"), relief="flat", command=lambda t=txt: self.eng_p(t))
                btn.grid(row=r, column=c, sticky="nsew", padx=2, pady=2)
                self.btn_widgets.append(btn); g.columnconfigure(c, weight=1); g.rowconfigure(r, weight=1)

    def eng_p(self, k):
        cur = self.e_disp.get()
        if cur == "ОШИБКА!!!": cur = "0"
        try:
            if k == "C": self.e_disp.delete(0, tk.END); self.e_disp.insert(0, "0")
            elif k == "DEL": v = cur[:-1]; self.e_disp.delete(0, tk.END); self.e_disp.insert(0, v if v else "0")
            elif k == "+/-": self.e_disp.insert(0, "-") if not cur.startswith("-") else self.e_disp.delete(0)
            elif k == "=":
                ex = cur.replace('^','**').replace('[','(').replace(']',')').replace('{','(').replace('}',')')
                res = eval(ex, {"__builtins__":None}, {"sin":math.sin,"cos":math.cos,"log":math.log10,"ln":math.log,"sqrt":math.sqrt,"exp":math.exp,"pi":math.pi})
                self.e_disp.delete(0, tk.END); self.e_disp.insert(0, str(res))
            else:
                if cur == "0": self.e_disp.delete(0, tk.END)
                self.e_disp.insert(tk.END, k)
        except: self.e_disp.delete(0, tk.END); self.e_disp.insert(0, "ОШИБКА!!!")

    def init_frac(self):
        c = tk.Frame(self.tab_frac); c.pack(pady=30); self.bg_widgets.append(c)
        def cf(col):
            f = tk.Frame(c); f.grid(row=0, column=col); self.bg_widgets.append(f)
            w, n, d = tk.Entry(f, width=3, font=20), tk.Entry(f, width=4, font=14), tk.Entry(f, width=4, font=14)
            w.grid(row=0, column=0, rowspan=2, padx=2); n.grid(row=0, column=1); d.grid(row=1, column=1)
            self.entry_widgets.extend([w, n, d]); return (w, n, d)
        self.f1, self.op1 = cf(0), ttk.Combobox(c, values=["+", "-", "*", "/", "%"], width=3)
        self.op1.grid(row=0, column=1, padx=5); self.op1.set("+")
        self.f2, self.op2 = cf(2), ttk.Combobox(c, values=["+", "-", "*", "/", "%"], width=3)
        self.op2.grid(row=0, column=3, padx=5); self.op2.set("+")
        self.f3 = cf(4)
        tk.Button(self.tab_frac, text="ВЫЧИСЛИТЬ", font=("Segoe UI", 12, "bold"), command=self.do_f).pack(pady=20)
        self.fr_res = tk.Label(self.tab_frac, text="Результат: ", font=("Segoe UI", 22)); self.fr_res.pack(); self.text_widgets.append(self.fr_res)

    def do_f(self):
        try:
            def p(f):
                w, n, d = int(f[0].get() or 0), int(f[1].get() or 0), int(f[2].get() or 1)
                s = -1 if str(f[0].get()).startswith('-') else 1
                return Fraction(abs(w)*d + n, d) * s
            def calc(a, b, op):
                if op == "+": return a + b
                if op == "-": return a - b
                if op == "*": return a * b
                if op == "/": return a / b
                return a % b
            res = calc(calc(p(self.f1), p(self.f2), self.op1.get()), p(self.f3), self.op2.get())
            wh = abs(res.numerator) // res.denominator
            nu = abs(res.numerator) % res.denominator
            sg = "-" if res < 0 else ""
            self.fr_res.config(text=f"Результат: {sg}{wh if wh!=0 or nu==0 else ''} {f'{nu}/{res.denominator}' if nu!=0 else ''}")
        except: self.fr_res.config(text="ОШИБКА!!!")

    # --- УЛУЧШЕННЫЙ КОНВЕРТЕР ---
    def init_conv(self):
        self.d = {
            "Масса": {"Тонна": 1000, "Центнер": 100, "Кг": 1, "Грамм": 0.001, "Мг": 1e-6},
            "Длина": {"Км": 1000, "Метр": 1, "См": 0.01, "Мм": 0.001, "Миля": 1609.34, "Дюйм": 0.0254},
            "Скорость": {"Км/ч": 1, "М/с": 3.6, "Узел": 1.852, "Мах": 1225},
            "Время": {"День": 86400, "Час": 3600, "Мин": 60, "Сек": 1, "Неделя": 604800},
            "Давление": {"Паскаль": 1, "Бар": 100000, "Атм": 101325, "мм рт.ст.": 133.322},
            "Объем": {"Литр": 1, "М3": 1000, "Мл": 0.001, "Галлон": 3.785},
            "Температура": {"Цельсий": "c", "Фаренгейт": "f", "Кельвин": "k"},
            "IT": {"ТБ": 1e12, "ГБ": 1e9, "МБ": 1e6, "КБ": 1e3, "Б": 1}
        }
        self.cat = ttk.Combobox(self.tab_conv, values=list(self.d.keys()), state="readonly", font=12); self.cat.pack(pady=20); self.cat.current(0)
        self.cat.bind("<<ComboboxSelected>>", self.upd_u)
        
        f = tk.Frame(self.tab_conv); f.pack(pady=10); self.bg_widgets.append(f)
        self.c_in = tk.Entry(f, width=12, font=("Segoe UI", 16), justify="center")
        self.u1 = ttk.Combobox(f, width=12, font=12, state="readonly")
        self.u2 = ttk.Combobox(f, width=12, font=12, state="readonly")
        
        self.c_in.grid(row=0, column=0, padx=5)
        self.u1.grid(row=0, column=1, padx=5)
        tk.Label(f, text="➡", font=20).grid(row=0, column=2, padx=5)
        self.u2.grid(row=0, column=3, padx=5)
        
        tk.Button(self.tab_conv, text="КОНВЕРТИРОВАТЬ", font=("Segoe UI", 12, "bold"), command=self.do_c).pack(pady=20)
        self.c_res = tk.Label(self.tab_conv, text="---", font=("Segoe UI", 26, "bold")); self.c_res.pack(); self.text_widgets.append(self.c_res)
        self.upd_u()

    def upd_u(self, e=None):
        v = list(self.d[self.cat.get()].keys())
        self.u1.config(values=v); self.u1.current(0)
        self.u2.config(values=v); self.u2.current(1 if len(v)>1 else 0)

    def do_c(self):
        try:
            cat = self.cat.get()
            val = float(self.c_in.get())
            unit1, unit2 = self.u1.get(), self.u2.get()
            
            if cat == "Температура":
                # Специальная логика для температур
                u1, u2 = self.d[cat][unit1], self.d[cat][unit2]
                # Сначала в Цельсий
                if u1 == 'f': c = (val - 32) * 5/9
                elif u1 == 'k': c = val - 273.15
                else: c = val
                # Из Цельсия в цель
                if u2 == 'f': r = (c * 9/5) + 32
                elif u2 == 'k': r = c + 273.15
                else: r = c
            else:
                # Обычная логика коэффициентов
                r = val * (self.d[cat][unit1] / self.d[cat][unit2])
            
            self.c_res.config(text=f"{r:.4f}".rstrip('0').rstrip('.'))
        except: self.c_res.config(text="ОШИБКА!!!")

    def apply_theme(self, name):
        self.current_theme = name
        self.is_animated = (name in ["rgb", "glitch", "cloud"])
        t = self.themes[name]
        if not self.is_animated:
            for w in self.bg_widgets: w.config(bg=t["bg"])
            for w in self.text_widgets: w.config(bg=t["bg"], fg=t["fg"])
            for w in self.btn_widgets: w.config(bg=t["btn"], fg=t["fg"])
            for w in self.entry_widgets: w.config(bg=t["disp"], fg=t["fg"])
        else:
            if name == "rgb" or name == "cloud":
                for w in self.btn_widgets: w.config(bg="black", fg="white")
            elif name == "glitch":
                for w in self.bg_widgets: w.config(bg="white")
                for w in self.text_widgets: w.config(bg="white", fg="black")
                for w in self.btn_widgets: w.config(bg="#F9F9F9", fg="black")
            self.run_animation()

    def run_animation(self):
        if not self.is_animated: return
        if self.current_theme == "rgb":
            rgb = colorsys.hls_to_rgb(self.hue, 0.5, 1.0)
            c = '#%02x%02x%02x' % (int(rgb[0]*255), int(rgb[1]*255), int(rgb[2]*255))
            for w in self.bg_widgets: w.config(bg=c)
            self.hue += 0.005
        elif self.current_theme == "cloud":
            val = int((math.sin(self.hue * 5) + 1) * 127)
            c = '#%02x%02x%02x' % (val, val, val)
            for w in self.bg_widgets: w.config(bg=c)
            self.hue += 0.01
        elif self.current_theme == "glitch":
            for b in self.btn_widgets:
                if random.random() > 0.85:
                    gray = random.randint(200, 255)
                    c_bg = '#%02x%02x%02x' % (gray, gray, gray)
                    c_fg = random.choice(["#222", "#000", "#FF00FF", "#00FFFF", "#555"])
                    b.config(bg=c_bg, fg=c_fg)
                else: b.config(bg="#F9F9F9", fg="black")
            self.hue += 0.1
        self.root.after(60, self.run_animation)

if __name__ == "__main__":
    root = tk.Tk(); app = OmegaUltraV17(root); root.mainloop()
