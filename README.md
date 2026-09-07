

Option Explicit

Sub SHOW_CAN1_ONLY()

    Dim doc, plotter, ch, i

    Set doc = Documents.Item("Plot11.plt")
    Set plotter = doc.ActiveWindow.Object

    'Hide every channel
    For Each ch In plotter.Channels
        ch.IsVisible = False
    Next

    'Show CAN1 = channels 1 through 11
    For i = 1 To 11
        plotter.Channels(i).IsVisible = True
    Next

End Sub