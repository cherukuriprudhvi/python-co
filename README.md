

# ============================================================
# 5-MINUTE MASTER TEST - ONE COMBINED TRACE
#
# Uses: ALL_6_CAN_TRACE.trc
#
# 00:00 -> STANDBY
# 00:05 -> FLOAT + Isolation OPEN
# 00:06 -> Isolation CLOSE
#
# 01:00 -> STANDBY
# 01:30 -> FLOAT + Isolation OPEN
# 01:31 -> Isolation CLOSE
#
# 02:30 -> STANDBY
# 03:00 -> FLOAT + Isolation OPEN
# 03:01 -> Isolation CLOSE
#
# 04:00 -> STANDBY
# 04:55 -> OFF
# 05:00 -> TEST FINISHED
#
# Saves ONE combined trace every 1 minute.
# ============================================================

import os
import shutil
import time
from datetime import datetime


# ------------------------------------------------------------
# SETTINGS
# ------------------------------------------------------------

LOG_FOLDER = r"C:\Users\pcherupr\OneDrive - Clarios\Documents\PCAN-Explorer 7\PCAN-Testing\Combined_Trace_Logs_ALL6"

TRACE_NAME = "ALL_6_CAN_TRACE.trc"

CAN_NAMES = [
    "CAN1",
    "CAN2",
    "CAN3",
    "CAN4",
    "CAN5",
    "CAN6"
]

TOTAL_TEST_SECONDS = 300       # 5 minutes
LOG_INTERVAL_SECONDS = 60      # 1 minute

os.makedirs(LOG_FOLDER, exist_ok=True)


# ------------------------------------------------------------
# FIND COMBINED TRACE
# ------------------------------------------------------------

trace_doc = None

for doc in App.Documents:
    try:
        tracer = doc.Tracer

        if doc.Name == TRACE_NAME:
            trace_doc = doc
            break

    except:
        pass


if trace_doc is None:
    raise Exception(
        "ALL_6_CAN_TRACE.trc is not open"
    )

print("COMBINED TRACE FOUND")


# ------------------------------------------------------------
# FIND MODE MESSAGES
# ------------------------------------------------------------

targets = []

for msg in App.TransmitMessages:

    try:
        if (
            msg.ID == 0x213
            and msg.Connection.Name in CAN_NAMES
        ):
            targets.append(msg)

    except:
        pass


print("FOUND", len(targets), "MODE MESSAGES")

if len(targets) < 1:
    raise Exception("No mode messages found")


# ------------------------------------------------------------
# FUNCTIONS
# ------------------------------------------------------------

def set_mode(value, text):

    for msg in targets:
        msg.SetSignalValue(
            "EMduleMde_D_Rq3",
            value
        )

    print("MODE ->", text)


def isolation_open():

    for msg in targets:
        msg.SetSignalValue(
            "IsolSwtch_B_Cmd3",
            0
        )

    print("Isolation -> OPEN")


def isolation_close():

    for msg in targets:
        msg.SetSignalValue(
            "IsolSwtch_B_Cmd3",
            1
        )

    print("Isolation -> CLOSE")


def save_combined_log(log_number, restart=True):

    print("SAVING COMBINED LOG", log_number)

    trace_doc.Tracer.Stop()

    timestamp = datetime.now().strftime(
        "%Y-%m-%d_%H-%M-%S"
    )

    filename = os.path.join(
        LOG_FOLDER,
        "ALL_6_CAN_1MIN_LOG{}_{}.trc".format(
            log_number,
            timestamp
        )
    )

    shutil.copy2(
        trace_doc.FullName,
        filename
    )

    print("Saved:", filename)

    if restart:
        trace_doc.Tracer.Start()
        print("COMBINED TRACER RESTARTED")


# ------------------------------------------------------------
# START TRACE IMMEDIATELY
# ------------------------------------------------------------

trace_doc.Tracer.Start()

print("COMBINED ALL-6 TRACE STARTED")


# ------------------------------------------------------------
# INITIAL STANDBY
# ------------------------------------------------------------

set_mode(1, "STANDBY")

start_time = time.monotonic()


# Event flags

event_5 = False
event_6 = False

event_60 = False

event_90 = False
event_91 = False

event_150 = False

event_180 = False
event_181 = False

event_240 = False

event_295 = False


next_log_time = 60
log_number = 1


# ------------------------------------------------------------
# MASTER LOOP
# ------------------------------------------------------------

while True:

    elapsed = time.monotonic() - start_time


    # 00:05 FLOAT + OPEN
    if elapsed >= 5 and not event_5:

        set_mode(3, "FLOAT")
        isolation_open()

        event_5 = True


    # 00:06 CLOSE
    if elapsed >= 6 and not event_6:

        isolation_close()

        event_6 = True


    # 01:00 STANDBY
    if elapsed >= 60 and not event_60:

        set_mode(1, "STANDBY")

        event_60 = True


    # 01:30 FLOAT + OPEN
    if elapsed >= 90 and not event_90:

        set_mode(3, "FLOAT")
        isolation_open()

        event_90 = True


    # 01:31 CLOSE
    if elapsed >= 91 and not event_91:

        isolation_close()

        event_91 = True


    # 02:30 STANDBY
    if elapsed >= 150 and not event_150:

        set_mode(1, "STANDBY")

        event_150 = True


    # 03:00 FLOAT + OPEN
    if elapsed >= 180 and not event_180:

        set_mode(3, "FLOAT")
        isolation_open()

        event_180 = True


    # 03:01 CLOSE
    if elapsed >= 181 and not event_181:

        isolation_close()

        event_181 = True


    # 04:00 STANDBY
    if elapsed >= 240 and not event_240:

        set_mode(1, "STANDBY")

        event_240 = True


    # 04:55 FINAL OFF
    if elapsed >= 295 and not event_295:

        set_mode(0, "OFF")

        event_295 = True


    # --------------------------------------------------------
    # SAVE COMBINED TRACE EVERY 1 MINUTE
    # --------------------------------------------------------

    if elapsed >= next_log_time:

        final_log = next_log_time >= TOTAL_TEST_SECONDS

        save_combined_log(
            log_number,
            restart=not final_log
        )

        log_number += 1
        next_log_time += LOG_INTERVAL_SECONDS


    # --------------------------------------------------------
    # FINISH AT 5 MINUTES
    # --------------------------------------------------------

    if elapsed >= TOTAL_TEST_SECONDS:
        break


    # Short wait = manual Stop Macro remains responsive
    App.Wait(200)


print("5-MINUTE ALL-6 COMBINED TEST FINISHED")