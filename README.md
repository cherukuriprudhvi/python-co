
Option Explicit

Sub TEST_CAN1_CAN2_EXISTING_CHANNELS()

    Dim doc, plotter
    Dim ch1, ch2
    Dim axVolt

    Set doc = Documents.Item("Plot13.plt")
    Set plotter = doc.ActiveWindow.Object

    Set axVolt = plotter.YAxes(1)

    'CAN1 / DBC1
    Set ch1 = plotter.Channels(1)
    Set ch1.Signal = Signals("DBC-1.EnergyMgmtSteeringData_2.EMduleInCirct_U_Actl3")
    Set ch1.YAxis = axVolt
    ch1.Title = "CAN1_InVoltage"

    'CAN2 / DBC2
    Set ch2 = plotter.Channels(2)
    Set ch2.Signal = Signals("DBC-2.EnergyMgmtSteeringData_2.EMduleInCirct_U_Actl3")
    Set ch2.YAxis = axVolt
    ch2.Title = "CAN2_InVoltage"

End Sub