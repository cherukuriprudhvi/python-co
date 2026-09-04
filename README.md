


for conn in App.Connections:
    if conn.Name in ["CAN1", "CAN2"]:
        conn.IsEnabled = True
        print(conn.Name, "ON")


for conn in App.Connections:
    if conn.Name in ["CAN1", "CAN2"]:
        conn.IsEnabled = False
        print(conn.Name, "OFF")