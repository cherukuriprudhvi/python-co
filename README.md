

Sub CreateEPASPlots()
    Dim doc, plotWin, plotterObj
    Dim i
    
    ' 1. Wipe out any messy open plots to clear your screen space
    ' (Using the global Documents collection directly)
    For i = Documents.Count To 1 Step -1
        If InStr(Documents(i).Name, ".plt") > 0 Then
            Documents(i).Close peChangesDiscard
        End If
    Next

    ' ----------------------------------------------------
    ' PLOT 1: ELECTRICAL (Voltages & Currents)
    ' ----------------------------------------------------
    Set doc = Documents.Add(peDocumentKindPlotter)
    Set plotWin = doc.ActiveWindow
    plotWin.Left = 0
    plotWin.Top = 0
    plotWin.Width = 650
    plotWin.Height = 450
    doc.Name = "EPAS_Electrical"
    
    Set plotterObj = plotWin.Object
    plotterObj.Channels.AddVariable "EPAS_Volt.EMduleInCrct_U_Act3"
    plotterObj.Channels.AddVariable "EPAS_Volt.EMduleOutCrct_U_Act3"
    plotterObj.Channels.AddVariable "EPAS_Volt.Cell_U_Act3"
    plotterObj.Channels.AddVariable "EPAS_Current.EMduleInCrct_I_Act3"
    plotterObj.Channels.AddVariable "EPAS_Current.EMduleOutCrct_I_Act3"
    
    ' Set Y-Axis for 12V electrical bounds
    plotterObj.YAxes(0).Autoscale = False
    plotterObj.YAxes(0).Min = 0.0
    plotterObj.YAxes(0).Max = 16.0

    ' ----------------------------------------------------
    ' PLOT 2: CONTROL STATES (Modes & Switches)
    ' ----------------------------------------------------
    Set doc = Documents.Add(peDocumentKindPlotter)
    Set plotWin = doc.ActiveWindow
    plotWin.Left = 655
    plotWin.Top = 0
    plotWin.Width = 650
    plotWin.Height = 450
    doc.Name = "EPAS_Control_States"
    
    Set plotterObj = plotWin.Object
    plotterObj.Channels.AddVariable "EPAS_SSR_Mode.EMdule_O_Stat3"
    plotterObj.Channels.AddVariable "EPAS_SSR_Mode.EMduleMde_D_Rq3"
    plotterObj.Channels.AddVariable "EPAS_PID.IsolSwtch_B_Cmd3"
    plotterObj.Channels.AddVariable "EPAS_PID.IsolSwtch_B_Stat3"
    
    ' Let state enums auto-scale their discrete steps
    plotterObj.YAxes(0).Autoscale = True

    ' ----------------------------------------------------
    ' PLOT 3: THERMAL (Temperatures)
    ' ----------------------------------------------------
    Set doc = Documents.Add(peDocumentKindPlotter)
    Set plotWin = doc.ActiveWindow
    plotWin.Left = 0
    plotWin.Top = 455
    plotWin.Width = 1305
    plotWin.Height = 450
    doc.Name = "EPAS_Thermal"
    
    Set plotterObj = plotWin.Object
    plotterObj.Channels.AddVariable "EPAS_Temperature.FET_Te_Act3"
    plotterObj.Channels.AddVariable "EPAS_Temperature.Cel_Te_Act3"
    
    ' Set Y-Axis for standard operating temperatures (degC)
    plotterObj.YAxes(0).Autoscale = False
    plotterObj.YAxes(0).Min = 15.0
    plotterObj.YAxes(0).Max = 100.0

    ' ----------------------------------------------------
    ' GLOBAL ALIGNMENT
    ' ----------------------------------------------------
    ' Sync the timelines horizontally so scrolling matches perfectly
    Commands.Execute "Plotter:EnableXAxisSync"

    MsgBox "EPAS System Plots generated cleanly!", vbInformation, "Automation Done"
End Sub
