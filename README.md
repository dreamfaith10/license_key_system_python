import random
import string
import tkinter as tk
import pyperclip
import mysql.connector

# Connect to the MySQL database
db = mysql.connector.connect(
    host="127.0.0.1",
    user="root",
    password="Mashudu2@",
    database="user"
)
cursor = db.cursor()

def generate_license_key(key_length=16, segments=4, segment_length=4):
    characters = string.ascii_letters + string.digits  # Alphanumeric characters

    while True:
        license_key = '-'.join(''.join(random.choice(characters) for _ in range(segment_length)) for _ in range(segments))
        cursor.execute("SELECT key_value FROM keys WHERE key_value = %s", (license_key,))
        if cursor.fetchone() is None:
            return license_key

def save_license_key_to_database(key):
    cursor.execute("INSERT INTO keys (key_value, is_activated) VALUES (%s, %s)", (key, 0))
    db.commit()

def activate_license(key):
    cursor.execute("UPDATE keys SET is_activated = 1 WHERE key_value = %s", (key,))
    db.commit()

def copy_key_to_clipboard():
    license_key = key_label.cget("text").split(": ")[1]
    pyperclip.copy(license_key)

# Create the main window
root = tk.Tk()
root.title("License Key Generator")

# Create a button to generate the license key
generate_button = tk.Button(root, text="Generate License Key", command=generate_key_and_display)
generate_button.pack(pady=10)

# Create a label to display the generated key
key_label = tk.Label(root, text="", font=("Helvetica", 12))
key_label.pack()

# Create an entry for the user to enter the license key for activation
key_entry = tk.Entry(root)
key_entry.pack(pady=5)

# Create a button to activate the entered license key
activate_button = tk.Button(root, text="Activate License Key", command=activate_license)
activate_button.pack()

# Create a button to copy the license key to the clipboard (initially disabled)
copy_button = tk.Button(root, text="Copy to Clipboard", command=copy_key_to_clipboard, state="disabled")
copy_button.pack()

# Run the Tkinter event loop
root.mainloop()

# Close the database connection when the application closes
db.close()
