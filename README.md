



for sig in App.Signals:
    if sig.Name == "DBC-1.EnergyMgmtBodyCtrl_4.EMduleMde_D_Rq3":
        print("BEFORE:", sig.Value)

        sig.Value = 1   # 1 = Standby

        print("AFTER:", sig.Value)
        print("CAN1 set to Standby")
        break