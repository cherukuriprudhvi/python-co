
# FILE DESCRIPTION:
# 10-minute master test for CAN1-CAN6.
#
# Automatically:
# - Starts all 6 separate trace logs
# - Repeats STANDBY / FLOAT mode sequence
# - Pulses Isolation OPEN for 2 seconds, then CLOSE
# - Saves separate CAN1-CAN6 trace logs every 2 minutes
# - Goes to OFF only at the very end
#
# LOG TIMES:
# 2, 4, 6, 8, 10 minutes
#
# MODE TIMES:
# 00:00 -> STANDBY
# 00:10 -> FLOAT + Isolation OPEN
# 00:12 -> Isolation CLOSE
# 02:00 -> STANDBY
# 03:00 -> FLOAT + Isolation OPEN
# 03:02 -> Isolation CLOSE
# 05:00 -> STANDBY
# 06:00 -> FLOAT + Isolation OPEN
# 06:02 -> Isolation CLOSE
# 08:00 -> STANDBY
# 09:50 -> OFF
# 10:00 -> TEST FINISHED

import os
import shutil
import time
from datetime import datetime


# -------------------------------------------------
# SETTINGS
# -------------------------------------------------

LOG_FOLDER = r"C:\Users\pcherupr\OneDrive - Clarios\Documents\PCAN-Explorer 7\PCAN-Testing\Separate_Trace_Logs_CAN1-6"

TRACE_NAMES = [
    "CAN_1_FILTER.trc",
    "CAN_2_FILTER.trc",
    "CAN_3_FILTER.trc",
    "CAN_4_FILTER.trc",
    "CAN_5_FILTER.trc",
    "CAN_6_FILTER.trc"
]

CAN_NAMES = [
    "CAN1",
    "CAN2",
    "CAN3",
    "CAN4",
    "CAN5",
    "CAN6"
]

TOTAL_TEST_SECONDS = 600       # 10 minutes
LOG_INTERVAL_SECONDS = 120     # every 2 minutes

os.makedirs(LOG_FOLDER, exist_ok=True)


# -------------------------------------------------
# FIND ALL 6 TRACE DOCUMENTS
# -------------------------------------------------

trace_docs = []

for doc in App.Documents:
    try:
        tracer = doc.Tracer

        if doc.Name in TRACE_NAMES:
            trace_docs.append(doc)

    except:
        pass

print("FOUND", len(trace_docs), "TRACE FILES")

if len(trace_docs) != 6:
    raise Exception(
        "Expected 6 trace files, found {}".format(len(trace_docs))
    )


# -------------------------------------------------
# FIND ALL 6 MODE MESSAGES
# -------------------------------------------------

targets = []

for msg in App.TransmitMessages:

    if (
        msg.ID == 0x213
        and msg.Connection.Name in CAN_NAMES
    ):
        targets.append(msg)

print("FOUND", len(targets), "MODE MESSAGES")

if len(targets) != 6:
    raise Exception(
        "Expected 6 mode messages, found {}".format(len(targets))
    )


# -------------------------------------------------
# HELPER FUNCTIONS
# -------------------------------------------------

def set_mode(value, text):

    for msg in targets:
        msg.SetSignalValue(
            "EMduleMde_D_Rq3",
            value
        )

    print("ALL 6 ->", text)


def isolation_open():

    for msg in targets:
        msg.SetSignalValue(
            "IsolSwtch_B_Cmd3",
            0
        )

    print("ALL 6 Isolation -> OPEN")


def isolation_close():

    for msg in targets:
        msg.SetSignalValue(
            "IsolSwtch_B_Cmd3",
            1
        )

    print("ALL 6 Isolation -> CLOSE")


def save_log_set(log_number, restart=True):

    print("SAVING LOG SET", log_number)

    # Stop all 6 traces
    for doc in trace_docs:
        doc.Tracer.Stop()

    timestamp = datetime.now().strftime(
        "%Y-%m-%d_%H-%M-%S"
    )

    # Create six separate copies
    for doc in trace_docs:

        filename = os.path.join(
            LOG_FOLDER,
            "{}_2MIN_LOG{}_{}.trc".format(
                os.path.splitext(doc.Name)[0],
                log_number,
                timestamp
            )
        )

        shutil.copy2(
            doc.FullName,
            filename
        )

        print("Saved:", filename)

    # Start next trace period
    if restart:

        for doc in trace_docs:
            doc.Tracer.Start()

        print("ALL 6 TRACERS RESTARTED")


# -------------------------------------------------
# START LOGGING IMMEDIATELY
# -------------------------------------------------

for doc in trace_docs:
    doc.Tracer.Start()

print("ALL 6 TRACERS STARTED")


# -------------------------------------------------
# INITIAL MODE
# -------------------------------------------------

set_mode(1, "STANDBY")

start_time = time.monotonic()

event_10 = False
event_12 = False
event_120 = False
event_180 = False
event_182 = False
event_300 = False
event_360 = False
event_362 = False
event_480 = False
event_590 = False

next_log_time = 120
log_number = 1


# -------------------------------------------------
# MASTER TEST LOOP
# -------------------------------------------------

while True:

    elapsed = time.monotonic() - start_time


    # 00:10 -> FLOAT + OPEN
    if elapsed >= 10 and not event_10:

        set_mode(3, "FLOAT")
        isolation_open()

        event_10 = True


    # 00:12 -> CLOSE
    if elapsed >= 12 and not event_12:

        isolation_close()

        event_12 = True


    # 02:00 -> STANDBY
    if elapsed >= 120 and not event_120:

        set_mode(1, "STANDBY")

        event_120 = True


    # 03:00 -> FLOAT + OPEN
    if elapsed >= 180 and not event_180:

        set_mode(3, "FLOAT")
        isolation_open()

        event_180 = True


    # 03:02 -> CLOSE
    if elapsed >= 182 and not event_182:

        isolation_close()

        event_182 = True


    # 05:00 -> STANDBY
    if elapsed >= 300 and not event_300:

        set_mode(1, "STANDBY")

        event_300 = True


    # 06:00 -> FLOAT + OPEN
    if elapsed >= 360 and not event_360:

        set_mode(3, "FLOAT")
        isolation_open()

        event_360 = True


    # 06:02 -> CLOSE
    if elapsed >= 362 and not event_362:

        isolation_close()

        event_362 = True


    # 08:00 -> STANDBY
    if elapsed >= 480 and not event_480:

        set_mode(1, "STANDBY")

        event_480 = True


    # 09:50 -> FINAL OFF
    if elapsed >= 590 and not event_590:

        set_mode(0, "OFF")

        event_590 = True


    # -------------------------------------------------
    # SAVE LOGS EVERY 2 MINUTES
    # -------------------------------------------------

    if elapsed >= next_log_time:

        final_log = next_log_time >= TOTAL_TEST_SECONDS

        save_log_set(
            log_number,
            restart=not final_log
        )

        log_number += 1
        next_log_time += LOG_INTERVAL_SECONDS


    # -------------------------------------------------
    # END AT 10 MINUTES
    # -------------------------------------------------

    if elapsed >= TOTAL_TEST_SECONDS:
        break


    # Short wait keeps PCAN responsive
    App.Wait(200)


print("10-MINUTE MASTER TEST FINISHED")