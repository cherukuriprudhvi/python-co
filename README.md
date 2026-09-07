
Sub CreateEPASPlots()
    Dim doc, plotWin, plotterObj, mainGraph
    Dim i
    
    ' 1. Wipe out any messy open plots to clear your screen space
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
    
    ' Correct nested object mapping: Document > ActiveSheet > Graphs(0)
    Set plotterObj = doc.ActiveSheet
    Set mainGraph = plotterObj.Graphs(0)
    
    ' Add variables to the graph channel list using your actual _Actl2 names
    mainGraph.Channels.Add "EPAS_Volt.EMduleInCrct_U_Actl2"
    mainGraph.Channels.Add "EPAS_Volt.EMduleOutCrct_U_Actl2"
    mainGraph.Channels.Add "EPAS_Volt.Cell_U_Actl2"
    mainGraph.Channels.Add "EPAS_Current.EMduleInCrct_I_Actl2"
    mainGraph.Channels.Add "EPAS_Current.EMduleOutCrct_I_Actl2"
    
    ' Set Y-Axis for your voltage/current bounds
    mainGraph.YAxes(0).Autoscale = False
    mainGraph.YAxes(0).Min = 0.0
    mainGraph.YAxes(0).Max = 16.0

    ' ----------------------------------------------------
    ' PLOT 2: THERMAL (Temperatures)
    ' ----------------------------------------------------
    Set doc = Documents.Add(peDocumentKindPlotter)
    Set plotWin = doc.ActiveWindow
    plotWin.Left = 655
    plotWin.Top = 0
    plotWin.Width = 650
    plotWin.Height = 450
    doc.Name = "EPAS_Thermal"
    
    Set plotterObj = doc.ActiveSheet
    Set mainGraph = plotterObj.Graphs(0)
    mainGraph.Channels.Add "EPAS_Temperature.FET_Te_Actl2"
    mainGraph.Channels.Add "EPAS_Temperature.Cel_Te_Actl2"
    
    ' Set Y-Axis for temperatures (degC)
    mainGraph.YAxes(0).Autoscale = False
    mainGraph.YAxes(0).Min = 15.0
    mainGraph.YAxes(0).Max = 100.0

    ' ----------------------------------------------------
    ' GLOBAL ALIGNMENT
    ' ----------------------------------------------------
    ' Synchronize horizontal scrolling timelines automatically
    Commands.Execute "Plotter:EnableXAxisSync"

    MsgBox "EPAS System Plots generated cleanly!", vbInformation, "Automation Done"
End Sub

