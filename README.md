



for msg in App.TransmitMessages:
    if msg.ID == 0x213 and msg.Connection.Name == "CAN1":

        # OFF -> STANDBY
        msg.SetSignalValue("EMduleMde_D_Rq3", 1)
        print("CAN1 -> STANDBY")
        App.Wait(5000)

        # STANDBY -> FLOAT
        msg.SetSignalValue("EMduleMde_D_Rq3", 3)
        print("CAN1 -> FLOAT")

        # Isolation pulse after entering Float
        msg.SetSignalValue("IsolSwtch_B_Cmd3", 0)
        print("Isolation -> OPEN")
        App.Wait(500)      # 0.5 second

        msg.SetSignalValue("IsolSwtch_B_Cmd3", 1)
        print("Isolation -> CLOSE")

        # Stay in Float for test
        App.Wait(5000)

        # FLOAT -> STANDBY
        msg.SetSignalValue("EMduleMde_D_Rq3", 1)
        print("CAN1 -> STANDBY")
        App.Wait(5000)

        # STANDBY -> OFF
        msg.SetSignalValue("EMduleMde_D_Rq3", 0)
        print("CAN1 -> OFF")

        break