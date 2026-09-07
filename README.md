Option Explicit

Sub BUILD_EPAS_CAN1_CAN2()

    Dim doc, plotter
    Dim axVolt, axCurr, axMode, axIso, axTemp

    Set doc = Documents.Item("Plot11.plt")
    Set plotter = doc.ActiveWindow.Object

    '==============================
    ' 5 Y-AXES
    '==============================

    'Reuse default Y-axis
    Set axVolt = plotter.YAxes(1)
    axVolt.Title = "EPAS_Voltage"
    axVolt.Min = 0
    axVolt.Max = 20

    Set axCurr = plotter.YAxes.Add()
    axCurr.Title = "EPAS_Current"
    axCurr.Min = -50
    axCurr.Max = 50

    Set axMode = plotter.YAxes.Add()
    axMode.Title = "EPAS_Mode"
    axMode.Min = 0
    axMode.Max = 10

    Set axIso = plotter.YAxes.Add()
    axIso.Title = "EPAS_Isolation"
    axIso.Min = 0
    axIso.Max = 10

    Set axTemp = plotter.YAxes.Add()
    axTemp.Title = "EPAS_Temperature"
    axTemp.Min = -40
    axTemp.Max = 120


    '==============================
    ' CAN1 / DBC-1
    '==============================

    BindFirst plotter, "DBC-1.", "EMduleInCirct_U_Actl3", axVolt, "CAN1_InVolt"

    BindNew plotter, "DBC-1.", "EMduleOutCirct_U_Actl3", axVolt, "CAN1_OutVolt"
    BindNew plotter, "DBC-1.", "Cell_U_Actl3", axVolt, "CAN1_CellVolt"

    BindNew plotter, "DBC-1.", "EMduleInCirct_I_Actl3", axCurr, "CAN1_InCurrent"
    BindNew plotter, "DBC-1.", "EMduleOutCirct_I_Actl3", axCurr, "CAN1_OutCurrent"

    BindNew plotter, "DBC-1.", "EMdule_D_Stat3", axMode, "CAN1_Status"
    BindNew plotter, "DBC-1.", "EMduleMde_D_Rq3", axMode, "CAN1_ModeReq"

    BindNew plotter, "DBC-1.", "IsolSwtch_B_Cmd3", axIso, "CAN1_IsolCmd"
    BindNew plotter, "DBC-1.", "IsolSwtch_B_Stat3", axIso, "CAN1_IsolStat"

    BindNew plotter, "DBC-1.", "FET_Te_Actl3", axTemp, "CAN1_FETTemp"
    BindNew plotter, "DBC-1.", "Cell_Te_Actl3", axTemp, "CAN1_CellTemp"


    '==============================
    ' CAN2 / DBC-2
    '==============================

    BindNew plotter, "DBC-2.", "EMduleInCirct_U_Actl3", axVolt, "CAN2_InVolt"
    BindNew plotter, "DBC-2.", "EMduleOutCirct_U_Actl3", axVolt, "CAN2_OutVolt"
    BindNew plotter, "DBC-2.", "Cell_U_Actl3", axVolt, "CAN2_CellVolt"

    BindNew plotter, "DBC-2.", "EMduleInCirct_I_Actl3", axCurr, "CAN2_InCurrent"
    BindNew plotter, "DBC-2.", "EMduleOutCirct_I_Actl3", axCurr, "CAN2_OutCurrent"

    BindNew plotter, "DBC-2.", "EMdule_D_Stat3", axMode, "CAN2_Status"
    BindNew plotter, "DBC-2.", "EMduleMde_D_Rq3", axMode, "CAN2_ModeReq"

    BindNew plotter, "DBC-2.", "IsolSwtch_B_Cmd3", axIso, "CAN2_IsolCmd"
    BindNew plotter, "DBC-2.", "IsolSwtch_B_Stat3", axIso, "CAN2_IsolStat"

    BindNew plotter, "DBC-2.", "FET_Te_Actl3", axTemp, "CAN2_FETTemp"
    BindNew plotter, "DBC-2.", "Cell_Te_Actl3", axTemp, "CAN2_CellTemp"

End Sub


'=================================================
' USE DEFAULT CHANNEL 1
'=================================================
Sub BindFirst(plotter, dbPrefix, shortName, yAxis, title)

    Dim ch, sig

    Set sig = FindSignal(dbPrefix, shortName)

    Set ch = plotter.Channels(1)
    Set ch.Signal = sig
    Set ch.YAxis = yAxis
    ch.Title = title

End Sub


'=================================================
' ADD NEXT CHANNEL
'=================================================
Sub BindNew(plotter, dbPrefix, shortName, yAxis, title)

    Dim ch, sig

    Set sig = FindSignal(dbPrefix, shortName)

    Set ch = plotter.Channels.Add()
    Set ch.Signal = sig
    Set ch.YAxis = yAxis
    ch.Title = title

End Sub


'=================================================
' FIND SIGNAL FROM CORRECT DBC
'=================================================
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