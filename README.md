


def TOGGLE_CAN():
    for conn in App.Connections:
        if conn.Name in ["CAN1", "CAN2"]:
            conn.IsEnabled = not conn.IsEnabled