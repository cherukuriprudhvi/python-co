

Option Explicit

Sub RENAME_EPAS_AXES()

    Dim doc, plotter

    Set doc = Documents.Item("Plot2.plt")
    Set plotter = doc.ActiveWindow.Object

    plotter.YAxes(1).Title = "EPAS_Volt"
    plotter.YAxes(1).IsTitleVisible = True

    plotter.YAxes(2).Title = "EPAS_Current"
    plotter.YAxes(2).IsTitleVisible = True

    plotter.YAxes(3).Title = "EPAS_SSR_Mode"
    plotter.YAxes(3).IsTitleVisible = True

    plotter.YAxes(4).Title = "EPAS_PID"
    plotter.YAxes(4).IsTitleVisible = True

    plotter.YAxes(5).Title = "EPAS_Temperature"
    plotter.YAxes(5).IsTitleVisible = True

End Sub