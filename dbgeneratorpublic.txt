import tkinter as tk
from tkinter import filedialog, messagebox, simpledialog, ttk
import csv
import random
import string
import re

class CSVEditor:
    def __init__(self, master):
        self.master = master
        self.master.title("CSV Editor")
        self.master.geometry("800x600")
        self.master.option_add('*TCombobox*Listbox.font', 'Helvetica 12')
        self.master.configure(bg='#000000')  # Black background for main window
        
        # Define some basic styles to mimic X platform's aesthetic
        style = ttk.Style()
        style.theme_use('alt')  # Use a theme that's more customizable
        style.configure('TButton', 
                        font=('Helvetica', 10), 
                        padding=6, 
                        background='#1DA1F2',  # X blue for buttons
                        foreground='#FFFFFF',  # White text
                        borderwidth=0)  # No border for flat look
        
        # Hover effect for buttons
        style.map('TButton', 
                  background=[('active', '#1976D2')],  # Darker blue on hover
                  foreground=[('active', '#FFFFFF')])
        
        # Style for Listbox
        style.configure('TListbox', 
                        background='#15202B',  # Dark gray background
                        foreground='#FFFFFF',  # White text
                        fieldbackground='#15202B')  # For entry or combobox fields
        
        # Style for Labels
        style.configure('TLabel', 
                        background='#000000',  # Black background
                        foreground='#FFFFFF')  # White text
        
        # Style for Entry
        style.configure('TEntry', 
                        background='#15202B',
                        foreground='#FFFFFF',
                        fieldbackground='#15202B')

        # UI elements
        ttk.Button(self.master, text="Open CSV", command=self.open_csv, style='TButton').pack(pady=5)
        ttk.Button(self.master, text="Save CSV", command=self.save_csv, style='TButton').pack(pady=5)

        ttk.Label(self.master, text="Columns:", style='TLabel').pack(pady=(10, 0))
        self.column_list = tk.Listbox(self.master, selectmode=tk.MULTIPLE, font=('Helvetica', 10), height=20)
        self.column_list.configure(bg='#15202B', fg='#FFFFFF', selectbackground='#1DA1F2', selectforeground='#FFFFFF')  # Dark theme colors
        self.column_list.pack(fill=tk.BOTH, expand=True, padx=10, pady=10)
        
        column_controls = ttk.Frame(self.master, style='Card.TFrame')
        column_controls.pack(pady=5, fill=tk.X)
        
        ttk.Label(column_controls, text="New Column Name:", style='TLabel').pack(side=tk.LEFT, padx=(0, 5))
        self.new_column_name = ttk.Entry(column_controls, font=('Helvetica', 10), style='TEntry')
        self.new_column_name.pack(side=tk.LEFT, padx=(0, 5), fill=tk.X, expand=True)
        ttk.Button(column_controls, text="Add", command=self.add_column, style='TButton').pack(side=tk.LEFT, padx=(0, 5))
        ttk.Button(column_controls, text="Remove", command=self.remove_column, style='TButton').pack(side=tk.LEFT)

        # Buttons for filling columns
        fill_options = ttk.Frame(self.master, style='Card.TFrame')
        fill_options.pack(pady=5, fill=tk.X)

        ttk.Button(fill_options, text="Custom Random Fill", command=self.custom_fill_random, style='TButton').pack(side=tk.LEFT, padx=5)
        ttk.Button(fill_options, text="Specific Fill", command=self.fill_specific, style='TButton').pack(side=tk.LEFT, padx=5)
        ttk.Button(fill_options, text="Email Fill", command=self.custom_email_fill, style='TButton').pack(side=tk.LEFT, padx=5)

        # Custom Style for Frames
        style.configure('Card.TFrame', 
                        background='#15202B',  # Dark gray for frames
                        relief='raised', 
                        borderwidth=1, 
                        padding=5)

        self.data = []

    def open_csv(self):
        file_path = filedialog.askopenfilename(filetypes=[("CSV Files", "*.csv")])
        if file_path:
            with open(file_path, newline='') as csvfile:
                reader = csv.reader(csvfile)
                self.data = list(reader)
            self.update_columns()
            messagebox.showinfo("Open File", "File opened successfully.")

    def update_columns(self):
        self.column_list.delete(0, tk.END)
        if self.data:
            for col in self.data[0]:
                self.column_list.insert(tk.END, col)

    def add_column(self):
        col_name = self.new_column_name.get()
        if col_name:
            if not self.data:
                self.data = [[""] * len(self.data[0]) if self.data else [""] for _ in range(101)]  # 100 rows + header

            self.data[0].append(col_name)
            
            # Fill the new column with random values
            col_index = len(self.data[0]) - 1
            for row in self.data[1:101]:  # Fill up to 100 rows
                if col_name.lower() in ["name", "lastname"]:
                    row.append(random.choice(["Alice", "Bob", "Charlie", "David", "Eve", "Frank"]))
                else:
                    row.append(f"Random{col_index}{random.randint(1,100)}")

            self.update_columns()
            self.new_column_name.delete(0, tk.END)

    def remove_column(self):
        selected_indices = sorted(self.column_list.curselection(), reverse=True)
        if selected_indices:
            for index in selected_indices:
                for row in self.data:
                    if index < len(row):
                        row.pop(index)
            self.update_columns()

    def custom_fill_random(self):
        if not self.data:
            messagebox.showwarning("Warning", "No CSV data to fill.")
            return
        columns = self.column_list.curselection()
        if not columns:
            messagebox.showwarning("Warning", "Please select columns to fill.")
            return

        dialog = tk.Toplevel(self.master)
        dialog.title("Fill Type")
        dialog.configure(bg='#000000')  # Dark background for dialog

        fill_type = tk.StringVar(value="Numeric")
        ttk.OptionMenu(dialog, fill_type, "Numeric", "Numeric", "Alphabetic", "Alphanumeric", style='TButton').pack(pady=10)

        def ok():
            self.fill_random(columns, fill_type.get())
            dialog.destroy()

        ttk.Button(dialog, text="OK", command=ok, style='TButton').pack()

    def fill_random(self, columns, fill_type):
        for col in columns:
            if col >= len(self.data[0]):
                for row in self.data:
                    row.append("")
            
            for row in self.data[1:]:  # Skipping header
                if fill_type == "Numeric":
                    row[col] = str(random.randint(1000000, 9999999))
                elif fill_type == "Alphabetic":
                    row[col] = ''.join(random.choices('ABCDEFGHIJKLMNOPQRSTUVWXYZ', k=random.randint(3, 10)))
                elif fill_type == "Alphanumeric":
                    row[col] = ''.join(random.choices('ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789', k=random.randint(3, 10)))

    def fill_specific(self):
        if not self.data:
            messagebox.showwarning("Warning", "No CSV data to fill.")
            return
        columns = self.column_list.curselection()
        if not columns:
            messagebox.showwarning("Warning", "Please select columns to fill.")
            return

        value = simpledialog.askstring("Input", "Enter the value to fill:")
        if value is not None:
            for col in columns:
                if col < len(self.data[0]):
                    for row in self.data[1:]:
                        if col < len(row):
                            row[col] = value

    def custom_email_fill(self):
        if not self.data:
            messagebox.showwarning("Warning", "No CSV data to fill.")
            return
        columns = self.column_list.curselection()
        if not columns:
            messagebox.showwarning("Warning", "Please select columns to fill.")
            return

        dialog = tk.Toplevel(self.master)
        dialog.title("Email Fill")
        dialog.geometry("300x250")  # Adjust size for extra field
        dialog.configure(bg='#000000')  # Dark background for dialog

        ttk.Label(dialog, text="Email User ID:", style='TLabel').pack()
        email_user_id = ttk.Entry(dialog, style='TEntry')
        email_user_id.pack(pady=(5,0))

        ttk.Label(dialog, text="Custom Domain:", style='TLabel').pack()
        email_domain = ttk.Entry(dialog, style='TEntry')
        email_domain.pack()

        fill_type = tk.StringVar(value="Alphanumeric")
        ttk.OptionMenu(dialog, fill_type, "Alphanumeric", "Alphanumeric", "Numeric", style='TButton').pack(pady=10)

        def ok():
            domain = email_domain.get().strip()
            if not re.match(r'^[a-zA-Z0-9][a-zA-Z0-9-]{1,61}[a-zA-Z0-9](?:\.[a-zA-Z]{2,})+$', domain):
                messagebox.showwarning("Warning", "Please enter a valid email domain.")
                return
            self.fill_email(columns, email_user_id.get(), fill_type.get(), domain)
            dialog.destroy()

        ttk.Button(dialog, text="OK", command=ok, style='TButton').pack()

    def fill_email(self, columns, user_id, fill_type, domain):
        for col in columns:
            if col < len(self.data[0]):
                for row in self.data[1:]:  # Skipping header
                    if len(row) <= col:
                        row.append("")
                    
                    random_value = ''.join(random.choices(
                        string.ascii_letters + string.digits if fill_type == "Alphanumeric" else string.digits,
                        k=random.randint(3, 10)
                    ))
                    row[col] = f"{user_id}+{random_value}@{domain}"

    def save_csv(self):
        if not self.data:
            messagebox.showwarning("No Data", "No data to save.")
            return
        file_path = filedialog.asksaveasfilename(defaultextension=".csv", filetypes=[("CSV Files", "*.csv")])
        if file_path:
            with open(file_path, 'w', newline='') as csvfile:
                writer = csv.writer(csvfile)
                writer.writerows(self.data)
            messagebox.showinfo("Save File", "File saved successfully.")

if __name__ == "__main__":
    try:
        root = tk.Tk()
        app = CSVEditor(root)
        root.mainloop()
    except Exception as e:
        print(f"An error occurred: {e}")