
Option Explicit

Sub ADD_EPAS_CAN3_TO_CAN6()

    Dim doc, plotter
    Dim axVolt, axCurr, axMode, axIso, axTemp

    Set doc = Documents.Item("Plot11.plt")
    Set plotter = doc.ActiveWindow.Object

    'Use the existing 5 axes
    Set axVolt = plotter.YAxes(1)
    Set axCurr = plotter.YAxes(2)
    Set axMode = plotter.YAxes(3)
    Set axIso  = plotter.YAxes(4)
    Set axTemp = plotter.YAxes(5)

    'CAN3
    AddOneCAN plotter, "DBC-3.", "CAN3", axVolt, axCurr, axMode, axIso, axTemp

    'CAN4
    AddOneCAN plotter, "DBC-4.", "CAN4", axVolt, axCurr, axMode, axIso, axTemp

    'CAN5
    AddOneCAN plotter, "DBC-5.", "CAN5", axVolt, axCurr, axMode, axIso, axTemp

    'CAN6
    AddOneCAN plotter, "DBC-6.", "CAN6", axVolt, axCurr, axMode, axIso, axTemp

End Sub


Sub AddOneCAN(plotter, dbPrefix, canName, axVolt, axCurr, axMode, axIso, axTemp)

    BindNew plotter, dbPrefix, "EMduleInCirct_U_Actl3",  axVolt, canName & "_InVolt"
    BindNew plotter, dbPrefix, "EMduleOutCirct_U_Actl3", axVolt, canName & "_OutVolt"
    BindNew plotter, dbPrefix, "Cell_U_Actl3",            axVolt, canName & "_CellVolt"

    BindNew plotter, dbPrefix, "EMduleInCirct_I_Actl3",  axCurr, canName & "_InCurrent"
    BindNew plotter, dbPrefix, "EMduleOutCirct_I_Actl3", axCurr, canName & "_OutCurrent"

    BindNew plotter, dbPrefix, "EMdule_D_Stat3",          axMode, canName & "_Status"
    BindNew plotter, dbPrefix, "EMduleMde_D_Rq3",        axMode, canName & "_ModeReq"

    BindNew plotter, dbPrefix, "IsolSwtch_B_Cmd3",       axIso, canName & "_IsolCmd"
    BindNew plotter, dbPrefix, "IsolSwtch_B_Stat3",      axIso, canName & "_IsolStat"

    BindNew plotter, dbPrefix, "FET_Te_Actl3",           axTemp, canName & "_FETTemp"
    BindNew plotter, dbPrefix, "Cell_Te_Actl3",          axTemp, canName & "_CellTemp"

End Sub


Sub BindNew(plotter, dbPrefix, shortName, yAxis, title)

    Dim ch, sig

    Set sig = FindSignal(dbPrefix, shortName)

    Set ch = plotter.Channels.Add()
    Set ch.Signal = sig
    Set ch.YAxis = yAxis
    ch.Title = title

End Sub


Function FindSignal(dbPrefix, shortName)

    Dim sig

    For Each sig In Signals

        If sig.ShortName = shortName Then

            If Left(sig.Name, Len(dbPrefix)) = dbPrefix Then
                Set FindSignal = sig
                Exit Function
            End If

        End If

    Next

    Err.Raise 1001, , "Signal not found: " & dbPrefix & shortName

End Function