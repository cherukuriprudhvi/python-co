

# FILE DESCRIPTION:
# Disables CAN1 through CAN6 in the current PCAN-Explorer project.
# Prints the OFF status for each disabled CAN connection.

for conn in App.Connections:
    if conn.Name in ["CAN1", "CAN2", "CAN3", "CAN4", "CAN5", "CAN6"]:
        conn.IsEnabled = False
        print(conn.Name, "OFF")