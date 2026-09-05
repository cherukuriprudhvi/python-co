



for sig in App.Signals:
    if "EMduleMde_D_Rq3" in sig.Name:
        print("FOUND:", sig.Name)
        print("BEFORE:", sig.Value)

        sig.Value = 1

        print("AFTER:", sig.Value)
