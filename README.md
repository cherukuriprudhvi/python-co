


# Find the CAN1 mode signal and change it once to Standby

for sig in App.Signals:
    if sig.Name.startswith("DBC-1.") and "EMduleMde_D_Rq3" in sig.Name:
        sig.Value = 1
        print("CAN1 changed to Standby")
        break