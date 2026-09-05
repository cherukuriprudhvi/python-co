

for conn in App.Connections:
    if conn.Name in ["CAN1", "CAN2"]:
        print(conn.Name, "BEFORE =", conn.IsEnabled)

for conn in App.Connections:
    if conn.Name in ["CAN1", "CAN2"]:
        conn.IsEnabled = not conn.IsEnabled
        print(conn.Name, "AFTER =", conn.IsEnabled)