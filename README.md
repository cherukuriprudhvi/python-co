



targets = []

for conn in App.Connections:
    if conn.Name in ["CAN1", "CAN2"]:
        targets.append(conn)

# If both are ON, turn both OFF.
# Otherwise, turn both ON.
turn_on = not all(conn.IsEnabled for conn in targets)

for conn in targets:
    conn.IsEnabled = turn_on
    print(conn.Name, "ON" if turn_on else "OFF")