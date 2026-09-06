

# FILE DESCRIPTION:
# Starts CAN1-CAN6 separate trace files.
# Records for 10 seconds.
# Stops all six tracers.
# Saves each master trace in place.
# Copies each trace into the separate log folder with a timestamp.
# Keeps the original CAN_1_FILTER.trc ... CAN_6_FILTER.trc names unchanged.

import os
import shutil
from datetime import datetime

log_folder = r"C:\Users\pcherupr\OneDrive - Clarios\Documents\PCAN-Explorer 7\PCAN-Testing\Separate_Trace_Logs_CAN1-6"

os.makedirs(log_folder, exist_ok=True)

trace_docs = []

for doc in App.Documents:
    try:
        tracer = doc.Tracer

        if doc.Name in [
            "CAN_1_FILTER.trc",
            "CAN_2_FILTER.trc",
            "CAN_3_FILTER.trc",
            "CAN_4_FILTER.trc",
            "CAN_5_FILTER.trc",
            "CAN_6_FILTER.trc"
        ]:
            trace_docs.append(doc)

    except:
        pass

print("FOUND", len(trace_docs), "TRACE FILES")

if len(trace_docs) != 6:
    raise Exception("Expected 6 trace files, found {}".format(len(trace_docs)))

# Start all 6
for doc in trace_docs:
    doc.Tracer.Start()

print("ALL 6 TRACERS STARTED")

# Record for 10 seconds
App.Wait(10000)

# Stop all 6
for doc in trace_docs:
    doc.Tracer.Stop()

print("ALL 6 TRACERS STOPPED")

timestamp = datetime.now().strftime("%Y-%m-%d_%H-%M-%S")

# Save masters and create timestamped copies
for doc in trace_docs:

    # Save original master trace without renaming it
    doc.Save()

    filename = os.path.join(
        log_folder,
        "{}_TEST_{}.trc".format(
            os.path.splitext(doc.Name)[0],
            timestamp
        )
    )

    # Copy the saved master trace into the log folder
    shutil.copy2(doc.FullName, filename)

    print("Saved copy:", filename)

print("ALL 6 SEPARATE TRACE TEST FINISHED")