

Option Explicit

Sub TEST_ACTIVE_PLOT_ONLY()

    Dim doc, plotter

    Set doc = ActiveDocument

    If doc.Kind = peDocumentKindPlotter Then

        Set plotter = doc.ActiveWindow.Object
        plotter.Title = "EPAS TEST"

    End If

End Sub