

# PCAN-Explorer 7 Native Python Macro (.pym)

def CreateEPASPlots():
    # 1. Clear out old messy plots
    doc_list = []
    for i in range(app.Documents.Count):
        doc_list.append(app.Documents.Item(i + 1))
        
    for doc in doc_list:
        if doc.Name.lower().endswith(".plt") or "plot" in doc.Name.lower():
            doc.Close()

    # 2. Target your physical EPAS signals based on the live instrumentation panel
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

    # Helper function to generate and space out the plot layouts
    def build_plot(title, signals, left, top, width, height):
        # peDocumentKindPlotter = 5
        doc = app.Documents.Add(5) 
        doc.Name = title
        
        win = doc.ActiveWindow
        win.Left = left
        win.Top = top
        win.Width = width
        win.Height = height
        
        plotter = win.Object
        
        for sig in signals:
            try:
                plotter.AddVariable(sig)
            except Exception as e:
                print("Could not bind signal: " + str(sig))

    # 3. Render side-by-side grids on your screen
    build_plot("EPAS_Electrical", electrical_signals, 0, 0, 650, 450)
    build_plot("EPAS_Thermal", thermal_signals, 655, 0, 650, 450)

    # Sync timelines horizontally
    try:
        app.Commands.Execute("Plotter:EnableXAxisSync")
    except:
        pass

# Run macro
CreateEPASPlots()
