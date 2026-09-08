

Option Explicit

Sub BUILD_EPAS_PLOT1_FINAL()

    Dim doc, plotter
    Dim axVolt, axCurr, axMode, axPID, axTemp
    Dim ch

    Set doc = Documents.Item("Plot1_Final.plt")
    Set plotter = doc.ActiveWindow.Object

    '================================
    ' 5 Y-AXES
    '================================

    Set axVolt = plotter.YAxes(1)
    axVolt.Title = "EPAS_Volt"
    axVolt.IsTitleVisible = True

    Set axCurr = plotter.YAxes.Add()
    axCurr.Title = "EPAS_Current"
    axCurr.IsTitleVisible = True

    Set axMode = plotter.YAxes.Add()
    axMode.Title = "EPAS_SSR_Mode"
    axMode.IsTitleVisible = True

    Set axPID = plotter.YAxes.Add()
    axPID.Title = "EPAS_PID"
    axPID.IsTitleVisible = True

    Set axTemp = plotter.YAxes.Add()
    axTemp.Title = "EPAS_Temperature"
    axTemp.IsTitleVisible = True


    '================================
    ' VOLTAGE
    '================================

    Set ch = plotter.Channels(1)
    Set ch.Signal = FindDBC1Signal("EMduleInCirct_U_Actl3")
    Set ch.YAxis = axVolt
    ch.Title = "EMduleInCirct_U_Actl3"

    AddEPASSignal plotter, "EMduleOutCirct_U_Actl3", axVolt
    AddEPASSignal plotter, "Cell_U_Actl3", axVolt


    '================================
    ' CURRENT
    '================================

    AddEPASSignal plotter, "EMduleInCirct_I_Actl3", axCurr
    AddEPASSignal plotter, "EMduleOutCirct_I_Actl3", axCurr


    '================================
    ' SSR MODE
    '================================

    AddEPASSignal plotter, "EMdule_D_Stat3", axMode
    AddEPASSignal plotter, "EMduleMde_D_Rq3", axMode


    '================================
    ' PID
    '================================

    AddEPASSignal plotter, "IsolSwtch_B_Cmd3", axPID
    AddEPASSignal plotter, "IsolSwtch _B_Stat3", axPID


    '================================
    ' TEMPERATURE
    '================================

    AddEPASSignal plotter, "FET_Te_Act3", axTemp
    AddEPASSignal plotter, "Cell_Te_Actl3", axTemp

End Sub


Sub AddEPASSignal(plotter, signalName, yAxis)

    Dim ch

    Set ch = plotter.Channels.Add()
    Set ch.Signal = FindDBC1Signal(signalName)
    Set ch.YAxis = yAxis
    ch.Title = signalName

End Sub


Function FindDBC1Signal(signalName)

    Dim sig

    For Each sig In Signals

        If sig.ShortName = signalName Then

            If Left(sig.Name, 6) = "DBC-1." Then
                Set FindDBC1Signal = sig
                Exit Function
            End If

        End If

    Next

    Err.Raise 1001, , "DBC-1 signal not found: " & signalName

End Function