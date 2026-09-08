

Option Explicit

Sub SETUP_EPAS_PLOT1()

    Dim doc, plotter
    Dim axVolt, axCurr, axMode, axPID, axTemp
    Dim ch

    Set doc = Documents.Item("Plot1.plt")
    Set plotter = doc.ActiveWindow.Object

    '5 Y-axis group names
    Set axVolt = plotter.YAxes(1)
    axVolt.Title = "EPAS_Volt"

    Set axCurr = plotter.YAxes(2)
    axCurr.Title = "EPAS_Current"

    Set axMode = plotter.YAxes(3)
    axMode.Title = "EPAS_SSR_Mode"

    Set axPID = plotter.YAxes(4)
    axPID.Title = "EPAS_PID"

    Set axTemp = plotter.YAxes(5)
    axTemp.Title = "EPAS_Temperature"

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

    'SSR / Mode
    Set ch = plotter.Channels(6)
    Set ch.Signal = Signals("EMdule_D_Stat3")
    Set ch.YAxis = axMode

    Set ch = plotter.Channels(7)
    Set ch.Signal = Signals("EMduleMde_D_Rq3")
    Set ch.YAxis = axMode

    'PID
    Set ch = plotter.Channels(8)
    Set ch.Signal = Signals("IsolSwtch_B_Cmd3")
    Set ch.YAxis = axPID

    Set ch = plotter.Channels(9)
    Set ch.Signal = Signals("IsolSwtch _B_Stat3")
    Set ch.YAxis = axPID

    'Temperature
    Set ch = plotter.Channels(10)
    Set ch.Signal = Signals("FET_Te_Act3")
    Set ch.YAxis = axTemp

    Set ch = plotter.Channels(11)
    Set ch.Signal = Signals("Cell_Te_Actl3")
    Set ch.YAxis = axTemp

End Sub