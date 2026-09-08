

Option Explicit

Sub BUILD_EPAS_PLOT2()

    Dim doc, plotter
    Dim axVolt, axCurr, axMode, axPID, axTemp
    Dim ch

    Set doc = Documents.Item("Plot2.plt")
    Set plotter = doc.ActiveWindow.Object

    '========================
    ' 5 Y-AXES
    '========================

    Set axVolt = plotter.YAxes(1)
    axVolt.Title = "EPAS_Volt"

    Set axCurr = plotter.YAxes.Add()
    axCurr.Title = "EPAS_Current"

    Set axMode = plotter.YAxes.Add()
    axMode.Title = "EPAS_SSR_Mode"

    Set axPID = plotter.YAxes.Add()
    axPID.Title = "EPAS_PID"

    Set axTemp = plotter.YAxes.Add()
    axTemp.Title = "EPAS_Temperature"


    '========================
    ' VOLTAGE - 3 SIGNALS
    '========================

    Set ch = plotter.Channels(1)
    Set ch.Signal = FindDBC2Signal("EMduleInCirct_U_Actl3")
    Set ch.YAxis = axVolt
    ch.Title = "EMduleInCirct_U_Actl3"

    AddSignal plotter, "EMduleOutCirct_U_Actl3", axVolt
    AddSignal plotter, "Cell_U_Actl3", axVolt


    '========================
    ' CURRENT - 2 SIGNALS
    '========================

    AddSignal plotter, "EMduleInCirct_I_Actl3", axCurr
    AddSignal plotter, "EMduleOutCirct_I_Actl3", axCurr


    '========================
    ' SSR MODE - 2 SIGNALS
    '========================

    AddSignal plotter, "EMdule_D_Stat3", axMode
    AddSignal plotter, "EMduleMde_D_Rq3", axMode


    '========================
    ' PID - 2 SIGNALS
    '========================

    AddSignal plotter, "IsolSwtch_B_Cmd3", axPID
    AddSignal plotter, "IsolSwtch _B_Stat3", axPID


    '========================
    ' TEMPERATURE - 2 SIGNALS
    '========================

    AddSignal plotter, "FET_Te_Act3", axTemp
    AddSignal plotter, "Cell_Te_Actl3", axTemp

End Sub


Sub AddSignal(plotter, signalName, axis)

    Dim ch

    Set ch = plotter.Channels.Add()
    Set ch.Signal = FindDBC2Signal(signalName)
    Set ch.YAxis = axis
    ch.Title = signalName

End Sub


Function FindDBC2Signal(signalName)

    Dim sig

    For Each sig In Signals

        If sig.ShortName = signalName Then
            If Left(sig.Name, 6) = "DBC-2." Then
                Set FindDBC2Signal = sig
                Exit Function
            End If
        End If

    Next

    Err.Raise 1001, , "DBC-2 signal not found: " & signalName

End Function
