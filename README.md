

print("---- AVAILABLE TRANSMIT MESSAGES ----")

for msg in App.TransmitMessages:
    try:
        print(
            msg.Connection.Name,
            hex(msg.ID)
        )
    except:
        pass