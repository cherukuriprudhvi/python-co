



for msg in App.TransmitMessages:
    if msg.ID == 0x213 and msg.Connection.Name == "CAN1":
        msg.SetSignalValue("EMduleMde_D_Rq3", 1)
        print("CAN1 changed to Standby")
        break