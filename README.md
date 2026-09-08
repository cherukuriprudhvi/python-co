

Option Explicit

Sub MAP_EPAS_PLOT2_CORRECT()

    Dim doc, plotter
    Dim axVolt, axCurr, axMode, axPID, axTemp

    Set doc = Documents.Item("Plot2.plt")
    Set plotter = doc.ActiveWindow.Object

    Set axVolt = plotter.YAxes(1)
    Set axCurr = plotter.YAxes(2)
    Set axMode = plotter.YAxes(3)
    Set axPID  = plotter.YAxes(4)
    Set axTemp = plotter.YAxes(5)

    'Voltage = Channels 2,3,4
    Set plotter.Channels(2).YAxis = axVolt
    Set plotter.Channels(3).YAxis = axVolt
    Set plotter.Channels(4).YAxis = axVolt

    'Current = Channels 5,6
    Set plotter.Channels(5).YAxis = axCurr
    Set plotter.Channels(6).YAxis = axCurr

    'SSR Mode = Channels 7,8
    Set plotter.Channels(7).YAxis = axMode
    Set plotter.Channels(8).YAxis = axMode

    'PID = Channels 9,10
    Set plotter.Channels(9).YAxis = axPID
    Set plotter.Channels(10).YAxis = axPID

    'Temperature = Channels 11,12
    Set plotter.Channels(11).YAxis = axTemp
    Set plotter.Channels(12).YAxis = axTemp

End Sub