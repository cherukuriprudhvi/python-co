
Option Explicit

Sub SETUP_EPAS_FINAL()

    Dim doc, plotter
    Dim axVolt, axCurr, axMode, axIso, axTemp
    Dim ch

    'Use existing Plot13 only
    Set doc = Documents.Item("Plot13.plt")
    Set plotter = doc.ActiveWindow.Object

    '5 existing Y-axes
    Set axVolt = plotter.YAxes(1)
    axVolt.Title = "EPAS_Volt"
    axVolt.Min = 0
    axVolt.Max = 20

    Set axCurr = plotter.YAxes(2)
    axCurr.Title = "EPAS_Current"
    axCurr.Min = -50
    axCurr.Max = 50

    Set axMode = plotter.YAxes(3)
    axMode.Title = "EPAS_SSR_Mode"
    axMode.Min = 0
    axMode.Max = 10

    Set axIso = plotter.YAxes(4)
    axIso.Title = "EPAS_PID"
    axIso.Min = 0
    axIso.Max = 10

    Set axTemp = plotter.YAxes(5)
    axTemp.Title = "EPAS_Temperature"
    axTemp.Min = -40
    axTemp.Max = 120

    'Voltage
    Set ch = plotter.Channels(1)
    Set ch.Signal = Signals("EMduleInCirct_U_Actl3")
    Set ch.YAxis = axVolt

    Set ch = plotter.Channels(2)
    Set ch.Signal = Signals("EMduleOutCirct_U_Actl3")
    Set ch.YAxis = axVolt

    Set ch = plotter.Channels(3)
    Set ch.Signal = Signals("Cell_U_Actl3")
    Set ch.YAxis = axVolt

    'Current
    Set ch = plotter.Channels(4)
    Set ch.Signal = Signals("EMduleInCirct_I_Actl3")
    Set ch.YAxis = axCurr

    Set ch = plotter.Channels(5)
    Set ch.Signal = Signals("EMduleOutCirct_I_Actl3")
    Set ch.YAxis = axCurr

    'Mode / Status
    Set ch = plotter.Channels(6)
    Set ch.Signal = Signals("EMdule_D_Stat3")
    Set ch.YAxis = axMode

    Set ch = plotter.Channels(7)
    Set ch.Signal = Signals("EMduleMde_D_Rq3")
    Set ch.YAxis = axMode

    'Isolation
    Set ch = plotter.Channels(8)
    Set ch.Signal = Signals("IsolSwtch_B_Cmd3")
    Set ch.YAxis = axIso

    Set ch = plotter.Channels(9)
    Set ch.Signal = Signals("IsolSwtch_B_Stat3")
    Set ch.YAxis = axIso

    'Temperature
    Set ch = plotter.Channels(10)
    Set ch.Signal = Signals("FET_Te_Actl3")
    Set ch.YAxis = axTemp

    Set ch = plotter.Channels(11)
    Set ch.Signal = Signals("Cell_Te_Actl3")
    Set ch.YAxis = axTemp

End Sub
