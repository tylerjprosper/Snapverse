#
# ███████╗███╗   ██╗ █████╗ ██████╗ ██╗   ██╗███████╗██████╗ ███████╗███████╗
# ██╔════╝████╗  ██║██╔══██╗██╔══██╗██║   ██║██╔════╝██╔══██╗██╔════╝██╔════╝
# ███████╗██╔██╗ ██║███████║██████╔╝██║   ██║█████╗  ██████╔╝███████╗█████╗
# ╚════██║██║╚██╗██║██╔══██║██╔═══╝ ╚██╗ ██╔╝██╔══╝  ██╔══██╗╚════██║██╔══╝  
# ███████║██║ ╚████║██║  ██║██║      ╚████╔╝ ███████╗██║  ██║███████║███████╗
# ╚══════╝╚═╝  ╚═══╝╚═╝  ╚═╝╚═╝       ╚═══╝  ╚══════╝╚═╝  ╚═╝╚══════╝╚══════╝
#                 

import tkinter as tk
import win32clipboard
from tkinter import ttk, colorchooser
from PIL import ImageGrab, Image, ImageTk
import requests
import pyperclip
from tkcolorpicker import askcolor
import keyboard
import io
from io import BytesIO
import webbrowser

GYAZO_ACCESS_TOKEN = "TOKEN"

def send_to_clipboard(img: Image.Image):
    output = BytesIO()
    img.convert('RGB').save(output, 'BMP')
    data = output.getvalue()[14:] 

    win32clipboard.OpenClipboard()
    win32clipboard.EmptyClipboard()
    win32clipboard.SetClipboardData(win32clipboard.CF_DIB, data)
    win32clipboard.CloseClipboard()

def upload_to_gyazo(image):
    buffer = io.BytesIO()
    image.save(buffer, format='PNG')
    buffer.seek(0)

    files = {
        'imagedata': ('screenshot.png', buffer, 'image/png')
    }
    headers = {
        'Authorization': f'Bearer {GYAZO_ACCESS_TOKEN}'
    }

    response = requests.post('https://upload.gyazo.com/api/upload', headers=headers, files=files)

    if response.ok:
        url = response.json().get('url')
        if url:
            pyperclip.copy(url)
            print("✅ Uploaded to Gyazo:", url)
            webbrowser.open(url)
        else:
            print("⚠️ Uploaded but no URL returned")
    else:
        print("❌ Upload failed:", response.status_code, response.text)

class RegionSnipper:
    def __init__(self):
        self.root = tk.Tk()
        self.root.attributes("-fullscreen", True)
        self.root.attributes("-alpha", 0.3)
        self.root.configure(bg='black')
        self.root.title("Select Region")

        self.canvas = tk.Canvas(self.root, cursor="cross", bg='gray', highlightthickness=0)
        self.canvas.pack(fill=tk.BOTH, expand=True)

        self.start_x = self.start_y = 0
        self.rect = None

        self.default_font = ("Segoe UI", 10)
        self.root.option_add("*Font", self.default_font)

        self.root.bind("<Button-1>", self.on_click)
        self.root.bind("<B1-Motion>", self.on_drag)
        self.root.bind("<ButtonRelease-1>", self.on_release)

    def on_click(self, event):
        self.start_x = self.canvas.canvasx(event.x)
        self.start_y = self.canvas.canvasy(event.y)
        if self.rect:
            self.canvas.delete(self.rect)
        self.rect = self.canvas.create_rectangle(self.start_x, self.start_y, self.start_x, self.start_y,
                                                 outline="#ff0000", width=2)

    def on_drag(self, event):
        cur_x = self.canvas.canvasx(event.x)
        cur_y = self.canvas.canvasy(event.y)
        self.canvas.coords(self.rect, self.start_x, self.start_y, cur_x, cur_y)

    def on_release(self, event):
        end_x = self.canvas.canvasx(event.x)
        end_y = self.canvas.canvasy(event.y)
        self.root.destroy()

        x1 = int(min(self.start_x, end_x))
        y1 = int(min(self.start_y, end_y))
        x2 = int(max(self.start_x, end_x))
        y2 = int(max(self.start_y, end_y))

        image = ImageGrab.grab(bbox=(x1, y1, x2, y2))
        send_to_clipboard(image) 
        open_draw_editor(image)

def open_draw_editor(image):
    strokes = []  
    current_stroke = []  
    window = tk.Tk()
    window.title("Snapverse - Edit Screenshot")
    window.configure(bg="#1e1e1e")
    window.resizable(False, False)

    screen_width = window.winfo_screenwidth()
    screen_height = window.winfo_screenheight()
    win_width, win_height = image.size
    x = int((screen_width - win_width) / 2)
    y = int((screen_height - win_height) / 2)
    window.geometry(f"{win_width}x{win_height + 70}+{x}+{y}")

    default_font = ("Segoe UI", 10)
    window.option_add("*Font", default_font)

    style = ttk.Style(window)
    style.theme_use("clam")
    style.configure("Accent.TButton",
                    foreground="black",
                    background="#0078d7",
                    padding=8,
                    font=("Segoe UI", 10, "bold"),
                    borderwidth=0)
    style.map("Accent.TButton",
              foreground=[('active', 'black')],
              background=[('active', '#005a9e')])
    style.configure("TLabel", background="#1e1e1e", foreground="white")

    width, height = image.size

    container = ttk.Frame(window, padding=(12, 12, 12, 24), style="TFrame")  # Add bottom padding here
    container.pack(fill="both", expand=True)

    # === Screenshot canvas ===
    canvas = tk.Canvas(container, width=width, height=height, bg="#000000", highlightthickness=0)
    canvas.grid(row=0, column=0, sticky="nsew")

    tk_img = ImageTk.PhotoImage(image)
    canvas.create_image(0, 0, anchor='nw', image=tk_img)

    # === Drawing logic ===
    last_x = last_y = None
    current_color = "#ff5555"
    lines_drawn = []

    def draw(event):
        nonlocal last_x, last_y, current_stroke
        if last_x is not None and last_y is not None:
            line_id = canvas.create_line(last_x, last_y, event.x, event.y, fill=current_color,
                                        width=3, capstyle="round", smooth=True)
            current_stroke.append(line_id)
        last_x = event.x
        last_y = event.y

    def reset(event):
        nonlocal last_x, last_y, current_stroke
        if current_stroke:
            strokes.append(current_stroke)
            current_stroke = []
        last_x = last_y = None

    def undo(event=None):
        if strokes:
            last_stroke = strokes.pop()
            for line_id in last_stroke:
                canvas.delete(line_id)

    def choose_color():
        nonlocal current_color
        color = askcolor(title="Choose Drawing Color", color=current_color)
        if color and color[1]:
            current_color = color[1]
            color_button.config(background=current_color)

    def upload_and_close():
        x = canvas.winfo_rootx()
        y = canvas.winfo_rooty()
        w = x + canvas.winfo_width()
        h = y + canvas.winfo_height()
        edited_image = ImageGrab.grab(bbox=(x, y, w, h))
        send_to_clipboard(edited_image)
        window.destroy()
        upload_to_gyazo(edited_image)

    def copy_canvas_to_clipboard(event=None):
        x = canvas.winfo_rootx()
        y = canvas.winfo_rooty()
        w = x + canvas.winfo_width()
        h = y + canvas.winfo_height()
        edited_image = ImageGrab.grab(bbox=(x, y, w, h))
        send_to_clipboard(edited_image)
        print("Copied to clipboard")

    canvas.bind("<B1-Motion>", draw)
    canvas.bind("<ButtonRelease-1>", reset)
    canvas.bind_all("<Control-z>", undo)
    canvas.focus_set() 
    window.bind_all("<Control-c>", copy_canvas_to_clipboard)

    # === Buttons area in a separate frame ===
    button_frame = ttk.Frame(container, style="TFrame")
    button_frame.grid(row=1, column=0, pady=(16, 0), sticky="ew")

    upload_button = ttk.Button(button_frame, text="Upload to Gyazo", command=upload_and_close, style="Accent.TButton")
    upload_button.grid(row=0, column=0, sticky="ew", padx=(0, 5))

    cancel_button = ttk.Button(button_frame, text="Cancel", command=window.destroy, style="Accent.TButton")
    cancel_button.grid(row=0, column=1, sticky="ew", padx=(5, 5))

    color_button = tk.Button(button_frame, text="Color", background=current_color, fg="black",
                             command=choose_color, relief="flat", width=10, height=1)
    color_button.grid(row=0, column=2, sticky="ew", padx=(5, 5))

    help_label = ttk.Label(button_frame, text="Ctrl+Z to Undo | Ctrl+C to Copy", style="TLabel")
    help_label.grid(row=0, column=3, sticky="e", padx=(5, 0))

    button_frame.columnconfigure(0, weight=1)
    button_frame.columnconfigure(1, weight=1)
    button_frame.columnconfigure(2, weight=0)
    button_frame.columnconfigure(3, weight=1)

    container.rowconfigure(0, weight=1)
    container.columnconfigure(0, weight=1)

    window.mainloop()

def start_snipping():
    snipper = RegionSnipper()
    snipper.root.mainloop()

if __name__ == "__main__":
    print("📸 Press Ctrl+Shift+X to snip, draw, and upload to Gyazo...")
    keyboard.add_hotkey('ctrl+shift+x', start_snipping)
    keyboard.wait()
