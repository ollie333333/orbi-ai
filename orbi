cat > orbi.py << 'EOF'
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

import sys, os, subprocess, json
from PySide6.QtWidgets import (
    QApplication, QWidget, QVBoxLayout, QTextEdit, QLineEdit,
    QPushButton, QHBoxLayout, QLabel, QListWidget, QColorDialog
)
from PySide6.QtGui import QTextCursor, QColor, QTextCharFormat, QFont
from PySide6.QtCore import QTimer, Qt

DATA_FILE = "orbi_chats.json"
OLLAMA_MODEL = "llama2:latest"
PULSE_INTERVAL = 120
MAX_CONTEXT = 10  # how many past messages to remember

# ==========================
# AI WITH MEMORY
# ==========================
def get_ai_response(prompt, history):
    try:
        # Build context from previous messages
        context = ""
        for msg in history[-MAX_CONTEXT:]:
            context += msg + "\n"

        full_prompt = context + "User: " + prompt + "\nOrbi:"

        result = subprocess.run(
            ["ollama", "run", OLLAMA_MODEL, full_prompt],
            capture_output=True, text=True, timeout=25
        )

        out = result.stdout.strip()
        return out if out else "[No response]"
    except Exception as e:
        return f"[Error: {e}]"

# ==========================
# Storage
# ==========================
def load_data():
    if not os.path.exists(DATA_FILE):
        return []
    with open(DATA_FILE, "r") as f:
        return json.load(f)

def save_data(data):
    with open(DATA_FILE, "w") as f:
        json.dump(data, f, indent=2)

def summarize(text):
    return text[:30] + ("..." if len(text) > 30 else "")

# ==========================
# App
# ==========================
class Orbi(QWidget):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("Orbi AI")
        self.resize(800, 500)

        self.chats = load_data()
        self.current_chat = None

        self.text_color = QColor("white")
        self.rainbow = False
        self.rainbow_index = 0
        self.colors = ["red","orange","yellow","green","blue","indigo","violet"]

        self.init_ui()
        self.refresh_chat_list()
        self.start_pulse()

        if not self.chats:
            self.new_chat()

        self.input.setFocus()

    def init_ui(self):
        main = QHBoxLayout(self)

        # Sidebar
        side = QVBoxLayout()
        self.chat_list = QListWidget()
        self.chat_list.clicked.connect(self.load_chat)
        side.addWidget(self.chat_list)

        new_btn = QPushButton("+ New Chat")
        new_btn.clicked.connect(self.new_chat)
        side.addWidget(new_btn)

        main.addLayout(side, 1)

        # Chat area
        layout = QVBoxLayout()

        self.display = QTextEdit()
        self.display.setReadOnly(True)
        self.display.setFont(QFont("Consolas", 12))
        layout.addWidget(self.display)

        # Input
        input_row = QHBoxLayout()
        self.input = QLineEdit()
        self.input.setPlaceholderText("Ask Orbi...")
        self.input.returnPressed.connect(self.send)
        input_row.addWidget(self.input)

        send_btn = QPushButton("Send")
        send_btn.clicked.connect(self.send)
        input_row.addWidget(send_btn)

        layout.addLayout(input_row)

        # Controls
        controls = QHBoxLayout()

        color_btn = QPushButton("Color Wheel")
        color_btn.clicked.connect(self.pick_color)
        controls.addWidget(color_btn)

        rainbow_btn = QPushButton("Rainbow Mode")
        rainbow_btn.clicked.connect(self.toggle_rainbow)
        controls.addWidget(rainbow_btn)

        reset_btn = QPushButton("Reset AI")
        reset_btn.clicked.connect(self.reset_all)
        controls.addWidget(reset_btn)

        layout.addLayout(controls)
        main.addLayout(layout, 3)

    def keyPressEvent(self, event):
        if event.key() in (Qt.Key_Return, Qt.Key_Enter):
            self.send()
        else:
            super().keyPressEvent(event)

    # ==========================
    # Chat logic
    # ==========================
    def new_chat(self):
        chat = {"title": "New Chat", "messages": []}
        self.chats.insert(0, chat)
        self.current_chat = chat
        self.refresh_chat_list()
        self.display.clear()
        self.input.setFocus()

    def load_chat(self):
        index = self.chat_list.currentRow()
        self.current_chat = self.chats[index]
        self.display.clear()
        for msg in self.current_chat["messages"]:
            self.append(msg)
        self.input.setFocus()

    def refresh_chat_list(self):
        self.chat_list.clear()
        for c in self.chats:
            self.chat_list.addItem(c["title"])

    def send(self):
        text = self.input.text().strip()
        if not text:
            return

        if not self.current_chat:
            self.new_chat()

        self.input.clear()
        self.append("You: " + text)

        history = self.current_chat["messages"]
        reply = get_ai_response(text, history)

        self.append("Orbi: " + reply)

        history.append("User: " + text)
        history.append("Orbi: " + reply)

        self.current_chat["title"] = summarize(text)

        save_data(self.chats)
        self.refresh_chat_list()

        self.input.setFocus()

    def append(self, text):
        cursor = self.display.textCursor()
        cursor.movePosition(QTextCursor.End)

        fmt = QTextCharFormat()
        fmt.setForeground(self.text_color)

        cursor.insertText(text + "\n", fmt)

    # ==========================
    # UI Features
    # ==========================
    def pick_color(self):
        color = QColorDialog.getColor()
        if color.isValid():
            self.text_color = color
            self.rainbow = False

    def toggle_rainbow(self):
        self.rainbow = not self.rainbow

    def start_pulse(self):
        self.timer = QTimer()
        self.timer.timeout.connect(self.pulse)
        self.timer.start(PULSE_INTERVAL)

    def pulse(self):
        if not self.rainbow:
            return
        self.text_color = QColor(self.colors[self.rainbow_index % len(self.colors)])
        self.rainbow_index += 1

    def reset_all(self):
        if os.path.exists(DATA_FILE):
            os.remove(DATA_FILE)
        self.chats = []
        self.current_chat = None
        self.display.clear()
        self.refresh_chat_list()

# ==========================
# Run
# ==========================
if __name__ == "__main__":
    app = QApplication(sys.argv)
    w = Orbi()
    w.show()
    sys.exit(app.exec())
EOF
