

Option Explicit

Sub MAP_EPAS_PLOT2()

    Dim doc, plotter
    Dim axVolt, axCurr, axMode, axPID, axTemp

    Set doc = Documents.Item("Plot2.plt")
    Set plotter = doc.ActiveWindow.Object

    Set axVolt = plotter.YAxes(1)
    Set axCurr = plotter.YAxes(2)
    Set axMode = plotter.YAxes(3)
    Set axPID  = plotter.YAxes(4)
    Set axTemp = plotter.YAxes(5)

    'Voltage
    Set plotter.Channels(1).YAxis = axVolt
    Set plotter.Channels(2).YAxis = axVolt
    Set plotter.Channels(3).YAxis = axVolt

    'Current
    Set plotter.Channels(4).YAxis = axCurr
    Set plotter.Channels(5).YAxis = axCurr

    'SSR Mode
    Set plotter.Channels(6).YAxis = axMode
    Set plotter.Channels(7).YAxis = axMode

    'PID
    Set plotter.Channels(8).YAxis = axPID
    Set plotter.Channels(9).YAxis = axPID

    'Temperature
    Set plotter.Channels(10).YAxis = axTemp
    Set plotter.Channels(11).YAxis = axTemp

End Sub