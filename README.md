
Option Explicit

Sub Update_Plot11()

    Dim doc, plotter
    Dim axVolt, axCurr, axMode, axIso, axTemp
    Dim ch

    'Use existing Plot11
    Set doc = Documents.Item("Plot11.plt")
    Set plotter = doc.ActiveWindow.Object

    'Existing 5 Y-axes
    Set axVolt = plotter.YAxes(1)
    Set axCurr = plotter.YAxes(2)
    Set axMode = plotter.YAxes(3)
    Set axIso = plotter.YAxes(4)
    Set axTemp = plotter.YAxes(5)

    'Axis titles and ranges
    axVolt.Title = "EPAS_Volt"
    axVolt.Min = 0
    axVolt.Max = 20

    axCurr.Title = "EPAS_Current"
    axCurr.Min = -50
    axCurr.Max = 50

    axMode.Title = "EPAS_SSR_Mode"
    axMode.Min = 0
    axMode.Max = 10

    axIso.Title = "EPAS_PID"
    axIso.Min = 0
    axIso.Max = 10

    axTemp.Title = "EPAS_Temperature"
    axTemp.Min = -40
    axTemp.Max = 120

    'Signal 1 - Voltage
    Set ch = plotter.Channels(1)
    Set ch.Signal = Signals("EMduleInCirct_U_Actl3")
    Set ch.YAxis = axVolt

    'Signal 2 - Voltage
    Set ch = plotter.Channels(2)
    Set ch.Signal = Signals("EMduleOutCirct_U_Actl3")
    Set ch.YAxis = axVolt

    'Signal 3 - Voltage
    Set ch = plotter.Channels(3)
    Set ch.Signal = Signals("Cell_U_Actl3")
    Set ch.YAxis = axVolt

    'Signal 4 - Current
    Set ch = plotter.Channels(4)
    Set ch.Signal = Signals("EMduleInCirct_I_Actl3")
    Set ch.YAxis = axCurr

    'Signal 5 - Current
    Set ch = plotter.Channels(5)
    Set ch.Signal = Signals("EMduleOutCirct_LActl3")
    Set ch.YAxis = axCurr

    'Signal 6 - Mode
    Set ch = plotter.Channels(6)
    Set ch.Signal = Signals("EMdule_D_Stat3")
    Set ch.YAxis = axMode

    'Signal 7 - Mode
    Set ch = plotter.Channels(7)
    Set ch.Signal = Signals("EMduleMde_D_Rq3")
    Set ch.YAxis = axMode

    'Signal 8 - Isolation
    Set ch = plotter.Channels(8)
    Set ch.Signal = Signals("IsolSwtch_B_Cmd3")
    Set ch.YAxis = axIso

    'Signal 9 - Isolation
    Set ch = plotter.Channels(9)
    Set ch.Signal = Signals("IsolSwtch_BStat3")
    Set ch.YAxis = axIso

    'Signal 10 - Temperature
    Set ch = plotter.Channels(10)
    Set ch.Signal = Signals("FET_Te_Act3")
    Set ch.YAxis = axTemp

    'Signal 11 - Temperature
    Set ch = plotter.Channels(11)
    Set ch.Signal = Signals("Cell_Te_Actl3")
    Set ch.YAxis = axTemp

End Sub
