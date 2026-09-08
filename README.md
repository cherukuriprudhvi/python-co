

Option Explicit

Sub BUILD_48V_PLOT5()

    Dim doc, plotter
    Dim axVolt, axCurr, axMode, axTemp
    Dim ch

    Set doc = Documents.Item("Plot5.plt")
    Set plotter = doc.ActiveWindow.Object

    '========================
    ' 4 Y-AXES
    '========================

    Set axVolt = plotter.YAxes(1)
    axVolt.Title = "48V_Voltage"
    axVolt.IsTitleVisible = True

    Set axCurr = plotter.YAxes.Add()
    axCurr.Title = "48v_Current"
    axCurr.IsTitleVisible = True

    Set axMode = plotter.YAxes.Add()
    axMode.Title = "48v_SSR_Mode"
    axMode.IsTitleVisible = True

    Set axTemp = plotter.YAxes.Add()
    axTemp.Title = "48v_Temperature"
    axTemp.IsTitleVisible = True


    '========================
    ' VOLTAGE - 3
    '========================

    Set ch = plotter.Channels(1)
    Set ch.Signal = FindDBC5Signal("UCapMduleAux_U_Act/")
    Set ch.YAxis = axVolt
    ch.Title = "UCapMduleAux_U_Act/"

    Add48VSignal plotter, "UCapMdule_U_Actl", axVolt
    Add48VSignal plotter, "Cell_U_Actl3", axVolt


    '========================
    ' CURRENT - 2
    '========================

    Add48VSignal plotter, "UCapMdule_I_Acti", axCurr

    'REPLACE THIS WITH EXACT FULL DBC SIGNAL NAME
    Add48VSignal plotter, "UCapMduleAux_I_Actl_[a...", axCurr


    '========================
    ' SSR MODE - 2
    '========================

    Add48VSignal plotter, "UCapMdule_D_Stat", axMode
    Add48VSignal plotter, "UCapMduleMde_D_Rq", axMode


    '========================
    ' TEMPERATURE / ESTIMATES - 4
    '========================

    Add48VSignal plotter, "FET_Te_Actl3", axTemp
    Add48VSignal plotter, "Cell_Te_Actl3", axTemp
    Add48VSignal plotter, "Cel_C_Est3", axTemp
    Add48VSignal plotter, "CelEsrST_R_Est3", axTemp

End Sub


Sub Add48VSignal(plotter, signalName, yAxis)

    Dim ch

    Set ch = plotter.Channels.Add()
    Set ch.Signal = FindDBC5Signal(signalName)
    Set ch.YAxis = yAxis
    ch.Title = signalName

End Sub


Function FindDBC5Signal(signalName)

    Dim sig

    For Each sig In Signals

        If sig.ShortName = signalName Then
            If Left(sig.Name, 6) = "DBC-5." Then
                Set FindDBC5Signal = sig
                Exit Function
            End If
        End If

    Next

    Err.Raise 1001, , "DBC-5 signal not found: " & signalName

End Function