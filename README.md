





def ALL_6_START():
    connection_names = [
        "Untitled@pcan_usb",
        "Untitled1@pcan_usb",
        "Untitled2@pcan_usb",
        "Untitled3@pcan_usb",
        "Untitled4@pcan_usb",
        "Untitled6@pcan_usb"
    ]
    
    # 2. Initialize the PCAN client interface
    try:
        myClient = win32com.client.Dispatch("PCAN.PCANClient")
        ...
