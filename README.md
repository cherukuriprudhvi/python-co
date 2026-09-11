

# ============================================================
# 2-MINUTE VALIDATION TEST
#
# INITIAL:
# OFF -> 1 sec
# STANDBY -> 1 sec
#
# EACH 1-MINUTE CYCLE:
# FLOAT -> wait 1 sec
# Isolation OPEN -> wait 1 sec
# Isolation CLOSE
# Continue FLOAT until 45 sec total
# STANDBY for 15 sec
#
# END MINUTE 1 -> SAVE LOG1
# END MINUTE 2 -> SAVE LOG2
#
# FINAL -> OFF
# ============================================================

import os
import shutil
import time


# ============================================================
# EXISTING LOG FOLDER - NO NEW FOLDER CREATED
# ============================================================

LOG_FOLDER = r"C:\Users\pcherupr\OneDrive - Clarios\Documents\PCAN-Explorer 7\PCAN-Testing\Separate_Trace_Logs_CAN1-6"


# ============================================================
# TIMING
# ============================================================

FLOAT_SECONDS = 45
STANDBY_SECONDS = 15

OFF_WAIT = 1
STANDBY_START_WAIT = 1

FLOAT_WAIT = 1
ISOLATION_OPEN_TIME = 1


# ============================================================
# EXISTING TRACE DOCUMENTS
# ============================================================

TRACE_NAMES = [
    "CAN_1_FILTER.trc",
    "CAN_2_FILTER.trc",
    "CAN_3_FILTER.trc",
    "CAN_4_FILTER.trc",
    "CAN_5_FILTER.trc",
    "CAN_6_FILTER.trc"
]


# ============================================================
# SAVED TRACE NAMES
# ============================================================

LOG_NAMES = {
    "CAN_1_FILTER.trc": "EPAS_8005",
    "CAN_2_FILTER.trc": "EPAS_8009",
    "CAN_3_FILTER.trc": "EBB_5002",
    "CAN_4_FILTER.trc": "EMB_6007",
    "CAN_5_FILTER.trc": "48_V_EPAS_9017",
    "CAN_6_FILTER.trc": "EPAS_8012"
}


# ============================================================
# RENAMED CONNECTIONS + COMMAND SIGNALS
# ============================================================

SYSTEMS = {

    "EPAS_8005": {
        "id": 0x213,
        "mode": "EMduleMde_D_Rq3",
        "isolation": "IsolSwtch_B_Cmd3"
    },

    "EPAS_8009": {
        "id": 0x213,
        "mode": "EMduleMde_D_Rq3",
        "isolation": "IsolSwtch_B_Cmd3"
    },

    "EBB_5002": {
        "id": 0x210,
        "mode": "EMduleMde_D_Rq",
        "isolation": "IsolSwtch_B_Cmd"
    },

    "EMB_6007": {
        "id": 0x211,
        "mode": "EMduleMde_D_Rq2",
        "isolation": "IsolSwtch_B_Cmd2"
    },

    "48_V_EPAS_9017": {
        "id": 0x212,
        "mode": "UCapMduleMde_D_Rq",
        "isolation": None
    },

    "EPAS_8012": {
        "id": 0x213,
        "mode": "EMduleMde_D_Rq3",
        "isolation": "IsolSwtch_B_Cmd3"
    }
}


# ============================================================
# RESPONSIVE WAIT
# ============================================================

def wait_seconds(seconds):

    end_time = time.monotonic() + seconds

    while time.monotonic() < end_time:
        App.Wait(100)


# ============================================================
# FIND 6 TRACE DOCUMENTS
# ============================================================

trace_docs = []

for doc in App.Documents:

    try:

        if doc.Name in TRACE_NAMES:

            tracer = doc.Tracer
            trace_docs.append(doc)

    except:
        pass


print("FOUND", len(trace_docs), "TRACE FILES")

if len(trace_docs) != 6:

    raise Exception(
        "STOPPED - Expected 6 traces, found {}".format(
            len(trace_docs)
        )
    )


# ============================================================
# FIND ALL 6 COMMAND MESSAGES
# ============================================================

targets = {}

for msg in App.TransmitMessages:

    try:

        name = msg.Connection.Name

        if name in SYSTEMS:

            if msg.ID == SYSTEMS[name]["id"]:
                targets[name] = msg

    except:
        pass


print("FOUND", len(targets), "SYSTEM COMMANDS")

if len(targets) != 6:

    raise Exception(
        "STOPPED - Did not find all 6 system commands"
    )


# ============================================================
# MODE CHANGE
# ============================================================

def set_mode(value, text):

    for name in SYSTEMS:

        try:

            targets[name].SetSignalValue(
                SYSTEMS[name]["mode"],
                value
            )

            print(name, "->", text)

        except Exception as e:

            print(
                "ERROR",
                name,
                text,
                e
            )


# ============================================================
# ISOLATION OPEN
# ============================================================

def isolation_open():

    for name in SYSTEMS:

        signal = SYSTEMS[name]["isolation"]

        if signal is None:
            print(name, "-> NO ISOLATION")
            continue

        try:

            targets[name].SetSignalValue(
                signal,
                0
            )

            print(
                name,
                "Isolation -> OPEN"
            )

        except Exception as e:

            print(
                "ERROR OPEN",
                name,
                e
            )


# ============================================================
# ISOLATION CLOSE
# ============================================================

def isolation_close():

    for name in SYSTEMS:

        signal = SYSTEMS[name]["isolation"]

        if signal is None:
            continue

        try:

            targets[name].SetSignalValue(
                signal,
                1
            )

            print(
                name,
                "Isolation -> CLOSE"
            )

        except Exception as e:

            print(
                "ERROR CLOSE",
                name,
                e
            )


# ============================================================
# SAVE ONE LOG SET
# ============================================================

def save_log_set(log_number, restart=True):

    print("")
    print("========================")
    print("SAVING LOG{}".format(log_number))
    print("========================")


    # STOP ALL SIX TRACES
    for doc in trace_docs:

        try:
            doc.Tracer.Stop()
        except Exception as e:
            print(
                "TRACE STOP ERROR",
                doc.Name,
                e
            )


    # SAVE 6 DIFFERENT FILES
    for doc in trace_docs:

        try:

            system_name = LOG_NAMES[
                doc.Name
            ]

            filename = os.path.join(
                LOG_FOLDER,
                "{}_LOG{}.trc".format(
                    system_name,
                    log_number
                )
            )

            shutil.copy2(
                doc.FullName,
                filename
            )

            print(
                "SAVED ->",
                "{}_LOG{}.trc".format(
                    system_name,
                    log_number
                )
            )

        except Exception as e:

            print(
                "SAVE ERROR",
                doc.Name,
                e
            )


    if restart:

        for doc in trace_docs:

            try:
                doc.Tracer.Start()
            except Exception as e:
                print(
                    "TRACE RESTART ERROR",
                    doc.Name,
                    e
                )


        print(
            "ALL 6 TRACES RESTARTED"
        )


    print(
        "LOG{} COMPLETED".format(
            log_number
        )
    )


# ============================================================
# START ALL SIX TRACES
# ============================================================

for doc in trace_docs:

    doc.Tracer.Start()


print("")
print("========================")
print("2-MIN TEST STARTED")
print("========================")


# ============================================================
# INITIAL STARTUP
# ============================================================

set_mode(
    0,
    "OFF"
)

wait_seconds(
    OFF_WAIT
)


set_mode(
    1,
    "STANDBY"
)

wait_seconds(
    STANDBY_START_WAIT
)


# ============================================================
# TWO 1-MINUTE CYCLES
# ============================================================

for cycle in range(1, 3):

    print("")
    print(
        "STARTING CYCLE",
        cycle
    )


    # ----------------------------------------
    # FLOAT
    # ----------------------------------------

    float_start = time.monotonic()

    set_mode(
        3,
        "FLOAT"
    )


    # Wait 1 second
    wait_seconds(
        FLOAT_WAIT
    )


    # Isolation OPEN
    isolation_open()


    # Keep open for 1 second
    wait_seconds(
        ISOLATION_OPEN_TIME
    )


    # Isolation CLOSE
    isolation_close()


    # Continue FLOAT until total 45 sec
    while (
        time.monotonic() - float_start
        < FLOAT_SECONDS
    ):

        App.Wait(100)


    print(
        "FLOAT 45 SEC COMPLETE"
    )


    # ----------------------------------------
    # STANDBY
    # ----------------------------------------

    set_mode(
        1,
        "STANDBY"
    )


    wait_seconds(
        STANDBY_SECONDS
    )


    print(
        "STANDBY 15 SEC COMPLETE"
    )


    # ----------------------------------------
    # SAVE
    # ----------------------------------------

    final_cycle = (
        cycle == 2
    )


    save_log_set(
        cycle,
        restart=not final_cycle
    )


# ============================================================
# FINAL OFF
# ============================================================

set_mode(
    0,
    "OFF"
)


print("")
print("========================")
print("TEST COMPLETED")
print("LOG1 + LOG2 SAVED")
print("ALL SYSTEMS -> OFF")
print("========================")