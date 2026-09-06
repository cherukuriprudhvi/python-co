

# FILE DESCRIPTION:
# Enables CAN1 through CAN6 in the current PCAN-Explorer project.
# Prints the ON status for each enabled CAN connection.

for conn in App.Connections:
    if conn.Name in ["CAN1", "CAN2", "CAN3", "CAN4", "CAN5", "CAN6"]:
        conn.IsEnabled = True
        print(conn.Name, "ON")