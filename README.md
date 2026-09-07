

Option Explicit

Sub TEST_ACTIVE_PLOT_ONLY()

    Dim doc, plotter

    Set doc = ActiveDocument
    Set plotter = doc.ActiveWindow.Object

    plotter.Title = "EPAS TEST"

End Sub