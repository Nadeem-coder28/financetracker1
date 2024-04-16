import tkinter as tk
from tkinter import ttk, messagebox
from PIL import Image, ImageTk
from datetime import datetime

class Transaction:
    def __init__(self, description, amount):
        self.description = description
        self.amount = amount
        self.date = datetime.now().strftime("%Y-%m-%d %H:%M:%S")

class FinanceTrackerGUI:
    def __init__(self, root):
        self.root = root
        self.root.title("Finance Tracker")

        # Load the background image
        background_image = Image.open(r"C:\Users\nadee\Downloads\finance.jpg")
        background_photo = ImageTk.PhotoImage(background_image)
        
        # Create a canvas and add the background image
        canvas = tk.Canvas(root, width=800, height=600)
        canvas.pack(fill="both", expand=True)
        canvas.create_image(0, 0, image=background_photo, anchor="nw")

        self.transactions = []
        self.transaction_count = 0
        self.balance = 0
        self.savings_goal = 0

        self.create_widgets()

        # Bind the window resize event
        self.root.bind("<Configure>", self.center_window)

    def create_widgets(self):
        style = ttk.Style()
        style.configure('TButton', padding=6, relief='flat', background='#4CAF50', foreground='black')
        style.map('TButton', background=[('active', '#45a049')])

        frame = ttk.Frame(self.root, padding="20")
        frame.place(relx=0.5, rely=0.5, anchor="center")

        # Labels
        ttk.Label(frame, text="Description:", font=('Helvetica', 12), foreground='black').grid(row=0, column=0, padx=5, pady=5)
        ttk.Label(frame, text="Amount: $", font=('Helvetica', 12), foreground='black').grid(row=1, column=0, padx=5, pady=5)

        # Entry fields
        self.description_entry = ttk.Entry(frame, font=('Helvetica', 12))
        self.description_entry.grid(row=0, column=1, padx=5, pady=5)
        self.amount_entry = ttk.Entry(frame, font=('Helvetica', 12))
        self.amount_entry.grid(row=1, column=1, padx=5, pady=5)

        # Buttons
        ttk.Button(frame, text="Add Income", command=self.add_income).grid(row=2, column=0, columnspan=2, sticky="we", padx=5, pady=5)
        ttk.Button(frame, text="Add Expense", command=self.add_expense).grid(row=3, column=0, columnspan=2, sticky="we", padx=5, pady=5)
        ttk.Button(frame, text="Show Transactions", command=self.show_transactions).grid(row=4, column=0, columnspan=2, sticky="we", padx=5, pady=5)
        
        # Search Bar
        self.search_entry = ttk.Entry(frame, font=('Helvetica', 12))
        self.search_entry.grid(row=5, column=1, padx=5, pady=5)
        ttk.Button(frame, text="Search", command=self.search_transactions).grid(row=5, column=0, padx=5, pady=5)

        ttk.Button(frame, text="Show Balance", command=self.show_balance).grid(row=6, column=0, columnspan=2, sticky="we", padx=5, pady=5)
        ttk.Button(frame, text="Set Savings Goal", command=self.set_savings_goal).grid(row=7, column=0, columnspan=2, sticky="we", padx=5, pady=5)

        # Exit Button
        ttk.Button(frame, text="Exit", command=self.exit_program).grid(row=8, column=0, columnspan=2, sticky="we", padx=5, pady=5)

    def center_window(self, event):
        screen_width = self.root.winfo_screenwidth()
        screen_height = self.root.winfo_screenheight()
        window_width = self.root.winfo_width()
        window_height = self.root.winfo_height()

        x = (screen_width - window_width) / 2
        y = (screen_height - window_height) / 2

        self.root.geometry("+%d+%d" % (x, y))

    def add_income(self):
        description = self.description_entry.get()
        amount = self.amount_entry.get()
        
        if not description or not amount:
            messagebox.showwarning("Warning", "Please enter both description and amount.")
            return

        if not amount.isdigit():
            messagebox.showwarning("Warning", "Amount must be an integer.")
            return

        amount = int(amount)
        self.transactions.append(Transaction(description, amount))
        self.transaction_count += 1
        self.balance += amount
        self.show_popup(f"Income of ${amount} added successfully.")

    def add_expense(self):
        description = self.description_entry.get()
        amount = self.amount_entry.get()
        
        if not description or not amount:
            messagebox.showwarning("Warning", "Please enter both description and amount.")
            return

        if not amount.isdigit():
            messagebox.showwarning("Warning", "Amount must be an integer.")
            return

        amount = -int(amount)
        potential_new_balance = self.balance + amount

        if potential_new_balance < self.savings_goal:
            response = messagebox.askyesno("Warning", f"This expense will cause your balance to fall below your savings goal of ${self.savings_goal}. Are you sure you want to proceed?")
            if not response:
                return

        self.transactions.append(Transaction(description, amount))
        self.transaction_count += 1
        self.balance += amount
        self.show_popup(f"Expense of ${abs(amount)} added successfully.")

    def show_transactions(self):
        if not self.transactions:
            self.show_popup("No transactions to display.")
            return

        transaction_text = "Transaction History:\n"
        for i, transaction in enumerate(self.transactions, 1):
            transaction_text += f"{i}. {transaction.description}: {'+' if transaction.amount >= 0 else '-'}${abs(transaction.amount)} ({transaction.date})\n"
        self.show_popup(transaction_text, title="Transactions")

    def search_transactions(self):
        keyword = self.search_entry.get().lower()
        found_transactions = [transaction for transaction in self.transactions if keyword in transaction.description.lower()]
        if not found_transactions:
            self.show_popup(f"No transactions found with keyword '{keyword}'.")
            return

        transaction_text = "Search Results:\n"
        for i, transaction in enumerate(found_transactions, 1):
            transaction_text += f"{i}. {transaction.description}: {'+' if transaction.amount >= 0 else '-'}${abs(transaction.amount)} ({transaction.date})\n"
        self.show_popup(transaction_text, title="Search Results")

    def show_balance(self):
        self.show_popup(f"Your current balance is: ${self.balance}", title="Balance")

    def set_savings_goal(self):
        try:
            self.savings_goal = int(self.amount_entry.get())
            self.show_popup(f"Savings goal set to ${self.savings_goal}.")
        except ValueError:
            self.show_popup("Invalid input for savings goal. Please enter an integer value.")

    def exit_program(self):
        self.root.destroy()

    def show_popup(self, message, title="Message"):
        popup = tk.Toplevel(self.root)
        popup.title(title)
        popup.geometry("400x300")
        popup_label = ttk.Label(popup, text=message, font=('Helvetica', 12), foreground='black', wraplength=380)
        popup_label.pack(padx=10, pady=10)

if __name__ == "__main__":
    root = tk.Tk()
    app = FinanceTrackerGUI(root)
    root.mainloop()
