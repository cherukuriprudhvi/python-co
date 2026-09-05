



for msg in App.TransmitMessages:
    if msg.ID == 0x213:
        print(
            "ID:", hex(msg.ID),
            "| Connection:", msg.Connection.Name
        )