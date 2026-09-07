

# PCAN-Explorer 7 Internal Python Macro

def CreateEPASPlots():
    # 1. Close existing messy plots to free screen space
    doc_list = []
    for i in range(Application.Documents.Count):
        doc_list.append(Application.Documents.Item(i + 1))
        
    for doc in doc_list:
        if doc.Name.lower().endswith(".plt") or "plot" in doc.Name.lower():
            doc.Close()

    # 2. Define your exact EPAS signal listings based on your live symbols
    electrical_signals = [
        "EPAS_Volt.EMduleInCrct_U_Actl2",
        "EPAS_Volt.EMduleOutCrct_U_Actl2",
        "EPAS_Volt.Cell_U_Actl2",
        "EPAS_Current.EMduleInCrct_I_Actl2",
        "EPAS_Current.EMduleOutCrct_I_Actl2"
    ]

    thermal_signals = [
        "EPAS_Temperature.FET_Te_Actl2",
        "EPAS_Temperature.Cel_Te_Actl2"
    ]

    # Helper function to create and position each plot window
    def build_plot(title, signals, left, top, width, height):
        # 5 corresponds to peDocumentKindPlotter
        doc = Application.Documents.Add(5) 
        doc.Name = title
        
        win = doc.ActiveWindow
        win.Left = left
        win.Top = top
        win.Width = width
        win.Height = height
        
        plotter = win.Object
        
        for sig in signals:
            try:
                # Add variable directly to the plotter scene view
                plotter.AddVariable(sig)
            except Exception as e:
                print("Could not bind signal: " + str(sig))

    # 3. Launch and layout the windows side-by-side cleanly
    build_plot("EPAS_Electrical", electrical_signals, 0, 0, 650, 450)
    build_plot("EPAS_Thermal", thermal_signals, 655, 0, 650, 450)

    # Enable timeline synchronization across windows
    try:
        Application.Commands.Execute("Plotter:EnableXAxisSync")
    except:
        pass

# Execute the macro function directly
CreateEPASPlots()
