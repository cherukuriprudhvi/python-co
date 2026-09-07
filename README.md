

Option Explicit

Sub TEST_ACTIVE_PLOT_ONLY()

    Dim doc, plotter

    Set doc = ActiveDocument
    Set plotter = doc.ActiveWindow.Object

    plotter.YAxes(1).Min = -10

End Sub