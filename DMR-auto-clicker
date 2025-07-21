import customtkinter as ctk
import win32api
import win32con
import time
import threading

KEY_MAP = {
    0x04: "Middle Button", 0x05: "Side Button 1", 0x06: "Side Button 2",
    0x08: "Backspace", 0x09: "Tab", 0x0D: "Enter", 0x10: "Shift", 0x11: "Ctrl",
    0x12: "Alt", 0x14: "Caps Lock", 0x1B: "Esc", 0x20: "Space", 0x25: "←",
    0x26: "↑", 0x27: "→", 0x28: "↓", 0x2E: "Delete",
    **{i: chr(i) for i in range(ord('0'), ord('9') + 1)},
    **{i: chr(i) for i in range(ord('A'), ord('Z') + 1)},
    **{i + 0x6F: f"F{i}" for i in range(1, 13)},
    0x90: "Num Lock", 0xBA: ";", 0xBB: "=", 0xBC: ",", 0xBD: "-", 0xBE: ".", 0xBF: "/", 0xC0: "'",
    0xDB: "[", 0xDC: "\\", 0xDD: "]", 0xDE: "#"
}

class AutoClickerApp(ctk.CTk):
    def __init__(self):
        super().__init__()

        self.title("DMR Auto Clicker")
        self.geometry("400x400")
        self.resizable(False, False)
        ctk.set_appearance_mode("dark")
        ctk.set_default_color_theme("blue")

        self.hotkey_code = win32con.VK_MBUTTON
        self.hotkey_name = KEY_MAP.get(self.hotkey_code, "Unknown")
        self.is_listening = False
        self.is_clicking = False
        self.master_switch_var = ctk.BooleanVar(value=False)
        self.running_event = threading.Event()
        self.running_event.set()

        self.hotkey_frame = ctk.CTkFrame(self, fg_color="transparent")
        self.hotkey_frame.pack(pady=15, padx=20, fill="x")

        self.cps_frame = ctk.CTkFrame(self, fg_color="transparent")
        self.cps_frame.pack(pady=15, padx=20, fill="x")

        self.control_frame = ctk.CTkFrame(self, fg_color="transparent")
        self.control_frame.pack(pady=20, padx=20, fill="x", side="bottom")

        self.label_hotkey_title = ctk.CTkLabel(self.hotkey_frame, text="⚙️ Activation Hotkey", font=ctk.CTkFont(size=18, weight="bold"))
        self.label_hotkey_title.pack(anchor="w")

        self.hotkey_display_frame = ctk.CTkFrame(self.hotkey_frame, corner_radius=10, height=50)
        self.hotkey_display_frame.pack(pady=10, fill="x")

        self.hotkey_display_label = ctk.CTkLabel(self.hotkey_display_frame, text=self.hotkey_name, font=ctk.CTkFont(size=20, weight="bold"))
        self.hotkey_display_label.pack(expand=True)

        self.btn_set_hotkey = ctk.CTkButton(self.hotkey_frame, text="Set New Hotkey", command=self.listen_for_hotkey)
        self.btn_set_hotkey.pack(fill="x", ipady=5)

        self.label_cps_title = ctk.CTkLabel(self.cps_frame, text="⚡ Speed", font=ctk.CTkFont(size=18, weight="bold"))
        self.label_cps_title.pack(anchor="w")

        self.cps_slider = ctk.CTkSlider(self.cps_frame, from_=1, to=500, number_of_steps=99, command=self.update_cps_label)
        self.cps_slider.set(50)
        self.cps_slider.pack(pady=10, fill="x")

        self.cps_value_label = ctk.CTkLabel(self.cps_frame, text="50 Clicks per Second", font=ctk.CTkFont(size=14, weight="normal"))
        self.cps_value_label.pack()

        self.master_switch = ctk.CTkSwitch(
            self.control_frame, text="Enable Clicker", variable=self.master_switch_var,
            font=ctk.CTkFont(size=16, weight="bold"), command=self.toggle_master_switch
        )
        self.master_switch.pack(side="left")

        self.status_label = ctk.CTkLabel(self.control_frame, text="Disabled", text_color="gray")
        self.status_label.pack(side="right")
        
        self.clicker_thread = threading.Thread(target=self.run_clicker_logic, daemon=True)
        self.clicker_thread.start()
        self.protocol("WM_DELETE_WINDOW", self.on_close)

    def update_cps_label(self, value):
        self.cps_value_label.configure(text=f"{int(value)} Clicks per Second")

    def toggle_master_switch(self):
        if self.master_switch_var.get():
            self.status_label.configure(text="Waiting for Hotkey...", text_color="yellow")
        else:
            self.status_label.configure(text="Disabled", text_color="gray")
            self.is_clicking = False
            
    def listen_for_hotkey(self):
        if self.is_listening: return
        self.is_listening = True
        self.btn_set_hotkey.configure(state="disabled")
        self.hotkey_display_label.configure(text="Press a key...")
        self.hotkey_display_frame.configure(fg_color="#334466")

    def set_new_hotkey(self, code, name):
        self.hotkey_code = code
        self.hotkey_name = name
        self.hotkey_display_label.configure(text=self.hotkey_name)
        self.btn_set_hotkey.configure(state="normal")
        self.hotkey_display_frame.configure(fg_color=ctk.ThemeManager.theme["CTkFrame"]["fg_color"])
        self.is_listening = False

    def run_clicker_logic(self):
        next_click_time = 0
        while self.running_event.is_set():
            time.sleep(0.001)

            if self.is_listening:
                for key_code in KEY_MAP:
                    if win32api.GetAsyncKeyState(key_code) & 0x8000:
                        self.after(0, self.set_new_hotkey, key_code, KEY_MAP[key_code])
                        break
                continue

            if not self.master_switch_var.get():
                time.sleep(0.1)
                continue

            if win32api.GetAsyncKeyState(self.hotkey_code) < 0:
                if not self.is_clicking:
                    self.is_clicking = True
                    self.after(0, self.status_label.configure, {"text": "ACTIVE! Clicking...", "text_color": "lightgreen"})
                    next_click_time = time.perf_counter()
                
                current_time = time.perf_counter()
                if current_time >= next_click_time:
                    win32api.mouse_event(win32con.MOUSEEVENTF_LEFTDOWN, 0, 0, 0, 0)
                    win32api.mouse_event(win32con.MOUSEEVENTF_LEFTUP, 0, 0, 0, 0)
                    click_interval = 1 / self.cps_slider.get()
                    next_click_time += click_interval
            else:
                if self.is_clicking:
                    self.is_clicking = False
                    self.after(0, self.status_label.configure, {"text": "Waiting for Hotkey...", "text_color": "yellow"})
        
    def on_close(self):
        self.running_event.clear()
        self.destroy()

if __name__ == "__main__":
    app = AutoClickerApp()
    app.mainloop()
