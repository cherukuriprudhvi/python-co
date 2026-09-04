


def CAN_ON():
    for conn in App.Connections:
        if conn.Name in ["CAN1", "CAN2"]:
            conn.IsEnabled = True

def CAN_OFF():
    for conn in App.Connections:
        if conn.Name in ["CAN1", "CAN2"]:
            conn.IsEnabled = False