


print("ALL 6 CHECK STARTED")

for conn in App.Connections:
    if conn.Name in ["CAN1", "CAN2", "CAN3", "CAN4", "CAN5", "CAN6"]:

        if conn.CommunicationObject is not None:
            print(
                conn.Name,
                "| Enabled:", conn.IsEnabled,
                "| Net:", conn.CommunicationObject.NetName
            )
        else:
            print(
                conn.Name,
                "| Enabled:", conn.IsEnabled,
                "| Net: No Net"
            )

print("ALL 6 CHECK FINISHED")