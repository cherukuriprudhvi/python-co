

SYSTEMS = {
    "CAN1": (0x213, "EMduleMde_D_Rq3"),
    "CAN2": (0x213, "EMduleMde_D_Rq3"),
    "CAN3": (0x210, "EMduleMde_D_Rq"),
    "CAN4": (0x211, "EMduleMde_D_Rq2"),
    "CAN5": (0x212, "UCapMduleMde_D_Rq"),
    "CAN6": (0x210, "EMduleMde_D_Rq")
}

for msg in App.TransmitMessages:
    try:
        can_name = msg.Connection.Name

        if can_name in SYSTEMS:
            expected_id, signal_name = SYSTEMS[can_name]

            if msg.ID == expected_id:
                msg.SetSignalValue(signal_name, 0)
                print(can_name, "-> OFF SUCCESS")

    except Exception as e:
        print("ERROR:", e)

print("ALL AVAILABLE SYSTEMS -> OFF")