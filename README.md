


Sub CreateOptimizedBMSPlots()
    Dim app
    Dim doc, plotWin, plotterObj
    Dim i
    
    Set app = CreateObject("PCANExplorer.Application")
    
    ' 1. Clear out any old open Plot files to reclaim screen space
    For i = app.Documents.Count To 1 Step -1
        If InStr(app.Documents(i).Name, ".plt") > 0 Then
            app.Documents(i).Close peChangesDiscard
        End If
    Next

    ' ----------------------------------------------------
    ' PLOT 1: CELL VOLTAGES (High Resolution Zoom)
    ' ----------------------------------------------------
    Set doc = app.Documents.Add(peDocumentKindPlotter)
    Set plotWin = doc.ActiveWindow
    
    ' Position Window (Left, Top, Width, Height)
    plotWin.Left = 0
    plotWin.Top = 0
    plotWin.Width = 600
    plotWin.Height = 450
    doc.Name = "Cell_Voltages"
    
    Set plotterObj = plotWin.Object
    ' Add your specific BMS voltage signals here
    plotterObj.Channels.AddVariable "BMS_Pack.Cell_Max_Volt"
    plotterObj.Channels.AddVariable "BMS_Pack.Cell_Min_Volt"
    
    ' Enforce strict Y-Axis limits for BMS cells (3.0V to 4.2V)
    plotterObj.YAxes(0).Autoscale = False
    plotterObj.YAxes(0).Min = 3.0
    plotterObj.YAxes(0).Max = 4.2

    ' ----------------------------------------------------
    ' PLOT 2: THERMAL PROFILE (Slow Dynamics)
    ' ----------------------------------------------------
    Set doc = app.Documents.Add(peDocumentKindPlotter)
    Set plotWin = doc.ActiveWindow
    
    ' Place it right next to Plot 1
    plotWin.Left = 605
    plotWin.Top = 0
    plotWin.Width = 600
    plotWin.Height = 450
    doc.Name = "BMS_Temperatures"
    
    Set plotterObj = plotWin.Object
    plotterObj.Channels.AddVariable "BMS_Thermal.Module1_Temp"
    plotterObj.Channels.AddVariable "BMS_Thermal.Module2_Temp"
    
    ' Set temperature limits (e.g., 0°C to 80°C)
    plotterObj.YAxes(0).Autoscale = False
    plotterObj.YAxes(0).Min = 0.0
    plotterObj.YAxes(0).Max = 80.0

    ' ----------------------------------------------------
    ' PLOT 3: PACK POWER & CURRENT (Dynamic Control)
    ' ----------------------------------------------------
    Set doc = app.Documents.Add(peDocumentKindPlotter)
    Set plotWin = doc.ActiveWindow
    
    ' Place it below or on your second monitor screen space
    plotWin.Left = 0
    plotWin.Top = 455
    plotWin.Width = 1205
    plotWin.Height = 450
    doc.Name = "Power_and_Current"
    
    Set plotterObj = plotWin.Object
    plotterObj.Channels.AddVariable "BMS_Control.Pack_Current"
    plotterObj.Channels.AddVariable "BMS_Control.State_Of_Charge"
    
    ' Enable X-Axis Sync across all plots so timelines match perfectly
    app.Commands.Execute "Plotter:EnableXAxisSync"

    MsgBox "BMS Plots generated and scaled successfully!", vbInformation, "Task Complete"
End Sub
