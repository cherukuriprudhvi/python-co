

for win in App.Windows:
    try:
        print(win.Caption, "| KIND:", win.Kind, "| OBJECT:", win.Object)
    except Exception as e:
        print("WINDOW ERROR:", e)