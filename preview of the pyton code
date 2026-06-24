import os
import subprocess
import tkinter as tk
from tkinter import ttk, filedialog, messagebox
# Trova la cartella esatta dove si trova questo script python
BASE_DIR = os.path.dirname(os.path.abspath(__file__))
# Unisce la cartella al nome del file
PATH_FILE = os.path.join(BASE_DIR, "path.txt")
# -------------------

class DolphinIniConfigurator:
    def __init__(self, root):
        self.root = root
        self.root.title("Dolphin INI Configurator")
        self.root.geometry("800x600")
        
        self.config_dir = ""
        self.dolphin_dir = ""
        self.preset_dir = ""
        self.rom_dirs = []
        
        self.games = []  # Lista di dizionari con i dati dei giochi
        self.gui_vars = []  # Variabili dell'interfaccia collegate ai giochi
        
        self.check_and_load_paths()

    def check_and_load_paths(self):
        # Legge il file path.txt o avvia il setup iniziale
        if os.path.exists(PATH_FILE):
            with open(PATH_FILE, 'r', encoding='utf-8') as f:
                lines = [line.strip() for line in f.readlines()]
                
            if len(lines) >= 4:
                self.config_dir = lines[0]
                self.dolphin_dir = lines[1]
                self.preset_dir = lines[2]
                self.rom_dirs = [line for line in lines[3:] if line]
                self.build_main_gui()
                return

        # Se il file non esiste o è incompleto, apri il setup dei percorsi
        self.build_settings_gui()

    # ==========================================
    # FASE 1: GUI IMPOSTAZIONI INIZIALI
    # ==========================================
    def build_settings_gui(self):
        self.clear_window()
        
        frame = ttk.Frame(self.root, padding=20)
        frame.pack(fill=tk.BOTH, expand=True)
        
        ttk.Label(frame, text="Path assignation", font=("Arial", 16, "bold")).pack(pady=10)
        
        # 1. Cartella Config (INI)
        ttk.Label(frame, text=r"1. GameSettings Folder path (default: C:\Users\[your user]\AppData\Roaming\Dolphin Emulator\GameSettings):").pack(anchor=tk.W, pady=(10, 0))
        self.ent_config = ttk.Entry(frame, width=80)
        self.ent_config.pack(anchor=tk.W)
        ttk.Button(frame, text="Browse", command=lambda: self.browse_folder(self.ent_config)).pack(anchor=tk.W, pady=5)
        
        # 2. Cartella DolphinTool
        ttk.Label(frame, text="2. Dolphin tool folder (the same folder of dolphin.exe):").pack(anchor=tk.W, pady=(10, 0))
        self.ent_dolphin = ttk.Entry(frame, width=80)
        self.ent_dolphin.pack(anchor=tk.W)
        ttk.Button(frame, text="Browse", command=lambda: self.browse_folder(self.ent_dolphin)).pack(anchor=tk.W, pady=5)

        # 3. Cartella Preset
        ttk.Label(frame, text=r"3. Controller preset folder (default: C:\Users\[your user]\AppData\Roaming\Dolphin Emulator\Config\Profiles\Wiimote):").pack(anchor=tk.W, pady=(10, 0))
        self.ent_preset = ttk.Entry(frame, width=80)
        self.ent_preset.pack(anchor=tk.W)
        ttk.Button(frame, text="Browse", command=lambda: self.browse_folder(self.ent_preset)).pack(anchor=tk.W, pady=5)

        # 4. Cartelle ROMs
        ttk.Label(frame, text="4. ROMs folders (you can add all of your rom folders here):").pack(anchor=tk.W, pady=(10, 0))
        
        roms_frame = ttk.Frame(frame)
        roms_frame.pack(fill=tk.BOTH, expand=True, pady=5)
        
        self.listbox_roms = tk.Listbox(roms_frame, height=5, width=70)
        self.listbox_roms.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)
        
        btn_frame = ttk.Frame(roms_frame)
        btn_frame.pack(side=tk.LEFT, padx=10)
        ttk.Button(btn_frame, text="Add this folder", command=self.add_rom_folder).pack(fill=tk.X, pady=2)
        ttk.Button(btn_frame, text="Remove selected", command=self.remove_rom_folder).pack(fill=tk.X, pady=2)

        # Pulsante Salva
        ttk.Button(frame, text="Save and continue", command=self.save_settings).pack(pady=20)

    def browse_folder(self, entry_widget):
        folder = filedialog.askdirectory()
        if folder:
            entry_widget.delete(0, tk.END)
            entry_widget.insert(0, folder)

    def add_rom_folder(self):
        folder = filedialog.askdirectory()
        if folder:
            self.listbox_roms.insert(tk.END, folder)

    def remove_rom_folder(self):
        selection = self.listbox_roms.curselection()
        if selection:
            self.listbox_roms.delete(selection)

    def save_settings(self):
        self.config_dir = self.ent_config.get().strip()
        self.dolphin_dir = self.ent_dolphin.get().strip()
        self.preset_dir = self.ent_preset.get().strip()
        self.rom_dirs = list(self.listbox_roms.get(0, tk.END))

        if not self.config_dir or not self.dolphin_dir or not self.rom_dirs:
            messagebox.showwarning("Attention!", "the config folders, dolphin and at least a rom folder are obligatory")
            return

        with open(PATH_FILE, 'w', encoding='utf-8') as f:
            f.write(f"{self.config_dir}\n")
            f.write(f"{self.dolphin_dir}\n")
            f.write(f"{self.preset_dir}\n")
            for r in self.rom_dirs:
                f.write(f"{r}\n")
                
        self.build_main_gui()

    # ==========================================
    # FASE 2: LETTURA ROM E GUI PRINCIPALE
    # ==========================================
    def build_main_gui(self):
        self.clear_window()
        
        ttk.Label(self.root, text="Game research in action...", font=("Arial", 14)).pack(pady=50)
        self.root.update()
        
        self.scan_roms()
        
        self.clear_window()
        
        # Titolo e Controlli Globali
        top_frame = ttk.Frame(self.root, padding=10)
        top_frame.pack(fill=tk.X)
        ttk.Label(top_frame, text="Dolphin Auto Config", font=("Arial", 16, "bold")).pack(side=tk.LEFT)
        ttk.Button(top_frame, text="Change paths", command=self.build_settings_gui).pack(side=tk.RIGHT)
        
        # Frame Scorrevole (Canvas)
        canvas = tk.Canvas(self.root)
        scrollbar = ttk.Scrollbar(self.root, orient="vertical", command=canvas.yview)
        scrollable_frame = ttk.Frame(canvas)

        scrollable_frame.bind(
            "<Configure>",
            lambda e: canvas.configure(scrollregion=canvas.bbox("all"))
        )

        canvas.create_window((0, 0), window=scrollable_frame, anchor="nw")
        canvas.configure(yscrollcommand=scrollbar.set)

        canvas.pack(side="left", fill="both", expand=True, padx=10, pady=10)
        scrollbar.pack(side="right", fill="y")

        # Cerca i preset disponibili
        available_presets = []
        if self.preset_dir and os.path.exists(self.preset_dir):
            available_presets = [f.replace('.ini', '') for f in os.listdir(self.preset_dir) if f.endswith('.ini')]

        # Intestazioni di colonna
        headers = ["File Name", "Game ID", "BT Passthrough", "Type of Wiimote", "Preset (type or select)"]
        for col, text in enumerate(headers):
            ttk.Label(scrollable_frame, text=text, font=("Arial", 10, "bold")).grid(row=0, column=col, padx=5, pady=5, sticky=tk.W)

        self.gui_vars = []
        
        # Crea una riga per ogni gioco trovato
        for i, game in enumerate(self.games):
            row = i + 1
            
            # File / ID
            ttk.Label(scrollable_frame, text=game['filename'][:30]).grid(row=row, column=0, padx=5, pady=2, sticky=tk.W)
            ttk.Label(scrollable_frame, text=game['game_id']).grid(row=row, column=1, padx=5, pady=2, sticky=tk.W)
            
            # Variabili
            var_bt = tk.StringVar(value="")
            var_wii = tk.StringVar(value="")
            var_preset = tk.StringVar(value="")
            
            # 1. Radiobuttons BT Passthrough (Yes / No)
            frame_bt = ttk.Frame(scrollable_frame)
            frame_bt.grid(row=row, column=2, padx=5, pady=2)
            rb_bt_yes = ttk.Radiobutton(frame_bt, text="Yes", variable=var_bt, value="yes")
            rb_bt_no = ttk.Radiobutton(frame_bt, text="No", variable=var_bt, value="no")
            rb_bt_yes.pack(side=tk.LEFT, padx=2)
            rb_bt_no.pack(side=tk.LEFT, padx=2)
            
            # 2. Radiobuttons Tipo Wiimote (Real / Emulated) - Disabilitati di default
            frame_wii = ttk.Frame(scrollable_frame)
            frame_wii.grid(row=row, column=3, padx=5, pady=2)
            rb_wii_real = ttk.Radiobutton(frame_wii, text="Real", variable=var_wii, value="real", state=tk.DISABLED)
            rb_wii_emu = ttk.Radiobutton(frame_wii, text="Emulated", variable=var_wii, value="emu", state=tk.DISABLED)
            rb_wii_real.pack(side=tk.LEFT, padx=2)
            rb_wii_emu.pack(side=tk.LEFT, padx=2)
            
            # 3. Combobox Preset - Disabilitata di default
            cmb_preset = ttk.Combobox(scrollable_frame, textvariable=var_preset, values=available_presets, width=20, state=tk.DISABLED)
            cmb_preset.grid(row=row, column=4, padx=5, pady=2)
            
            # Logica per abilitare/disabilitare i campi successivi in base alla scelta del BT
            def make_callback(w_real, w_emu, cmb, v_bt, v_wii, v_pres):
                def update_state(*args):
                    if v_bt.get() == "no":
                        w_real.config(state=tk.NORMAL)
                        w_emu.config(state=tk.NORMAL)
                        cmb.config(state=tk.NORMAL)
                    else:
                        w_real.config(state=tk.DISABLED)
                        w_emu.config(state=tk.DISABLED)
                        cmb.config(state=tk.DISABLED)
                        v_wii.set("")
                        v_pres.set("")
                return update_state
            
            var_bt.trace_add("write", make_callback(rb_wii_real, rb_wii_emu, cmb_preset, var_bt, var_wii, var_preset))
            
            self.gui_vars.append({
                "game_id": game['game_id'],
                "var_bt": var_bt,
                "var_wii": var_wii,
                "var_preset": var_preset
            })
            
        # Pulsante Generazione
        bot_frame = ttk.Frame(self.root, padding=10)
        bot_frame.pack(fill=tk.X)
        ttk.Button(bot_frame, text="Generate INI Files", command=self.generate_inis).pack(pady=10)

    def scan_roms(self):
        self.games = []
        dolphin_tool_path = os.path.join(self.dolphin_dir, "DolphinTool.exe")
        
        for directory in self.rom_dirs:
            if not os.path.exists(directory):
                continue
                
            for filename in os.listdir(directory):
                full_path = os.path.join(directory, filename)
                if not os.path.isfile(full_path):
                    continue
                    
                ext = filename.lower().split('.')[-1]
                game_id = "Unknown"
                
                try:
                    if ext == 'iso':
                        with open(full_path, 'rb') as file_file:
                            first_6_bytes = file_file.read(6)
                        game_id = first_6_bytes.decode('utf-8', errors='replace').strip()
                        
                    elif ext in ['wbfs', 'gcm', 'rvz', 'wia', 'wad']:
                        if os.path.exists(dolphin_tool_path):
                            cmd = [dolphin_tool_path, "header", "-i", full_path]
                            result = subprocess.run(cmd, capture_output=True, text=True, check=True)
                            for line in result.stdout.splitlines():
                                if "game id" in line.lower() or "id:" in line.lower():
                                    game_id = line.split(':')[-1].strip()
                                    break
                            else:
                                game_id = result.stdout.strip()[:6]
                except Exception as e:
                    print(f"Errore lettura {filename}: {e}")
                    
                if game_id and game_id != "Unknown" and len(game_id) > 0:
                    self.games.append({"filename": filename, "game_id": game_id})

    # ==========================================
    # FASE 3: SCRITTURA FILE INI
    # ==========================================
    def generate_inis(self):
        if not os.path.exists(self.config_dir):
            try:
                os.makedirs(self.config_dir)
            except Exception as e:
                messagebox.showerror("Error", f"Impossible to create config folder:\n{e}")
                return

        generated_count = 0
        
        for item in self.gui_vars:
            game_id = item['game_id']
            bt_choice = item['var_bt'].get()
            wii_choice = item['var_wii'].get()
            preset = item['var_preset'].get().strip()
            
            if bt_choice == "" or (bt_choice == "no" and wii_choice == ""):
                continue
                
            ini_content = "[Dolphin.BluetoothPassthrough]\n"
            
            if bt_choice == "yes":
                ini_content += "Enabled = True\n"
            elif bt_choice == "no":
                ini_content += "Enabled = False\n"
                ini_content += "[Controls]\n"
                
                if wii_choice == "real":
                    ini_content += "WiimoteSource0 = 2\n"
                elif wii_choice == "emu":
                    if preset != "":
                        ini_content += f"WiimoteProfile1 = {preset}\n"
                    ini_content += "WiimoteSource0 = 1\n"
            
            file_path = os.path.join(self.config_dir, f"{game_id}.ini")
            try:
                with open(file_path, "w", encoding="utf-8") as file:
                    file.write(ini_content)
                generated_count += 1
            except Exception as e:
                print(f"Errore scrittura {game_id}.ini: {e}")

        messagebox.showinfo("Complete", f"Operation complete.\nINI files generated: {generated_count}")

    def clear_window(self):
        for widget in self.root.winfo_children():
            widget.destroy()

if __name__ == "__main__":
    root = tk.Tk()
    app = DolphinIniConfigurator(root)
    root.mainloop()
