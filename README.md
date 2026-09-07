

Option Explicit

Sub TEST_PLOT13_ONLY()

    Dim doc, plotter

    Set doc = Documents.Item("Plot13.plt")
    Set plotter = doc.ActiveWindow.Object

    plotter.YAxes(1).Min = -10

End Sub