

Option Explicit

Sub Update_Plot11_Ranges()

    Dim doc, plotter

    Set doc = Documents.Item("Plot11.plt")
    Set plotter = doc.ActiveWindow.Object

    plotter.YAxes(1).Min = 0
    plotter.YAxes(1).Max = 20

    plotter.YAxes(2).Min = -50
    plotter.YAxes(2).Max = 50

    plotter.YAxes(3).Min = 0
    plotter.YAxes(3).Max = 10

    plotter.YAxes(4).Min = 0
    plotter.YAxes(4).Max = 10

    plotter.YAxes(5).Min = -40
    plotter.YAxes(5).Max = 120

End Sub