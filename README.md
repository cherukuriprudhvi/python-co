

Option Explicit

Sub TEST_EXISTING_PLOT_ONLY()

    Dim doc, plotter

    Set doc = Documents.Item("Plot12.plt")
    Set plotter = doc.ActiveWindow.Object

    plotter.Title = "EPAS TEST"

End Sub