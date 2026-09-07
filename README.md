

Option Explicit

Sub FIND_SIGNAL_SOURCES()

    Dim sig, src

    For Each sig In Signals

        If sig.Name = "EMduleInCirct_U_Actl3" Then

            Set src = sig.Source

            If Not (src Is Nothing) Then
                PrintToOutputWindow sig.Name & "  -->  " & src.QualifiedName
            End If

        End If

    Next

End Sub