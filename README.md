

# ============================================================
# 2-MINUTE FINAL VALIDATION TEST
#
# 2 cycles:
# FLOAT   = 45 sec
# STANDBY = 15 sec
#
# INITIAL:
# OFF -> 1 sec
# STANDBY -> 1 sec
#
# EACH FLOAT:
# FLOAT -> wait 1 sec
# Isolation OPEN -> wait 1 sec
# Isolation CLOSE
# Continue FLOAT until 45 sec total
#
# SAVE:
# End of minute 1 -> LOG1
# End of minute 2 -> LOG2
#
# FINAL:
# OFF
# ============================================================

import os
import shutil
import time
from datetime import datetime


# ============================================================
# SETTINGS
# ============================================================

BASE_LOG_FOLDER = r"C:\Users\pcherupr\OneDrive - Clarios\Documents\PCAN-Explorer 7\PCAN-Testing\Separate_Trace_Logs_CAN1-6"

CYCLES = 2

FLOAT_SECONDS = 45
STANDBY_SECONDS = 15

OFF_WAIT_SECONDS = 1
INITIAL_STANDBY_WAIT_SECONDS = 1

FLOAT_BEFORE_ISOLATION_SECONDS = 1
ISOLATION_OPEN_SECONDS = 1


# ============================================================
# CREATE NEW FOLDER FOR THIS RUN
# Prevents old logs being overwritten
# ============================================================

RUN_TIME = datetime.now().strftime("%Y-%m-%d_%H-%M-%S")

LOG_FOLDER = os.path.join(
    BASE_LOG_FOLDER,
    "2MIN_TEST_" + RUN_TIME
)

os.makedirs(LOG_FOLDER, exist_ok=True)


# ============================================================
# TRACE DOCUMENTS
# Keep original PCAN trace document names
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
# SIMPLE SAVED LOG NAMES
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
# RENAMED PCAN CONNECTIONS
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

    # CAN6 changed from EBB to EPAS
    "EPAS_8012": {
        "id": 0x213,
        "mode": "EMduleMde_D_Rq3",
        "isolation": "IsolSwtch_B_Cmd3"
    }
}


# ============================================================
# PLOTS / PANEL
# ============================================================

PLOT_NAMES = [
    "Plot1_Final.plt",
    "Plot2_Final.plt",
    "Plot3_Final.plt",
    "Plot4_Final.plt",
    "Plot5_Final.plt",
    "Plot6_Final.plt"
]

PANEL_NAME = "Panel1.ipf"


# ============================================================
# STATUS LOG
# ============================================================

STATUS_FILE = os.path.join(
    LOG_FOLDER,
    "STATUS.txt"
)


def log_status(message):

    line = "{} - {}".format(
        datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
        message
    )

    print(line)

    try:
        with open(STATUS_FILE, "a") as f:
            f.write(line + "\n")
    except:
        pass


# ============================================================
# RESPONSIVE WAIT
# ============================================================

def wait_seconds(seconds):

    end_time = time.monotonic() + seconds

    while time.monotonic() < end_time:
        App.Wait(100)


# ============================================================
# FIND ALL SIX TRACE DOCUMENTS
# ============================================================

trace_docs = []

for doc in App.Documents:

    try:

        tracer = doc.Tracer

        if doc.Name in TRACE_NAMES:
            trace_docs.append(doc)

    except:
        pass


log_status(
    "FOUND {} TRACE FILES".format(
        len(trace_docs)
    )
)


if len(trace_docs) != 6:

    raise Exception(
        "STOPPED - EXPECTED 6 TRACE FILES, FOUND {}".format(
            len(trace_docs)
        )
    )


# ============================================================
# FIND ALL SIX COMMAND MESSAGES
# ============================================================

def find_targets():

    found = {}

    for msg in App.TransmitMessages:

        try:

            connection_name = msg.Connection.Name

            if connection_name in SYSTEMS:

                expected_id = SYSTEMS[
                    connection_name
                ]["id"]

                if msg.ID == expected_id:

                    found[
                        connection_name
                    ] = msg

        except:
            pass

    return found


targets = {}


# Try for up to 10 seconds
for attempt in range(10):

    targets = find_targets()

    if len(targets) == 6:
        break

    log_status(
        "FOUND {}/6 SYSTEM COMMANDS - RETRYING".format(
            len(targets)
        )
    )

    wait_seconds(1)


if len(targets) != 6:

    missing = []

    for name in SYSTEMS:

        if name not in targets:
            missing.append(name)

    raise Exception(
        "STOPPED - MISSING: {}".format(
            ", ".join(missing)
        )
    )


log_status(
    "ALL 6 SYSTEM COMMANDS FOUND"
)


# ============================================================
# MODE CHANGE
# ============================================================

def set_mode(value, mode_name):

    for system_name in SYSTEMS:

        try:

            signal_name = SYSTEMS[
                system_name
            ]["mode"]

            targets[
                system_name
            ].SetSignalValue(
                signal_name,
                value
            )

            log_status(
                "{} -> {}".format(
                    system_name,
                    mode_name
                )
            )

        except Exception as e:

            log_status(
                "ERROR {} {} : {}".format(
                    system_name,
                    mode_name,
                    e
                )
            )


# ============================================================
# ISOLATION OPEN
# ============================================================

def isolation_open():

    for system_name in SYSTEMS:

        signal_name = SYSTEMS[
            system_name
        ]["isolation"]

        # 48V system has no isolation
        if signal_name is None:

            log_status(
                "{} -> ISOLATION SKIPPED".format(
                    system_name
                )
            )

            continue

        try:

            targets[
                system_name
            ].SetSignalValue(
                signal_name,
                0
            )

            log_status(
                "{} Isolation -> OPEN".format(
                    system_name
                )
            )

        except Exception as e:

            log_status(
                "ERROR {} OPEN : {}".format(
                    system_name,
                    e
                )
            )


# ============================================================
# ISOLATION CLOSE
# ============================================================

def isolation_close():

    for system_name in SYSTEMS:

        signal_name = SYSTEMS[
            system_name
        ]["isolation"]

        if signal_name is None:
            continue

        try:

            targets[
                system_name
            ].SetSignalValue(
                signal_name,
                1
            )

            log_status(
                "{} Isolation -> CLOSE".format(
                    system_name
                )
            )

        except Exception as e:

            log_status(
                "ERROR {} CLOSE : {}".format(
                    system_name,
                    e
                )
            )


# ============================================================
# SAVE 6 SEPARATE TRACE FILES
# ============================================================

def save_log_set(log_number, restart=True):

    log_status(
        "===== SAVING LOG{} =====".format(
            log_number
        )
    )


    # Stop all traces first
    for doc in trace_docs:

        try:
            doc.Tracer.Stop()

        except Exception as e:

            log_status(
                "TRACE STOP ERROR {} : {}".format(
                    doc.Name,
                    e
                )
            )


    # Save six separate files
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

            log_status(
                "SAVED -> {}_LOG{}.trc".format(
                    system_name,
                    log_number
                )
            )

        except Exception as e:

            log_status(
                "SAVE ERROR {} : {}".format(
                    doc.Name,
                    e
                )
            )


    # Restart for next minute
    if restart:

        for doc in trace_docs:

            try:
                doc.Tracer.Start()

            except Exception as e:

                log_status(
                    "TRACE RESTART ERROR {} : {}".format(
                        doc.Name,
                        e
                    )
                )

        log_status(
            "ALL 6 TRACES RESTARTED"
        )


# ============================================================
# START PLOTS
# ============================================================

plotters = []

for plot_name in PLOT_NAMES:

    try:

        plot_doc = App.Documents.Item(
            plot_name
        )

        plotter = plot_doc.ActiveWindow.Object

        plotter.Start()

        plotters.append(
            plotter
        )

        log_status(
            "{} -> PLOT STARTED".format(
                plot_name
            )
        )

    except Exception as e:

        log_status(
            "PLOT WARNING {} : {}".format(
                plot_name,
                e
            )
        )


# ============================================================
# PANEL -> RUN MODE
# ============================================================

panel = None

try:

    panel_doc = App.Documents.Item(
        PANEL_NAME
    )

    panel = panel_doc.ActiveWindow.Object

    panel.RunMode = True

    log_status(
        "PANEL -> RUN MODE"
    )

except Exception as e:

    log_status(
        "PANEL WARNING : {}".format(
            e
        )
    )


# ============================================================
# START ALL SIX TRACE LOGGERS
# ============================================================

for doc in trace_docs:

    try:

        doc.Tracer.Start()

        log_status(
            "{} -> TRACE STARTED".format(
                doc.Name
            )
        )

    except Exception as e:

        log_status(
            "TRACE START ERROR {} : {}".format(
                doc.Name,
                e
            )
        )


# ============================================================
# TEST START
# ============================================================

log_status(
    "=================================="
)

log_status(
    "2-MINUTE VALIDATION STARTING"
)

log_status(
    "=================================="
)


# ============================================================
# INITIAL STARTUP ONLY ONCE
#
# OFF -> 1 sec
# STANDBY -> 1 sec
# ============================================================

set_mode(
    0,
    "OFF"
)

wait_seconds(
    OFF_WAIT_SECONDS
)


set_mode(
    1,
    "STANDBY"
)

wait_seconds(
    INITIAL_STANDBY_WAIT_SECONDS
)


# ============================================================
# TWO 1-MINUTE CYCLES
# ============================================================

for cycle in range(1, CYCLES + 1):

    log_status(
        "===== CYCLE {} START =====".format(
            cycle
        )
    )


    # --------------------------------------------------------
    # FLOAT
    # --------------------------------------------------------

    float_start = time.monotonic()

    set_mode(
        3,
        "FLOAT"
    )


    # Wait 1 sec after FLOAT
    wait_seconds(
        FLOAT_BEFORE_ISOLATION_SECONDS
    )


    # Isolation OPEN
    isolation_open()


    # OPEN for exactly 1 sec
    wait_seconds(
        ISOLATION_OPEN_SECONDS
    )


    # Isolation CLOSE
    isolation_close()


    # --------------------------------------------------------
    # Continue FLOAT until 45 sec total
    # --------------------------------------------------------

    while True:

        float_elapsed = (
            time.monotonic() - float_start
        )

        if float_elapsed >= FLOAT_SECONDS:
            break

        App.Wait(100)


    log_status(
        "CYCLE {} -> FLOAT COMPLETE".format(
            cycle
        )
    )


    # --------------------------------------------------------
    # STANDBY
    # --------------------------------------------------------

    set_mode(
        1,
        "STANDBY"
    )

    standby_start = time.monotonic()


    while True:

        standby_elapsed = (
            time.monotonic() - standby_start
        )

        if standby_elapsed >= STANDBY_SECONDS:
            break

        App.Wait(100)


    log_status(
        "CYCLE {} -> STANDBY COMPLETE".format(
            cycle
        )
    )


    # --------------------------------------------------------
    # SAVE LOG SET
    # --------------------------------------------------------

    final_cycle = (
        cycle == CYCLES
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

log_status(
    "ALL SYSTEMS -> FINAL OFF"
)


# ============================================================
# STOP PLOTS
# ============================================================

for plotter in plotters:

    try:
        plotter.Stop()
    except:
        pass


log_status(
    "ALL PLOTS STOPPED"
)


# ============================================================
# PANEL -> DESIGN MODE
# ============================================================

if panel is not None:

    try:

        panel.RunMode = False

        log_status(
            "PANEL -> DESIGN MODE"
        )

    except Exception as e:

        log_status(
            "PANEL DESIGN ERROR : {}".format(
                e
            )
        )


# ============================================================
# FINISHED
# ============================================================

log_status(
    "=================================="
)

log_status(
    "2-MINUTE TEST COMPLETED SUCCESSFULLY"
)

log_status(
    "2 LOG SETS SAVED - 12 TRACE FILES"
)

log_status(
    "=================================="
)