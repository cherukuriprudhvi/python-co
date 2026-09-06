

# FILE DESCRIPTION:
# Starts all six separate CAN trace files.
# Records for 10 seconds.
# Stops and saves each trace separately.

import os
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

for doc in trace_docs:
    doc.Tracer.Start()

print("ALL 6 TRACERS STARTED")

App.Wait(10000)

for doc in trace_docs:
    doc.Tracer.Stop()

timestamp = datetime.now().strftime("%Y-%m-%d_%H-%M-%S")

for doc in trace_docs:

    filename = os.path.join(
        log_folder,
        "{}_TEST_{}.trc".format(
            os.path.splitext(doc.Name)[0],
            timestamp
        )
    )

    doc.Save(filename)
    print("Saved:", filename)

print("ALL 6 SEPARATE TRACE TEST FINISHED")