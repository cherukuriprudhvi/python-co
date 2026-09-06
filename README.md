

# FILE DESCRIPTION:
# CAN1 short sequence test.
# OFF -> STANDBY -> FLOAT -> Isolation OPEN/CLOSE
# -> STANDBY -> OFF

for msg in App.TransmitMessages:
    if msg.ID == 0x213 and msg.Connection.Name == "CAN1":

        # OFF
        msg.SetSignalValue("EMduleMde_D_Rq3", 0)
        print("CAN1 -> OFF")
        App.Wait(5000)

        # STANDBY
        msg.SetSignalValue("EMduleMde_D_Rq3", 1)
        print("CAN1 -> STANDBY")
        App.Wait(5000)

        # FLOAT
        msg.SetSignalValue("EMduleMde_D_Rq3", 3)
        print("CAN1 -> FLOAT")
        App.Wait(5000)

        # Isolation OPEN
        msg.SetSignalValue("IsolSwtch_B_Cmd3", 0)
        print("Isolation -> OPEN")
        App.Wait(2000)

        # Isolation CLOSE
        msg.SetSignalValue("IsolSwtch_B_Cmd3", 1)
        print("Isolation -> CLOSE")
        App.Wait(2000)

        # FLOAT -> STANDBY
        msg.SetSignalValue("EMduleMde_D_Rq3", 1)
        print("CAN1 -> STANDBY")
        App.Wait(5000)

        # STANDBY -> OFF
        msg.SetSignalValue("EMduleMde_D_Rq3", 0)
        print("CAN1 -> OFF")

        break

print("CAN1 TEST FINISHED")