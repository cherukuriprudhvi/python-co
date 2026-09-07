

Option Explicit

Sub Create_CAN1_EPAS_Grouped()

    Dim doc, plotter
    Dim axVolt, axCurr, axMode, axIso, axTemp
    Dim ch

    Set doc = Documents.Add(peDocumentKindPlotter)
    Set plotter = doc.ActiveWindow.Object

    'Voltage axis
    Set axVolt = plotter.YAxes(1)
    axVolt.Title = "EPAS_Volt"
    axVolt.Min = 0
    axVolt.Max = 20

    'Current axis
    Set axCurr = plotter.YAxes.Add()
    axCurr.Title = "EPAS_Current"
    axCurr.Min = -50
    axCurr.Max = 50

    'Mode / Status axis
    Set axMode = plotter.YAxes.Add()
    axMode.Title = "EPAS_SSR_Mode"
    axMode.Min = 0
    axMode.Max = 10

    'Isolation axis
    Set axIso = plotter.YAxes.Add()
    axIso.Title = "EPAS_PID"
    axIso.Min = 0
    axIso.Max = 10

    'Temperature axis
    Set axTemp = plotter.YAxes.Add()
    axTemp.Title = "EPAS_Temperature"
    axTemp.Min = -40
    axTemp.Max = 120

    'Voltage
    Set ch = plotter.Channels(1)
    Set ch.Signal = Signals("EMduleInCirct_U_Actl3")
    Set ch.YAxis = axVolt

    Set ch = plotter.Channels.Add()
    Set ch.Signal = Signals("EMduleOutCirct_U_Actl3")
    Set ch.YAxis = axVolt

    Set ch = plotter.Channels.Add()
    Set ch.Signal = Signals("Cell_U_Actl3")
    Set ch.YAxis = axVolt

    'Current
    Set ch = plotter.Channels.Add()
    Set ch.Signal = Signals("EMduleInCirct_I_Actl3")
    Set ch.YAxis = axCurr

    Set ch = plotter.Channels.Add()
    Set ch.Signal = Signals("EMduleOutCirct_LActl3")
    Set ch.YAxis = axCurr

    'Mode / Status
    Set ch = plotter.Channels.Add()
    Set ch.Signal = Signals("EMdule_D_Stat3")
    Set ch.YAxis = axMode

    Set ch = plotter.Channels.Add()
    Set ch.Signal = Signals("EMduleMde_D_Rq3")
    Set ch.YAxis = axMode

    'Isolation
    Set ch = plotter.Channels.Add()
    Set ch.Signal = Signals("IsolSwtch_B_Cmd3")
    Set ch.YAxis = axIso

    Set ch = plotter.Channels.Add()
    Set ch.Signal = Signals("IsolSwtch_BStat3")
    Set ch.YAxis = axIso

    'Temperature
    Set ch = plotter.Channels.Add()
    Set ch.Signal = Signals("FET_Te_Act3")
    Set ch.YAxis = axTemp

    Set ch = plotter.Channels.Add()
    Set ch.Signal = Signals("Cell_Te_Actl3")
    Set ch.YAxis = axTemp

End Sub