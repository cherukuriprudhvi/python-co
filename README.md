

Sub SignalExample()

    Dim doc, plotter, ch

    Set doc = ActiveDocument
    Set plotter = doc.ActiveWindow.Object

    Set ch = plotter.Channels.Add
    Set ch.Signal = Signals("EMduleInCrct_U_Actl3")

End Sub