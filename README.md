

Option Explicit

Sub SignalExample()

    Dim doc, plotter, ch
    Dim sigs, i

    Set doc = Documents.Item("Plot11.plt")
    Set plotter = doc.ActiveWindow.Object

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
        Set ch = plotter.Channels.Add
        Set ch.Signal = Signals(sigs(i))
    Next

End Sub