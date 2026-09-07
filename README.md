

Option Explicit

Sub FIND_ALL_SIGNAL_SOURCES()

    Dim sig, src

    For Each sig In Signals

        Set src = sig.Source

        If Not (src Is Nothing) Then
            PrintToOutputWindow sig.Name & " --> " & src.QualifiedName
        Else
            PrintToOutputWindow sig.Name & " --> NO SOURCE"
        End If

    Next

End Sub