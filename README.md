
Option Explicit

Sub TEST_QUALIFIED_SIGNALS()

    Dim s1, s2

    Set s1 = Signals("DBC-1.EnergyMgmtSteeringData_2.EMduleInCirct_U_Actl3")
    Set s2 = Signals("DBC-2.EnergyMgmtSteeringData_2.EMduleInCirct_U_Actl3")

    PrintToOutputWindow "DBC1 FOUND: " & s1.Name
    PrintToOutputWindow "DBC2 FOUND: " & s2.Name

End Sub