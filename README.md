# StaffTrack: Employee Management System

StaffTrack is a desktop application that simplifies employee management and tracks administrative details like department assignment, salaries, and performance reviews. The application consists of two main modules:

1. **Login & Registration System** (`stafftrack_login.py`): Handles user authentication.
2. **Employee Management System** (`staff_track.py`): Manages employee records, including adding, updating, and searching for employee details.

## Features

### Login & Registration Module
- Secure user login and registration.
- Validates input for mandatory fields.
- Integrates with a MySQL database for user data storage.

### Employee Management Module
- Add, update, delete, and search employee records.
- Assign employees to departments.
- Track salaries and performance reviews.
- Calculate the total salary of all employees.
- Store employee and department information in a SQLite database.

## Prerequisites

### Software Requirements
- **Python 3.8 or later**
- **MySQL Server** (for authentication)
- **SQLite** (for employee and department data storage)

### Python Libraries
Install the required Python libraries using:

```bash
pip install mysql-connector-python pillow customtkinter
```

## Setup

### Database Configuration

#### MySQL (Login & Registration)
1. Create a MySQL database:
   ```sql
   CREATE DATABASE staff_track_db;
   ```
2. Create the `admin` table:
   ```sql
   USE staff_track_db;
   CREATE TABLE admin (
       id INT AUTO_INCREMENT PRIMARY KEY,
       username VARCHAR(255) NOT NULL,
       email VARCHAR(255) UNIQUE NOT NULL,
       password VARCHAR(255) NOT NULL
   );
   ```
3. Update the MySQL credentials in `stafftrack_login.py`:
   ```python
   self.db = mysql.connector.connect(
       host="localhost",
       user="StaffTracker",
       password="your_password",
       database="staff_track_db"
   )
   ```

#### SQLite (Employee Management)
The application automatically initializes the SQLite database (`employee_management.db`) when `staff_track.py` is run for the first time.

### Running the Application

#### Step 1: Launch the Login System
Run the following command to start the login and registration module:
```bash
python stafftrack_login.py
```

#### Step 2: Access the Employee Management System
Upon successful login, the employee management module (`staff_track.py`) is launched automatically.

## File Structure

- **`stafftrack_login.py`**: Manages user login and registration.
- **`staff_track.py`**: Handles employee data management.
- **`employee_management.db`**: SQLite database for employee and department data (auto-created).
- **MySQL Database**: Stores login and registration data.

## Code Snippets

### `stafftrack_login.py`
```python
import tkinter as tk
import customtkinter as ctk
import mysql.connector
from tkinter import messagebox
import subprocess
from PIL import Image, ImageTk

# StaffTrack Register and Login of Human Resource Manager
# Login and Register Setup
class LoginRegisterApp:
    def __init__(self, root):
        self.root = root
        self.root.geometry("900x600")
        self.root.title("Login & Register")
        self.root.resizable(False, False)
        image = Image.open('/Users/elizamagluyan/Desktop/FinalProj_ACP/download.png')
        photo = ImageTk.PhotoImage(image)
        self.root.iconphoto(False, photo)

        # Establish MySQL connection
        try:
            self.db = mysql.connector.connect(
                host="localhost",
                user="StaffTracker",
                password="0801",
                database="staff_track_db"
            )
            self.cursor = self.db.cursor()
            print("Database connection established!")
        except mysql.connector.Error as err:
            messagebox.showerror("Database Connection Error", f"Error: {err}")
            exit()

        self.show_login_frame()

    #Login & SignUP
    def show_login_frame(self):
        for widget in self.root.winfo_children():
            widget.destroy()

        self.root.geometry("900x600")
        self.root.resizable(False, False)

        main_container = tk.Frame(self.root)
        main_container.pack(fill="both", expand=True)

        left_frame = tk.Frame(main_container, bg="#FFBC70")
        left_frame.pack(side="left", fill="both", expand=True)

        right_frame = tk.Frame(main_container, bg="#5A5A5A")
        right_frame.pack(side="right", fill="both", expand=True)

        title_container = tk.Frame(left_frame, bg="#FFBC70")
        title_container.place(relx=0.5, rely=0.5, anchor="center")

        system_name = tk.Label(title_container, text="Staff Track", font=("Poppins", 60, "bold"), bg="#5A5A5A", fg="#FFBC70")
        system_name.pack()

        system_label = tk.Label(title_container, text="Your Employee Tracker", font=("Verdana", 15, "bold"), bg="#FFBC70", fg="#5A5A5A", justify="center")
        system_label.pack(pady=(10, 0))

        container = tk.Frame(right_frame, bg="#5A5A5A")
        container.place(relx=0.5, rely=0.5, anchor="center")

        tk.Label(container, text="Email", bg="#5A5A5A", fg="white", font=("Poppins", 17, "bold"), padx=5).pack(pady=(10, 10), anchor="w")
        email = ctk.CTkEntry(container, width=350, height=50, fg_color="white", text_color="black", corner_radius=15)
        email.pack(pady=(0, 10), anchor="w")

        tk.Label(container, text="Password", fg="white", bg="#5A5A5A" ,font=("Poppins", 17, "bold"), padx=5).pack(pady=(10, 10), anchor="w")
        password = ctk.CTkEntry(container, width=350, height=50, fg_color="white", text_color="black", show="*", corner_radius=15)
        password.pack(pady=(0, 10), anchor="w")

        login_admin_button = ctk.CTkButton(container, text="Login Now", fg_color="#FFBC70", text_color="white", font=("Poppins", 15, "bold"), width=200, height=50, corner_radius=15, hover_color="#000080", command=lambda: self.login(email.get().strip(), password.get()))
        login_admin_button.pack(pady=20)

        hyperlink_frame = tk.Frame(container, bg="#5A5A5A")
        hyperlink_frame.pack(pady=(0, 10))

        sign_up = tk.Label(hyperlink_frame, text="Don't have an Account yet? ", bg="#5A5A5A", fg="white", font=("Verdana", 12))
        sign_up.pack(side=tk.LEFT)

        back_to_menu_link = tk.Label(hyperlink_frame, text="Sign Up!", bg="#5A5A5A", fg="#FFBC70", font=("Verdana", 12, "bold"), cursor="hand2")
        back_to_menu_link.pack(side=tk.LEFT)

        back_to_menu_link.bind("<Button-1>", lambda e: self.show_register_frame())
```

### `staff_track.py`
```python
import tkinter as tk
from tkinter import ttk, messagebox
from tkinter import ttk
import sqlite3
from PIL import Image, ImageTk

# Database setup
def create_database():
    conn = sqlite3.connect("employee_management.db")
    cursor = conn.cursor()

    # Create departments table
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS departments (
            dept_id INTEGER PRIMARY KEY AUTOINCREMENT,
            department_name TEXT UNIQUE NOT NULL
        )
    ''')

    # Create employees table with date_added
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS employees (
            emp_id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            role TEXT,
            salary REAL,
            performance_review TEXT,
            date_added TEXT DEFAULT (DATE('now')),
            dept_id INTEGER,
            FOREIGN KEY(dept_id) REFERENCES departments(dept_id)
        )
    ''')
    conn.commit()
    conn.close()

# Add employee
def add_employee():
    name = name_entry.get().strip()
    role = role_entry.get().strip()
    salary = salary_entry.get().strip()
    dept_name = dept_entry_emp.get().strip()
    review = review_text.get("1.0", tk.END).strip()

    if not name or not role or not salary or not dept_name:
        messagebox.showwarning("Input Error", "All fields except review are mandatory.")
        return

    try:
        salary = float(salary)
    except ValueError:
        messagebox.showerror("Input Error", "Salary must be a valid number.")
        return

    conn = sqlite3.connect("employee_management.db")
    cursor = conn.cursor()

    # Check if the department exists, create if it does not
    cursor.execute("SELECT dept_id FROM departments WHERE department_name = ?", (dept_name,))
    dept_id = cursor.fetchone()

    if not dept_id:
        # Prompt the user to confirm creating the department
        if messagebox.askyesno("Department Not Found", f"Department '{dept_name}' does not exist. Create it?"):
            cursor.execute("INSERT INTO departments (department_name) VALUES (?)", (dept_name,))
            dept_id = cursor.lastrowid
        else:
            conn.close()
            return
    else:
        dept_id = dept_id[0]

    cursor.execute('''
        INSERT INTO employees (name, role, salary, performance_review, dept_id)
        VALUES (?, ?, ?, ?, ?)
    ''', (name, role, salary, review, dept_id))
    conn.commit()
    conn.close()
    messagebox.showinfo("Success", "Employee added successfully.")
    display_employees()
    clear_fields()
```

## Future Enhancements

- **Password Hashing**: Enhance security by storing hashed passwords.
- **Role-Based Access Control**: Restrict access to certain features based on user roles.
- **Reports Generation**: Include detailed employee and salary reports.
- **Responsive Design**: Improve the GUI for varying screen resolutions.

## License

This project is licensed under the MIT License. See the LICENSE file for details.

---

Feel free to contribute or report any issues to help improve StaffTrack!
