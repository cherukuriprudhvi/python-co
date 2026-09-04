print("ALL 6 macro started")

for conn in App.Connections:
    print(
        "Name:", conn.Name,
        "| Enabled:", conn.IsEnabled,
        "| Net:", conn.CommunicationObject.NetName
    )

print("Finished checking connections")