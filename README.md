



for msg in App.TransmitMessages:
    if msg.ID == 0x213 and msg.Connection.Name == "CAN1":

        # Off -> Standby
        msg.SetSignalValue("EMduleMde_D_Rq3", 1)
        print("CAN1 -> STANDBY")
        App.Wait(5000)

        # Standby -> Float
        msg.SetSignalValue("EMduleMde_D_Rq3", 3)
        print("CAN1 -> FLOAT")
        App.Wait(5000)

        # Float -> Standby
        msg.SetSignalValue("EMduleMde_D_Rq3", 1)
        print("CAN1 -> STANDBY")
        App.Wait(5000)

        # Standby -> Off
        msg.SetSignalValue("EMduleMde_D_Rq3", 0)
        print("CAN1 -> OFF")

        break
