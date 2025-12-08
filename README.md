import customtkinter as ctk
from tkinter import messagebox
from datetime import datetime
import random
import json
import os

# ---------------------------- JSON Persistence ----------------------------
if os.path.exists("accounts.json"):
    with open("accounts.json", "r") as f:
        accounts = json.load(f)
else:
    accounts = {
        "123456": {"name": "NERDSQUAD", "pin": "1234", "balance": 1000, "history": []}
    }

def save_accounts():
    with open("accounts.json", "w") as f:
        json.dump(accounts, f, indent=4)

# ---------------------------- Config ----------------------------
MIN_DEPOSIT = 500
MIN_WITHDRAW = 500

ctk.set_appearance_mode("dark")
ctk.set_default_color_theme("green")

# ---------------------------- Base Page ----------------------------
class BasePage(ctk.CTkFrame):
    def _init_(self, parent, controller):
        super()._init_(parent)
        self.controller = controller
        self.card = ctk.CTkFrame(self, width=850, height=600, corner_radius=25,
                                 fg_color=("gray30", "#1b1b1b"), border_width=2, border_color="#1b8a5a")
        self.card.place(relx=0.5, rely=0.5, anchor="center")

# ---------------------------- Neon Button ----------------------------
class NeonButton(ctk.CTkButton):
    def _init_(self, parent, **kwargs):
        super()._init_(parent, **kwargs)
        self.bind("<Enter>", self.on_hover)
        self.bind("<Leave>", self.on_leave)

    def on_hover(self, e):
        self.configure(scale=1.05)

    def on_leave(self, e):
        self.configure(scale=1.0)

# ---------------------------- Login Page ----------------------------
class LoginPage(BasePage):
    def _init_(self, parent, controller):
        super()._init_(parent, controller)
        ctk.CTkLabel(self.card, text="🏦NERDS BANK ",
                     font=ctk.CTkFont(size=36, weight="bold"), text_color="#d4af37").pack(pady=30)

        self.acc_entry = ctk.CTkEntry(self.card, placeholder_text="Account Number",
                                      width=450, height=55, font=ctk.CTkFont(size=20),
                                      fg_color="#2b2b2b", text_color="white", border_width=2, border_color="#1b8a5a")
        self.acc_entry.pack(pady=20)

        self.pin_entry = ctk.CTkEntry(self.card, placeholder_text="PIN", show="*",
                                      width=450, height=55, font=ctk.CTkFont(size=20),
                                      fg_color="#2b2b2b", text_color="white", border_width=2, border_color="#1b8a5a")
        self.pin_entry.pack(pady=20)

        NeonButton(self.card, text="Login", width=350, height=65,
                   fg_color="#1b8a5a", hover_color="#26b070", font=ctk.CTkFont(size=22, weight="bold"),
                   command=self.login).pack(pady=15)
        NeonButton(self.card, text="Create Account", width=350, height=65,
                   fg_color="#4444aa", hover_color="#6666cc", font=ctk.CTkFont(size=22, weight="bold"),
                   command=lambda: controller.show_frame(CreateAccountPage)).pack(pady=15)
        NeonButton(self.card, text="Exit", width=350, height=65,
                   fg_color="#aa3333", hover_color="#cc5555", font=ctk.CTkFont(size=22, weight="bold"),
                   command=self.quit).pack(pady=15)

    def login(self):
        acc = self.acc_entry.get().strip()
        pin = self.pin_entry.get().strip()
        if acc in accounts and accounts[acc]["pin"] == pin:
            self.controller.current_account = acc
            messagebox.showinfo("Welcome", f"Hello {accounts[acc]['name']} 👑")
            self.controller.show_frame(MenuPage)
        else:
            messagebox.showerror("Error", "Invalid Account Number or PIN")

# ---------------------------- Create Account Page ----------------------------
class CreateAccountPage(BasePage):
    def _init_(self, parent, controller):
        super()._init_(parent, controller)

        ctk.CTkLabel(self.card, text="👤 Create New Account",
                     font=ctk.CTkFont(size=36, weight="bold")).pack(pady=30)

        self.name_entry = ctk.CTkEntry(self.card, placeholder_text="Full Name",
                                       width=450, height=55, font=ctk.CTkFont(size=20))
        self.name_entry.pack(pady=20)

        self.pin_entry = ctk.CTkEntry(self.card, placeholder_text="4-digit PIN", show="*",
                                      width=450, height=55, font=ctk.CTkFont(size=20))
        self.pin_entry.pack(pady=20)

        NeonButton(self.card, text="Create Account", width=350, height=65,
                   fg_color="#1b8a5a", hover_color="#26b070",
                   font=ctk.CTkFont(size=22, weight="bold"), command=self.create_account).pack(pady=15)
        NeonButton(self.card, text="Back", width=350, height=65,
                   fg_color="#555555", hover_color="#777777",
                   font=ctk.CTkFont(size=22, weight="bold"),
                   command=lambda: controller.show_frame(LoginPage)).pack(pady=15)

    def create_account(self):
        name = self.name_entry.get().strip()
        pin = self.pin_entry.get().strip()
        if not name or not pin.isdigit() or len(pin) != 4:
            messagebox.showerror("Error", "Enter a valid name and 4-digit PIN")
            return
        acc_num = str(random.randint(100000, 999999))
        accounts[acc_num] = {"name": name, "pin": pin, "balance": 0, "history": []}
        save_accounts()
        messagebox.showinfo("Success", f"Account Created!\nNumber: {acc_num}\nPIN: {pin}")
        self.controller.show_frame(LoginPage)

# ---------------------------- Menu Page ----------------------------
class MenuPage(BasePage):
    def _init_(self, parent, controller):
        super()._init_(parent, controller)
        ctk.CTkLabel(self.card, text="🏦 MAIN MENU", font=ctk.CTkFont(size=36, weight="bold")).pack(pady=30)

        btn_params = {"width": 350, "height": 70, "font": ctk.CTkFont(size=22, weight="bold"),
                      "corner_radius": 15, "fg_color": "#1b8a5a"}

        NeonButton(self.card, text="💰 Check Balance", command=lambda: controller.show_frame(BalancePage), **btn_params).pack(pady=15)
        NeonButton(self.card, text="📥 Deposit Money", command=lambda: controller.show_frame(DepositPage), **btn_params).pack(pady=15)
        NeonButton(self.card, text="💸 Withdraw Money", command=lambda: controller.show_frame(WithdrawPage), **btn_params).pack(pady=15)
        NeonButton(self.card, text="📜 Transaction History", command=lambda: controller.show_frame(HistoryPage), **btn_params).pack(pady=15)
        NeonButton(self.card, text="🚪 Logout", width=350, height=70, font=ctk.CTkFont(size=22, weight="bold"),
                   corner_radius=15, fg_color="#aa3333", hover_color="#cc5555", command=self.logout).pack(pady=20)

    def logout(self):
        self.controller.current_account = None
        self.controller.show_frame(LoginPage)

# ---------------------------- Balance Page ----------------------------
class BalancePage(BasePage):
    def _init_(self, parent, controller):
        super()._init_(parent, controller)
        self.label = ctk.CTkLabel(self.card, text="", font=ctk.CTkFont(size=32, weight="bold"))
        self.label.pack(pady=50)
        NeonButton(self.card, text="⬅ Back", width=300, height=60, font=ctk.CTkFont(size=22, weight="bold"),
                   fg_color="#555555", hover_color="#777777", command=lambda: controller.show_frame(MenuPage)).pack(pady=20)

    def refresh(self):
        acc = self.controller.current_account
        self.label.configure(text=f"💰 Balance: ₹{accounts[acc]['balance']:,}")

# ---------------------------- Deposit Page ----------------------------
class DepositPage(BasePage):
    def _init_(self, parent, controller):
        super()._init_(parent, controller)
        ctk.CTkLabel(self.card, text=f"Minimum Deposit: ₹{MIN_DEPOSIT}", font=ctk.CTkFont(size=20)).pack(pady=15)
        self.entry = ctk.CTkEntry(self.card, placeholder_text="Enter amount", width=400, height=55, font=ctk.CTkFont(size=20))
        self.entry.pack(pady=20)
        NeonButton(self.card, text="Deposit", width=300, height=65, font=ctk.CTkFont(size=22, weight="bold"),
                   fg_color="#1b8a5a", hover_color="#26b070", command=self.deposit).pack(pady=15)
        NeonButton(self.card, text="⬅ Back", width=300, height=65, font=ctk.CTkFont(size=22, weight="bold"),
                   fg_color="#555555", hover_color="#777777", command=lambda: controller.show_frame(MenuPage)).pack(pady=15)

    def deposit(self):
        acc = self.controller.current_account
        try:
            amt = float(self.entry.get())
            if amt < MIN_DEPOSIT:
                raise ValueError(f"Minimum deposit is ₹{MIN_DEPOSIT}")
        except Exception as e:
            messagebox.showerror("Error", str(e))
            return
        accounts[acc]["balance"] += amt
        accounts[acc]["history"].append(f"{datetime.now().strftime('%d-%m-%Y %H:%M:%S')} | Deposited ₹{amt:.2f}")
        save_accounts()
        messagebox.showinfo("Success", f"₹{amt:.2f} Deposited Successfully!")
        self.entry.delete(0, 'end')

# ---------------------------- Withdraw Page ----------------------------
class WithdrawPage(BasePage):
    def _init_(self, parent, controller):
        super()._init_(parent, controller)
        ctk.CTkLabel(self.card, text=f"Minimum Withdrawal: ₹{MIN_WITHDRAW}", font=ctk.CTkFont(size=20)).pack(pady=15)
        self.entry = ctk.CTkEntry(self.card, placeholder_text="Enter amount", width=400, height=55, font=ctk.CTkFont(size=20))
        self.entry.pack(pady=20)
        NeonButton(self.card, text="Withdraw", width=300, height=65, font=ctk.CTkFont(size=22, weight="bold"),
                   fg_color="#1b8a5a", hover_color="#26b070", command=self.withdraw).pack(pady=15)
        NeonButton(self.card, text="⬅ Back", width=300, height=65, font=ctk.CTkFont(size=22, weight="bold"),
                   fg_color="#555555", hover_color="#777777", command=lambda: controller.show_frame(MenuPage)).pack(pady=15)

    def withdraw(self):
        acc = self.controller.current_account
        try:
            amt = float(self.entry.get())
            if amt < MIN_WITHDRAW:
                raise ValueError(f"Minimum withdrawal is ₹{MIN_WITHDRAW}")
            if amt > accounts[acc]["balance"]:
                raise ValueError("Insufficient Balance")
        except Exception as e:
            messagebox.showerror("Error", str(e))
            return
        accounts[acc]["balance"] -= amt
        accounts[acc]["history"].append(f"{datetime.now().strftime('%d-%m-%Y %H:%M:%S')} | Withdrawn ₹{amt:.2f}")
        save_accounts()
        messagebox.showinfo("Success", f"₹{amt:.2f} Withdrawn Successfully!")
        self.entry.delete(0, 'end')

# ---------------------------- History Page ----------------------------
class HistoryPage(BasePage):
    def _init_(self, parent, controller):
        super()._init_(parent, controller)
        self.text = ctk.CTkTextbox(self.card, width=820, height=500, font=ctk.CTkFont(size=16))
        self.text.pack(pady=(10,10))
        NeonButton(self.card, text="⬅ Back", width=300, height=65, font=ctk.CTkFont(size=22, weight="bold"),
                   fg_color="#555555", hover_color="#777777", command=lambda: controller.show_frame(MenuPage)).pack(pady=10)

    def refresh(self):
        acc = self.controller.current_account
        self.text.delete("1.0", "end")
        if accounts[acc]["history"]:
            for h in reversed(accounts[acc]["history"]):
                self.text.insert("end", h + "\n")
        else:
            self.text.insert("end", "No transactions yet.")

# ---------------------------- Main App ----------------------------
class ATMApp(ctk.CTk):
    def _init_(self):
        super()._init_()
        self.title("🏦 BENNETT BANKING SYSTEM")
        self.state("zoomed")  # full screen
        self.bind("<F11>", lambda e: self.attributes("-fullscreen", not self.attributes("-fullscreen")))
        self.bind("<Escape>", lambda e: self.state("zoomed"))
        self.current_account = None

        self.frames = {}
        for F in (LoginPage, CreateAccountPage, MenuPage, BalancePage, DepositPage, WithdrawPage, HistoryPage):
            frame = F(self, self)
            self.frames[F] = frame
            frame.place(relx=0, rely=0, relwidth=1, relheight=1)
        self.show_frame(LoginPage)

    def show_frame(self, page):
        frame = self.frames[page]
        frame.tkraise()
        if hasattr(frame, "refresh"):
            frame.refresh()

# ---------------------------- Run App ----------------------------
if _name_ == "_main_":
    app = ATMApp()
    app.mainloop()
