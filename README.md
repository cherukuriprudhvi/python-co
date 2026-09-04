print("ALL 6 macro started")

for conn in App.Connections:

    net_name = "No Net"

    if conn.CommunicationObject is not None:

        net_name = conn.CommunicationObject.NetName

    print(

        "Name:", conn.Name,

        "| Enabled:", conn.IsEnabled,

        "| Net:", net_name

    )

print("Finished checking connections")