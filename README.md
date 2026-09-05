



for msg in App.TransmitMessages:
    if msg.ID == 0x213 and msg.Connection.Name == "CAN1":

        msg.SetSignalValue("EMduleMde_D_Rq3", 1)
        print("CAN1 -> STANDBY")
        App.Wait(5000)

        msg.SetSignalValue("EMduleMde_D_Rq3", 3)
        print("CAN1 -> FLOAT")
        App.Wait(5000)

        # Immediate pulse
        msg.SetSignalValue("IsolSwtch_B_Cmd3", 0)
        print("Isolation -> OPEN")

        msg.SetSignalValue("IsolSwtch_B_Cmd3", 1)
        print("Isolation -> CLOSE")

        break
