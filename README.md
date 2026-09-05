


targets = [conn for conn in App.Connections if conn.Name in ["CAN1", "CAN2"]]

if len(targets) == 0:
    print("CAN1/CAN2 not found")

else:
    # Check current state of CAN1 only
    turn_on = not targets[0].IsEnabled

    for conn in targets:
        conn.IsEnabled = turn_on
        print(conn.Name, "ON" if turn_on else "OFF")