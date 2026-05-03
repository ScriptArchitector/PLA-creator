# ==============================================================================
# █▀▀ █  █ ▀█▀ █▀▀ █▀▀▄ █▀▀ ▀█▀ █▀▀█ █▀▀▄   █▀▀█ 
# █▀▀ ▄▀▄  ░█░ █▀▀ █  █ ▀▀█  █  █  █ █  █   █▄▄█ 
# ▀▀▀ ▀ ▀   ▀  ▀▀▀ ▀  ▀ ▀▀▀ ▀▀▀ ▀▀▀▀ ▀  ▀   █    
#
# EXTENSION.P LAUNCHER | COMPLETE DEVELOPER REFERENCE MANUAL (v1.21.1)
# ==============================================================================
# WARNING: This documentation describes how to work with the patched launcher 
# core. To use the extended API, the CoreAccess (Level 3) plugin MUST be 
# present in the /plugins/ folder.

# ------------------------------------------------------------------------------
# CHAPTER 1: ARCHITECTURE AND FILE SYSTEM
# ------------------------------------------------------------------------------
# The launcher is based in the %APPDATA%\.p_launcher directory.
# 
# Key directories and files (accessible via 'app' object properties):
# ● app.base_dir     -> The launcher's root folder.
# ● app.plugins_dir  -> Folder for compiled plugins (.pla).
# ● app.mods_dir     -> Folder for Minecraft modifications.
# ● app.rt_dir       -> Runtime environment folder (Java Runtime).
# ● app.cfg_path     -> Path to engine_settings.json (RAM, nickname, resolution).
# ● app.log_cfg_path -> Path to log.json (terminal settings).
# ● app.core_db      -> Path to db.json (plugin activity database).

# ------------------------------------------------------------------------------
# CHAPTER 2: SECURITY SYSTEM AND PLAMANAGER
# ------------------------------------------------------------------------------
# 1. Kill-Switch (creator.json):
#    PLAManager will not load any plugins if the creator.json file is missing 
#    from the root directory, or if it lacks: {"Pla manager app": "true"}.
# 2. Serialization (.pla files):
#    Plugins are not plain text. They are Python bytecode that has gone through:
#    Compilation -> Marshal -> Zlib compression -> Base85 encoding.
# 3. Isolation:
#    Each plugin executes in its own namespace.
#    The entry point is the `main(app)` function.

# ------------------------------------------------------------------------------
# CHAPTER 3: COREACCESS API (FULL METHODS REFERENCE)
# ------------------------------------------------------------------------------
# Inside the main(app) function, you have access to the 'app' object. Use these:

# ● app.reg_sidebar(name_string)
#   Locates the launcher's sidebar (width 220px) and returns it as a parent 
#   frame for your UI.
#   Side Effect: Registers a "sidebar_use" hook for your plugin in db.json.

# ● app.get_val(key_string, default_value=None)
#   Fast and safe reading from app.cfg (engine_settings.json).
#   Example keys: "ram_gb", "nickname", "res", "fs", "terminal_mode".

# ● app.set_status(text_string, progress_float=None)
#   Controls the bottom bar of the launcher (ps_status and ps_bar).
#   text_string: Status text (e.g., "Loading resources...").
#   progress_float: Number from 0.0 to 1.0 to fill the orange progress bar.

# ● app.write_log(text_string)
#   Safely writes a string to the launcher's built-in terminal/log.
#   This method automatically handles the text box state (normal/disabled).

# ------------------------------------------------------------------------------
# CHAPTER 4: ADDITIONAL BUILT-IN FEATURES (MAINAPP & JAVAENGINE)
# ------------------------------------------------------------------------------
# ● app.cfg
#   A dictionary (dict) with current settings. Can be modified directly:
#   app.cfg["ram_gb"] = 8
#   app.save_cfg() # You MUST call this to save changes to the disk!

# ● app.java_engine.get_path()
#   Returns the absolute path to java.exe. It first looks for Eclipse Temurin 
#   JRE 21 in the runtime/ folder; if not found, it checks the system Java.

# ------------------------------------------------------------------------------
# CHAPTER 5: INTERFACE STANDARDS (CUSTOMTKINTER)
# ------------------------------------------------------------------------------
# To maintain the "native" look, use the launcher's system colors:
# - WIN_BG (Window Background): "#191919"
# - WIN_SIDEBAR (Sidebar):      "#202020"
# - WIN_STATUS (Bottom Bar):    "#1E1E1E"
# - WIN_TEXT (Text):            "#FFFFFF"
# - WIN_MUTED (Dimmed Text):    "#AAAAAA"
# - WIN_ACCENT (Orange):        "#FFA54C" (Use for buttons/progress bars)
# - WIN_HOVER (Hover state):    "#2D2D2D"
# - WIN_BORDER (Borders):       "#333333"

# Fonts:
# - Headers: font=("Segoe UI", 16, "bold")
# - Text/Buttons: font=("Segoe UI", 13)
# - Logs/Terminal: font=("Consolas", 11)

# ------------------------------------------------------------------------------
# CHAPTER 6: LAUNCHER TERMINAL AND PLUGIN COMPILATION
# ------------------------------------------------------------------------------
# If terminal_mode = True, the command line is available to the user.
# Built-in commands:
# > --mods             (Opens the mods folder)
# > --memory [GB]      (Sets allocated RAM)
# > --nick [Name]      (Changes nickname)
# > --reset            (Resets config to defaults)
# > --install-fabric   (Background installation of Fabric for version 1.21.1)
# > --clear            (Clears the log screen)

# CREATING A PLUGIN (.PLA):
# 1. Write your code in a script.py file (see Template below).
# 2. Download the official PLA-Creator utility via this link:
#    >>> https://github.com/ScriptArchitector/PLA-creator/releases/latest
#    It will automatically package your code into the correct format.
# 3. Or use the built-in compiler: > --compile C:\path\to\script.py
# 4. Move the resulting script.pla into the /plugins/ folder.

# ------------------------------------------------------------------------------
# CHAPTER 7: FULL PLUGIN TEMPLATE (READY-TO-COMPILE)
# ------------------------------------------------------------------------------
# STRICT REQUIREMENT: Plugin metadata must be provided as comments using '#'.
# Plugins without these comment tags will not be accepted into the PLA Store.
# Author: Your Nickname / Team Name
# Version: 1.0

import customtkinter as ctk
import os
import threading

def main(app):
    # Check for CoreAccess privileges
    if getattr(app, 'is_dev_access', False) is False:
        app.write_log("[MyPlugin] Denied: CoreAccess is required!")
        return

    # 1. Register in the sidebar (Will be logged in db.json)
    plugin_frame = app.reg_sidebar("MyModLoader")
    
    # 2. Create the UI
    lbl = ctk.CTkLabel(plugin_frame, text="MOD LOADER", text_color="#FFA54C", 
                       font=("Segoe UI", 14, "bold"))
    lbl.pack(pady=(10, 5))
    
    def do_work():
        app.set_status("Processing data...", progress=0.3)
        app.write_log("[Plugin] Reading config...")
        
        current_ram = app.get_val("ram_gb", 4)
        app.write_log(f"[MyModLoader] Allocated memory: {current_ram} GB")
        
        def bg_task():
            import time
            time.sleep(2)
            app.set_status("Done!", progress=1.0)
            app.write_log("[MyModLoader] Operation completed successfully.")
            
        threading.Thread(target=bg_task, daemon=True).start()

    btn = ctk.CTkButton(plugin_frame, text="ANALYZE", command=do_work, 
                        fg_color="#FFA54C", text_color="black", hover_color="#CC7A00")
    btn.pack(padx=10, pady=10, fill="x")

# ------------------------------------------------------------------------------
# CHAPTER 8: PLA STORE PUBLISHING RULES
# ------------------------------------------------------------------------------
# The launcher's store downloads plugins directly from the GitHub API.
# Want to add your plugin to the official store? 
# Send your source .py code and a description to: plateamdevs@gmail.com

# Mandatory condition: The code MUST contain the author's name as a comment 
# at the top of the file (e.g., # Author: Developer Name).

# LEVEL STRUCTURE (.pla):
# ● 3.pla: CRITICAL UPDATES. 
#   System modules from Extension.P developers. 
#   IMPORTANT: Custom and third-party plugins NEVER receive this level.
#   Synchronized automatically upon every launch.
#
# ● 2.pla: RECOMMENDED. 
#   Plugins that significantly improve the launcher, adding new functions 
#   and heavily expanding its core capabilities.
#
# ● 1.pla: STANDARD. 
#   Plugins whose functionality is aimed exclusively at customizing the 
#   launcher (changing the interface, themes, and visual elements).
# ==============================================================================
