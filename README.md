

Option Explicit

Sub GROUP_EPAS_PLOT2()

    Dim doc, plotter
    Dim axVolt, axCurr, axMode, axPID, axTemp

    Set doc = Documents.Item("Plot2.plt")
    Set plotter = doc.ActiveWindow.Object

    'Use first axis + create 4 more
    Set axVolt = plotter.YAxes(1)
    axVolt.Title = "EPAS_Volt"

    Set axCurr = plotter.YAxes.Add()
    axCurr.Title = "EPAS_Current"

    Set axMode = plotter.YAxes.Add()
    axMode.Title = "EPAS_SSR_Mode"

    Set axPID = plotter.YAxes.Add()
    axPID.Title = "EPAS_PID"

    Set axTemp = plotter.YAxes.Add()
    axTemp.Title = "EPAS_Temperature"

    'Channels 1-3 = Voltage
    Set plotter.Channels(1).YAxis = axVolt
    Set plotter.Channels(2).YAxis = axVolt
    Set plotter.Channels(3).YAxis = axVolt

    'Channels 4-5 = Current
    Set plotter.Channels(4).YAxis = axCurr
    Set plotter.Channels(5).YAxis = axCurr

    'Channels 6-7 = SSR Mode
    Set plotter.Channels(6).YAxis = axMode
    Set plotter.Channels(7).YAxis = axMode

    'Channels 8-9 = PID
    Set plotter.Channels(8).YAxis = axPID
    Set plotter.Channels(9).YAxis = axPID

    'Channels 10-11 = Temperature
    Set plotter.Channels(10).YAxis = axTemp
    Set plotter.Channels(11).YAxis = axTemp

End Sub