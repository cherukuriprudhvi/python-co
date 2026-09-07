

Option Explicit

Sub FIND_EPAS_SIGNAL()

    Dim sig

    For Each sig In Signals

        If sig.ShortName = "EMduleInCirct_U_Actl3" Then
            PrintToOutputWindow sig.Name
        End If

    Next

End Sub