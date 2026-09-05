



# CAN1 short timing test

for msg in App.TransmitMessages:
    if msg.ID == 0x213 and msg.Connection.Name == "CAN1":

        # Float
        msg.SetSignalValue("EMduleMde_D_Rq3", 3)
        print("CAN1 -> FLOAT")

        App.Wait(10000)   # 10 seconds

        # Standby
        msg.SetSignalValue("EMduleMde_D_Rq3", 1)
        print("CAN1 -> STANDBY")

        App.Wait(5000)    # 5 seconds

        print("TEST FINISHED")
        break