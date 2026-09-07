

# PCAN-Explorer 7 Native Python Macro (.pym)

def CreateEPASPlots():

    # 1. Clear out old plot documents
    doc_list = []

    for i in range(App.Documents.Count):
        doc_list.append(App.Documents.Item(i + 1))

    for doc in doc_list:
        try:
            if doc.Name.lower().endswith(".plt") or "plot" in doc.Name.lower():
                doc.Close()
        except:
            pass


    # 2. EPAS signal groups

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


    # 3. Helper function to create a plot

    def build_plot(title, signals, left, top, width, height):

        # Test creating a Plotter document
        doc = App.Documents.Add(5)

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
                print("Added:", sig)

            except Exception as e:
                print("Could not bind:", sig)
                print(e)


    # 4. Create two test plots

    build_plot(
        "EPAS_Electrical",
        electrical_signals,
        0,
        0,
        650,
        450
    )

    build_plot(
        "EPAS_Thermal",
        thermal_signals,
        655,
        0,
        650,
        450
    )


    # 5. Try X-axis synchronization

    try:
        App.Commands.Execute(
            "Plotter:EnableXAxisSync"
        )

        print("X-axis synchronization enabled")

    except Exception as e:
        print("X-axis sync command not available")
        print(e)


    print("EPAS PLOT TEST FINISHED")


# Run macro
CreateEPASPlots()