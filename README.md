

Option Explicit

Sub GROUP_EMB_PLOT4()

    Dim doc, plotter
    Dim axVolt, axCurr, axMode, axPID, axTemp

    Set doc = Documents.Item("Plot4.plt")
    Set plotter = doc.ActiveWindow.Object

    '========================
    ' 5 Y-AXES
    '========================

    Set axVolt = plotter.YAxes(1)
    axVolt.Title = "EMB_Voltage"
    axVolt.IsTitleVisible = True

    Set axCurr = plotter.YAxes.Add()
    axCurr.Title = "EMB_Current"
    axCurr.IsTitleVisible = True

    Set axMode = plotter.YAxes.Add()
    axMode.Title = "EMB_SSR_Mode"
    axMode.IsTitleVisible = True

    Set axPID = plotter.YAxes.Add()
    axPID.Title = "EMB_PID"
    axPID.IsTitleVisible = True

    Set axTemp = plotter.YAxes.Add()
    axTemp.Title = "EMB_Temperature"
    axTemp.IsTitleVisible = True


    '========================
    ' MAP EXISTING CHANNELS
    '========================

    'Voltage = first 3 signals
    Set plotter.Channels(1).YAxis = axVolt
    Set plotter.Channels(2).YAxis = axVolt
    Set plotter.Channels(3).YAxis = axVolt

    'Current = next 3 signals
    Set plotter.Channels(4).YAxis = axCurr
    Set plotter.Channels(5).YAxis = axCurr
    Set plotter.Channels(6).YAxis = axCurr

    'SSR Mode = next 2 signals
    Set plotter.Channels(7).YAxis = axMode
    Set plotter.Channels(8).YAxis = axMode

    'PID = next 2 signals
    Set plotter.Channels(9).YAxis = axPID
    Set plotter.Channels(10).YAxis = axPID

    'Temperature / threshold = last 3
    Set plotter.Channels(11).YAxis = axTemp
    Set plotter.Channels(12).YAxis = axTemp
    Set plotter.Channels(13).YAxis = axTemp

End Sub