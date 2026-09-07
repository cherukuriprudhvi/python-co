


Option Explicit

Sub Create_CAN1_EPAS_Plot()

    Dim doc, plotter
    Dim ch, yAxis
    Dim sigs, i

    'Create a new Plotter
    Set doc = Documents.Add(peDocumentKindPlotter)
    Set plotter = doc.ActiveWindow.Object

    'CAN1 EPAS signals
    sigs = Array( _
        "EMduleInCirct_U_Actl3", _
        "EMduleOutCirct_U_Actl3", _
        "Cell_U_Actl3", _
        "EMduleInCirct_I_Actl3", _
        "EMduleOutCirct_LActl3", _
        "EMdule_D_Stat3", _
        "EMduleMde_D_Rq3", _
        "IsolSwtch_B_Cmd3", _
        "IsolSwtch_BStat3", _
        "FET_Te_Act3", _
        "Cell_Te_Actl3" _
    )

    For i = 0 To UBound(sigs)

        'Create separate Y-axis
        Set yAxis = plotter.YAxes.Add()

        'Create channel
        Set ch = plotter.Channels.Add()

        'Bind signal
        Set ch.Signal = Signals(sigs(i))

        'Put this signal on its own Y-axis
        Set ch.YAxis = yAxis

        'Show signal name
        ch.Title = sigs(i)

    Next

End Sub