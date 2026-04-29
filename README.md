# xcvfdg123
крутое
#Шаг 1. Базовая структура приложения
import tkinter as tk
from tkinter import ttk, messagebox
import json
from datetime import datetime

class ExpenseTracker:
    def __init__(self, root):
        self.root = root
        self.root.title("Expense Tracker")
        self.expenses = []
        self.load_data()
        self.create_widgets()

    def create_widgets(self):
        # Поля ввода
        tk.Label(self.root, text="Сумма:").grid(row=0, column=0, padx=5, pady=5)
        self.amount_entry = tk.Entry(self.root)
        self.amount_entry.grid(row=0, column=1, padx=5, pady=5)

        tk.Label(self.root, text="Категория:").grid(row=1, column=0, padx=5, pady=5)
        self.category_var = tk.StringVar()
        categories = ["Еда", "Транспорт", "Развлечения", "Жильё", "Другое"]
        self.category_combo = ttk.Combobox(self.root, textvariable=self.category_var, values=categories)
        self.category_combo.grid(row=1, column=1, padx=5, pady=5)

        tk.Label(self.root, text="Дата (ГГГГ-ММ-ДД):").grid(row=2, column=0, padx=5, pady=5)
        self.date_entry = tk.Entry(self.root)
        self.date_entry.grid(row=2, column=1, padx=5, pady=5)

        # Кнопка добавления
        self.add_button = tk.Button(self.root, text="Добавить расход", command=self.add_expense)
        self.add_button.grid(row=3, column=0, columnspan=2, pady=10)

        # Таблица
        self.tree = ttk.Treeview(self.root, columns=("Сумма", "Категория", "Дата"), show="headings")
        self.tree.heading("Сумма", text="Сумма")
        self.tree.heading("Категория", text="Категория")
        self.tree.heading("Дата", text="Дата")
        self.tree.grid(row=4, column=0, columnspan=2, padx=5, pady=5)

        # Подсчёт суммы за период
        tk.Label(self.root, text="Начало периода (ГГГГ-ММ-ДД):").grid(row=5, column=0, padx=5, pady=5)
        self.start_date_entry = tk.Entry(self.root)
        self.start_date_entry.grid(row=5, column=1, padx=5, pady=5)

        tk.Label(self.root, text="Конец периода (ГГГГ-ММ-ДД):").grid(row=6, column=0, padx=5, pady=5)
        self.end_date_entry = tk.Entry(self.root)
        self.end_date_entry.grid(row=6, column=1, padx=5, pady=5)

        self.calculate_button = tk.Button(self.root, text="Посчитать сумму за период", command=self.calculate_period_sum)
        self.calculate_button.grid(row=7, column=0, columnspan=2, pady=5)

        self.sum_label = tk.Label(self.root, text="Общая сумма за период: 0")
        self.sum_label.grid(row=8, column=0, columnspan=2, pady=5)

        # Фильтры
        tk.Label(self.root, text="Фильтр по категории:").grid(row=9, column=0, padx=5, pady=5)
        self.filter_category_var = tk.StringVar()
        self.filter_category_combo = ttk.Combobox(self.root, textvariable=self.filter_category_var, values=["Все"] + categories)
        self.filter_category_combo.set("Все")
        self.filter_category_combo.grid(row=9, column=1, padx=5, pady=5)

        tk.Label(self.root, text="Фильтр по дате (ГГГГ-ММ-ДД):").grid(row=10, column=0, padx=5, pady=5)
        self.filter_date_entry = tk.Entry(self.root)
        self.filter_date_entry.grid(row=10, column=1, padx=5, pady=5)

        self.filter_button = tk.Button(self.root, text="Применить фильтры", command=self.apply_filters)
        self.filter_button.grid(row=11, column=0, columnspan=2, pady=5)
#Шаг 2. Добавление расходов с проверкой данных
    def add_expense(self):
        try:
            amount = float(self.amount_entry.get())
            if amount <= 0:
                messagebox.showerror("Ошибка", "Сумма должна быть положительным числом")
                return

            category = self.category_var.get()
            if not category:
                messagebox.showerror("Ошибка", "Выберите категорию")
                return

            date_str = self.date_entry.get()
            try:
                date = datetime.strptime(date_str, "%Y-%m-%d")
            except ValueError:
                messagebox.showerror("Ошибка", "Неверный формат даты. Используйте ГГГГ-ММ-ДД")
                return

            expense = {
                "amount": amount,
                "category": category,
                "date": date_str
            }
            self.expenses.append(expense)
            self.update_table()
            self.save_data()

            # Очищаем поля
            self.amount_entry.delete(0, tk.END)
            self.date_entry.delete(0, tk.END)

        except ValueError:
            messagebox.showerror("Ошибка", "Некорректный ввод суммы")
#Шаг 3. Отображение и обновление таблицы
    def update_table(self, expenses=None):
        for item in self.tree.get_children():
            self.tree.delete(item)

        if expenses is None:
            expenses = self.expenses

        for expense in expenses:
            self.tree.insert("", "end", values=(expense["amount"], expense["category"], expense["date"]))
#Шаг 4. Сохранение и загрузка данных в JSON
    def save_data(self):
        with open("expenses.json", "w", encoding="utf-8") as f:
            json.dump(self.expenses, f, ensure_ascii=False, indent=4)

    def load_data(self):
        try:
            with open("expenses.json", "r", encoding="utf-8") as f:
                self.expenses = json.load(f)
            self.update_table()
        except FileNotFoundError:
            self.expenses = []
#Шаг 5. Подсчёт суммы за период
    def calculate_period_sum(self):
        start_str = self.start_date_entry.get()
        end_str = self.end_date_entry.get()

        try:
            start_date = datetime.strptime(start_str, "%Y-%m-%d") if start_str else None
            end_date = datetime.strptime(end_str, "%Y-%m-%d") if end_str else None
        except ValueError:
            messagebox.showerror("Ошибка", "Неверный формат даты")
            return

        total = 0
        for expense in self.expenses:
            exp_date = datetime.strptime(expense["date"], "%Y-%m-%d")
            if start_date and exp_date < start_date:
                continue
            if end_date and exp_date > end_date:
                continue
            total += expense["amount"]

        self.sum_label.config(text=f"Общая сумма за период: {total}")
#Шаг 6. Фильтрация данных
    def apply_filters(self):
        category_
