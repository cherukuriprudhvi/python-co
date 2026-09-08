
Option Explicit

Sub BUILD_EBB_PLOT2()

    Dim doc, plotter
    Dim axVolt, axCurr, axMode, axPID, axTemp
    Dim ch

    Set doc = Documents.Item("Plot2.plt")
    Set plotter = doc.ActiveWindow.Object

    '========================
    ' 5 Y-AXES
    '========================

    Set axVolt = plotter.YAxes(1)
    axVolt.Title = "EBB_Voltage"
    axVolt.IsTitleVisible = True

    Set axCurr = plotter.YAxes.Add()
    axCurr.Title = "EBB_Current"
    axCurr.IsTitleVisible = True

    Set axMode = plotter.YAxes.Add()
    axMode.Title = "EBB_SSR_Mode"
    axMode.IsTitleVisible = True

    Set axPID = plotter.YAxes.Add()
    axPID.Title = "EBB_PID"
    axPID.IsTitleVisible = True

    Set axTemp = plotter.YAxes.Add()
    axTemp.Title = "EBB_Temperature"
    axTemp.IsTitleVisible = True


    '========================
    ' VOLTAGE - 3 SIGNALS
    '========================

    Set ch = plotter.Channels(1)
    Set ch.Signal = FindDBC3Signal("EMduleInCirct_U_Actl")
    Set ch.YAxis = axVolt
    ch.Title = "EMduleInCirct_U_Actl"

    AddEBBSignal plotter, "EMduleOutCirct_U_Actl", axVolt
    AddEBBSignal plotter, "CelLU_Actl", axVolt


    '========================
    ' CURRENT - 2 SIGNALS
    '========================

    AddEBBSignal plotter, "EMduleInCirc_I_Actl", axCurr
    AddEBBSignal plotter, "EMduleOutCirct_I_Actl", axCurr


    '========================
    ' SSR MODE - 2 SIGNALS
    '========================

    AddEBBSignal plotter, "EMduleMde_D_Rq", axMode
    AddEBBSignal plotter, "EMdule_D_Stat", axMode


    '========================
    ' PID - 2 SIGNALS
    '========================

    AddEBBSignal plotter, "IsolSwtch_B_Cmd", axPID
    AddEBBSignal plotter, "IsolSwtch_B_Stat", axPID


    '========================
    ' TEMPERATURE / ESTIMATES - 4
    '========================

    AddEBBSignal plotter, "FET_Te_Actl", axTemp
    AddEBBSignal plotter, "Cell_Te_Act", axTemp
    AddEBBSignal plotter, "Cell_C_Est", axTemp
    AddEBBSignal plotter, "CellEsrST_R_Est", axTemp

End Sub


Sub AddEBBSignal(plotter, signalName, yAxis)

    Dim ch

    Set ch = plotter.Channels.Add()
    Set ch.Signal = FindDBC3Signal(signalName)
    Set ch.YAxis = yAxis
    ch.Title = signalName

End Sub


Function FindDBC3Signal(signalName)

    Dim sig

    For Each sig In Signals

        If sig.ShortName = signalName Then

            If Left(sig.Name, 6) = "DBC-3." Then
                Set FindDBC3Signal = sig
                Exit Function
            End If

        End If

    Next

    Err.Raise 1001, , "DBC-3 signal not found: " & signalName

End Function
