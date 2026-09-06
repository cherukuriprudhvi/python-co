import os
from datetime import datetime

log_folder = r"C:\Users\pcherupr\OneDrive - Clarios\Documents\PCAN-Explorer 7\PCAN-Testing\Logs"

os.makedirs(log_folder, exist_ok=True)

traceDoc = App.ActiveDocument
tracer = traceDoc.Tracer

print("AUTO TRACE TEST STARTED")

for i in range(1, 4):

    tracer.Start()
    print("Recording test", i)

    App.Wait(10000)   # 10 seconds only for test

    tracer.Stop()

    timestamp = datetime.now().strftime("%Y-%m-%d_%H-%M-%S")
    filename = os.path.join(
        log_folder,
        "Trace_Test_{}_{}.trc".format(i, timestamp)
    )

    traceDoc.Save(filename)

    print("Saved:", filename)

print("AUTO TRACE TEST FINISHED")

