

# FILE DESCRIPTION:
# Runs the short mode sequence on CAN1-CAN6 together.

targets = []

for msg in App.TransmitMessages:
    if msg.ID == 0x213 and msg.Connection.Name in [
        "CAN1", "CAN2", "CAN3", "CAN4", "CAN5", "CAN6"
    ]:
        targets.append(msg)

# OFF
for msg in targets:
    msg.SetSignalValue("EMduleMde_D_Rq3", 0)

print("ALL 6 -> OFF")
App.Wait(5000)


# STANDBY
for msg in targets:
    msg.SetSignalValue("EMduleMde_D_Rq3", 1)

print("ALL 6 -> STANDBY")
App.Wait(5000)


# FLOAT
for msg in targets:
    msg.SetSignalValue("EMduleMde_D_Rq3", 3)

print("ALL 6 -> FLOAT")


# Immediately OPEN Isolation
for msg in targets:
    msg.SetSignalValue("IsolSwtch_B_Cmd3", 0)

print("ALL 6 Isolation -> OPEN")

# Keep OPEN for 2 seconds
App.Wait(2000)


# CLOSE Isolation
for msg in targets:
    msg.SetSignalValue("IsolSwtch_B_Cmd3", 1)

print("ALL 6 Isolation -> CLOSE")


# Stay in FLOAT another 23 seconds
# Total FLOAT time = 25 seconds
App.Wait(23000)


# FLOAT -> STANDBY
for msg in targets:
    msg.SetSignalValue("EMduleMde_D_Rq3", 1)

print("ALL 6 -> STANDBY")
App.Wait(5000)


# STANDBY -> OFF
for msg in targets:
    msg.SetSignalValue("EMduleMde_D_Rq3", 0)

print("ALL 6 -> OFF")

print("ALL 6 TEST FINISHED")