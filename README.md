


# FILE DESCRIPTION:
# Records CAN1-CAN6 together in one trace file.
# Saves one combined trace file every 6 hours.
# Runs for 5 days total = 20 trace files.

import os
from datetime import datetime

# Folder where trace files will be saved
log_folder = r"C:\Users\pcherupr\OneDrive - Clarios\Documents\PCAN-Explorer 7\PCAN-Testing\Logs"

# Create the folder if it does not already exist
os.makedirs(log_folder, exist_ok=True)

# IMPORTANT:
# Make sure the Trace .trc tab is active before running this macro.
traceDoc = App.ActiveDocument
tracer = traceDoc.Tracer

print("AUTO TRACE ALL 6 STARTED")

# 20 files x 6 hours = 5 days
for i in range(1, 21):

    print("Starting 6-hour trace", i)

    tracer.Start()

    # Wait 6 hours
    App.Wait(6 * 60 * 60 * 1000)

    tracer.Stop()

    # Create timestamp for filename
    timestamp = datetime.now().strftime("%Y-%m-%d_%H-%M-%S")

    filename = os.path.join(
        log_folder,
        "ALL_6_TRACE_{}_{}.trc".format(i, timestamp)
    )

    # Save the trace file
    traceDoc.Save(filename)

    print("Saved:", filename)

print("5-DAY AUTO TRACE FINISHED")