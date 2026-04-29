import tkinter as tk
from tkinter import ttk, messagebox
import json
import os
from datetime import datetime

class TrainingPlanner:
    def __init__(self, root):
        self.root = root
        self.root.title("Training Planner")
        self.root.geometry("800x600")

        # Данные тренировок
        self.trainings = []
        self.load_data()

        self.setup_ui()

    def setup_ui(self):
        # Фрейм для формы ввода
        input_frame = ttk.Frame(self.root)
        input_frame.pack(pady=10, padx=10, fill="x")

        # Поле Дата
        ttk.Label(input_frame, text="Дата (ГГГГ-ММ-ДД):").grid(row=0, column=0, sticky="w")
        self.date_entry = ttk.Entry(input_frame)
        self.date_entry.grid(row=0, column=1, padx=5)

        # Поле Тип тренировки
        ttk.Label(input_frame, text="Тип тренировки:").grid(row=1, column=0, sticky="w")
        self.type_entry = ttk.Combobox(input_frame,
                                       values=["Кардио", "Силовая", "Йога", "Плавание", "Бег"])
        self.type_entry.grid(row=1, column=1, padx=5)

        # Поле Длительность
        ttk.Label(input_frame, text="Длительность (мин):").grid(row=2, column=0, sticky="w")
        self.duration_entry = ttk.Entry(input_frame)
        self.duration_entry.grid(row=2, column=1, padx=5)

        # Кнопка Добавить
        add_btn = ttk.Button(input_frame, text="Добавить тренировку",
                            command=self.add_training)
        add_btn.grid(row=3, column=0, columnspan=2, pady=10)

        # Фрейм для фильтрации
        filter_frame = ttk.Frame(self.root)
        filter_frame.pack(pady=5, padx=10, fill="x")

        ttk.Label(filter_frame, text="Фильтр по типу:").pack(side="left")
        self.filter_type = ttk.Combobox(filter_frame,
                                  values=["Все", "Кардио", "Силовая", "Йога", "Плавание", "Бег"],
                                  state="readonly")
        self.filter_type.set("Все")
        self.filter_type.pack(side="left", padx=5)
        self.filter_type.bind("<<ComboboxSelected>>", self.apply_filters)

        ttk.Label(filter_frame, text="Фильтр по дате:").pack(side="left", padx=(20, 5))
        self.filter_date = ttk.Entry(filter_frame)
        self.filter_date.pack(side="left", padx=5)
        self.filter_date.bind("<KeyRelease>", self.apply_filters)

        # Таблица тренировок
        columns = ("Дата", "Тип", "Длительность")
        self.tree = ttk.Treeview(self.root, columns=columns, show="headings")
        for col in columns:
            self.tree.heading(col, text=col)
            self.tree.column(col, width=200)
        self.tree.pack(pady=10, padx=10, fill="both", expand=True)

        # Обновление таблицы
        self.update_table()

    def validate_input(self):
        """Проверка корректности ввода"""
        # Проверка даты
        try:
            date_obj = datetime.strptime(self.date_entry.get(), "%Y-%m-%d")
            if date_obj > datetime.now():
                messagebox.showerror("Ошибка", "Дата не может быть в будущем!")
                return False
        except ValueError:
            messagebox.showerror("Ошибка", "Неверный формат даты! Используйте ГГГГ-ММ-ДД")
            return False

        # Проверка длительности
        try:
            duration = float(self.duration_entry.get())
            if duration <= 0:
                messagebox.showerror("Ошибка", "Длительность должна быть положительным числом!")
                return False
        except ValueError:
            messagebox.showerror("Ошибка", "Длительность должна быть числом!")
            return False

        return True

    def add_training(self):
        """Добавление новой тренировки"""
        if not self.validate_input():
            return

        training = {
            "date": self.date_entry.get(),
            "type": self.type_entry.get(),
            "duration": float(self.duration_entry.get())
        }

        self.trainings.append(training)
        self.save_data()
        self.update_table()

        # Очистка полей ввода
        self.date_entry.delete(0, tk.END)
        self.type_entry.set("")
        self.duration_entry.delete(0, tk.END)

        messagebox.showinfo("Успех", "Тренировка добавлена!")

    def update_table(self):
        """Обновление таблицы с учётом фильтров"""
        for item in self.tree.get_children():
            self.tree.delete(item)

        filtered_trainings = self.apply_filter_logic()

        for training in filtered_trainings:
            self.tree.insert("", "end", values=(
                training["date"],
                training["type"],
                f"{training['duration']} мин"
            ))

    def apply_filter_logic(self):
        """Логика фильтрации данных"""
        filtered = self.trainings

        # Фильтр по типу
        selected_type = self.filter_type.get()
        if selected_type != "Все":
            filtered = [t for t in filtered if t["type"] == selected_type]

        # Фильтр по дате
        date_filter = self.filter_date.get().strip()
        if date_filter:
            filtered = [t for t in filtered if date_filter in t["date"]]

        return filtered

    def apply_filters(self, event=None):
        """Применение фильтров при изменении"""
        self.update_table()

    def save_data(self):
        """Сохранение данных в JSON"""
        with open("trainings.json", "w", encoding="utf-8") as f:
            json.dump(self.trainings, f, ensure_ascii=False, indent=2)

    def load_data(self):
        """Загрузка данных из JSON"""
        if os.path.exists("trainings.json"):
            try:
                with open("trainings.json", "r", encoding="utf-8") as f:
                    self.trainings = json.load(f)
            except (json.JSONDecodeError, IOError):
                self.trainings = []

# Запуск приложения
if __name__ == "__main__":
    root = tk.Tk()
    app = TrainingPlanner(root)
    root.mainloop()
