# Training Planner

**Автор:** Галиев Наиль Дамирович

## Описание

Training Planner — это графическое приложение на Python для планирования тренировок. Позволяет добавлять, удалять и фильтровать тренировки по дате и типу. Данные сохраняются в JSON-файл.

## Функциональность

- Добавление тренировок (дата, тип, длительность)
- Валидация ввода (формат даты, положительная длительность)
- Фильтрация по типу тренировки и дате
- Удаление тренировок
- Сохранение/загрузка в JSON
- Простой графический интерфейс (tkinter)

## Установка и запуск

1. Убедитесь, что установлен Python 3.6+
2. Скачайте файл `training_planner.py`
3. Запустите приложение:
   ```bash
   python training_planner.py
import json
import os
from datetime import datetime
import tkinter as tk
from tkinter import ttk, messagebox

DATA_FILE = "training_data.json"


class TrainingPlanner:
    def __init__(self, root):
        self.root = root
        self.root.title("Training Planner")
        self.root.geometry("700x500")

        # Данные тренировок
        self.trainings = []
        self.filtered_trainings = []

        # Загрузка данных
        self.load_data()

        # Создание интерфейса
        self.create_widgets()
        self.update_table()

    def create_widgets(self):
        # Рамка для ввода
        input_frame = ttk.LabelFrame(self.root, text="Добавить тренировку", padding=10)
        input_frame.pack(fill="x", padx=10, pady=5)

        # Поле Дата
        ttk.Label(input_frame, text="Дата (ГГГГ-ММ-ДД):").grid(row=0, column=0, sticky="w", padx=5, pady=5)
        self.date_entry = ttk.Entry(input_frame, width=15)
        self.date_entry.grid(row=0, column=1, padx=5, pady=5)
        self.date_entry.insert(0, datetime.now().strftime("%Y-%m-%d"))

        # Поле Тип тренировки
        ttk.Label(input_frame, text="Тип тренировки:").grid(row=0, column=2, sticky="w", padx=5, pady=5)
        self.type_entry = ttk.Entry(input_frame, width=15)
        self.type_entry.grid(row=0, column=3, padx=5, pady=5)

        # Поле Длительность
        ttk.Label(input_frame, text="Длительность (мин):").grid(row=0, column=4, sticky="w", padx=5, pady=5)
        self.duration_entry = ttk.Entry(input_frame, width=10)
        self.duration_entry.grid(row=0, column=5, padx=5, pady=5)

        # Кнопка Добавить
        add_btn = ttk.Button(input_frame, text="Добавить тренировку", command=self.add_training)
        add_btn.grid(row=0, column=6, padx=10, pady=5)

        # Рамка для фильтров
        filter_frame = ttk.LabelFrame(self.root, text="Фильтрация", padding=10)
        filter_frame.pack(fill="x", padx=10, pady=5)

        ttk.Label(filter_frame, text="Фильтр по типу:").grid(row=0, column=0, padx=5, pady=5)
        self.filter_type = ttk.Entry(filter_frame, width=15)
        self.filter_type.grid(row=0, column=1, padx=5, pady=5)

        ttk.Label(filter_frame, text="Фильтр по дате (ГГГГ-ММ-ДД):").grid(row=0, column=2, padx=5, pady=5)
        self.filter_date = ttk.Entry(filter_frame, width=15)
        self.filter_date.grid(row=0, column=3, padx=5, pady=5)

        filter_btn = ttk.Button(filter_frame, text="Применить фильтр", command=self.apply_filter)
        filter_btn.grid(row=0, column=4, padx=10, pady=5)

        reset_btn = ttk.Button(filter_frame, text="Сбросить фильтр", command=self.reset_filter)
        reset_btn.grid(row=0, column=5, padx=5, pady=5)

        # Таблица для отображения
        table_frame = ttk.Frame(self.root)
        table_frame.pack(fill="both", expand=True, padx=10, pady=5)

        columns = ("date", "type", "duration")
        self.tree = ttk.Treeview(table_frame, columns=columns, show="headings")

        self.tree.heading("date", text="Дата")
        self.tree.heading("type", text="Тип тренировки")
        self.tree.heading("duration", text="Длительность (мин)")

        self.tree.column("date", width=120)
        self.tree.column("type", width=200)
        self.tree.column("duration", width=120)

        scrollbar = ttk.Scrollbar(table_frame, orient="vertical", command=self.tree.yview)
        self.tree.configure(yscrollcommand=scrollbar.set)

        self.tree.pack(side="left", fill="both", expand=True)
        scrollbar.pack(side="right", fill="y")

        # Кнопка удаления
        delete_btn = ttk.Button(self.root, text="Удалить выбранную тренировку", command=self.delete_training)
        delete_btn.pack(pady=10)

    def add_training(self):
        date = self.date_entry.get().strip()
        training_type = self.type_entry.get().strip()
        duration = self.duration_entry.get().strip()

        # Проверка корректности ввода
        if not date or not training_type or not duration:
            messagebox.showerror("Ошибка", "Заполните все поля!")
            return

        # Проверка формата даты
        try:
            datetime.strptime(date, "%Y-%m-%d")
        except ValueError:
            messagebox.showerror("Ошибка", "Неверный формат даты! Используйте ГГГГ-ММ-ДД")
            return

        # Проверка длительности (положительное число)
        try:
            duration_min = float(duration)
            if duration_min <= 0:
                raise ValueError
        except ValueError:
            messagebox.showerror("Ошибка", "Длительность должна быть положительным числом!")
            return

        # Добавление записи
        training = {"date": date, "type": training_type, "duration": duration_min}
        self.trainings.append(training)
        self.save_data()
        self.reset_filter()
        self.update_table()

        # Очистка полей (кроме даты, оставляем сегодняшнюю)
        self.type_entry.delete(0, tk.END)
        self.duration_entry.delete(0, tk.END)
        self.date_entry.delete(0, tk.END)
        self.date_entry.insert(0, datetime.now().strftime("%Y-%m-%d"))

        messagebox.showinfo("Успех", "Тренировка добавлена!")

    def delete_training(self):
        selected = self.tree.selection()
        if not selected:
            messagebox.showwarning("Предупреждение", "Выберите тренировку для удаления!")
            return

        # Получаем индекс выбранной записи в текущем отфильтрованном списке
        item = self.tree.item(selected[0])
        selected_values = item["values"]

        # Находим и удаляем запись из исходного списка
        for i, training in enumerate(self.trainings):
            if (training["date"] == selected_values[0] and
                training["type"] == selected_values[1] and
                training["duration"] == float(selected_values[2])):
                del self.trainings[i]
                break

        self.save_data()
        self.apply_filter()  # Обновляем с текущим фильтром
        messagebox.showinfo("Успех", "Тренировка удалена!")

    def apply_filter(self):
        filter_type = self.filter_type.get().strip().lower()
        filter_date = self.filter_date.get().strip()

        self.filtered_trainings = self.trainings.copy()

        if filter_type:
            self.filtered_trainings = [
                t for t in self.filtered_trainings
                if filter_type in t["type"].lower()
            ]

        if filter_date:
            try:
                datetime.strptime(filter_date, "%Y-%m-%d")
                self.filtered_trainings = [
                    t for t in self.filtered_trainings
                    if t["date"] == filter_date
                ]
            except ValueError:
                messagebox.showerror("Ошибка", "Неверный формат даты фильтра!")
                return

        self.update_table()

    def reset_filter(self):
        self.filter_type.delete(0, tk.END)
        self.filter_date.delete(0, tk.END)
        self.filtered_trainings = self.trainings.copy()
        self.update_table()

    def update_table(self):
        # Очищаем таблицу
        for item in self.tree.get_children():
            self.tree.delete(item)

        # Заполняем данными
        for training in self.filtered_trainings:
            self.tree.insert("", "end", values=(
                training["date"],
                training["type"],
                training["duration"]
            ))

    def load_data(self):
        if os.path.exists(DATA_FILE):
            try:
                with open(DATA_FILE, "r", encoding="utf-8") as f:
                    self.trainings = json.load(f)
                self.filtered_trainings = self.trainings.copy()
            except json.JSONDecodeError:
                self.trainings = []
                self.filtered_trainings = []

    def save_data(self):
        with open(DATA_FILE, "w", encoding="utf-8") as f:
            json.dump(self.trainings, f, ensure_ascii=False, indent=4)


if __name__ == "__main__":
    root = tk.Tk()
    app = TrainingPlanner(root)
    root.mainloop()
  
