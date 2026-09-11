

# ============================================================
# 5-DAY MASTER TEST - FINAL
#
# INITIAL STARTUP:
# OFF -> 1 sec
# STANDBY -> 1 sec
#
# EACH FLOAT ENTRY:
# FLOAT -> wait 1 sec
# Isolation OPEN -> wait 1 sec
# Isolation CLOSE
# Continue FLOAT
#
# EACH CYCLE:
# FLOAT = 18 hours
# STANDBY = 6 hours
#
# 5 cycles = 120 hours
#
# TRACE:
# Save SIX separate trace files every 6 hours
#
# FINAL:
# After Day 5 Standby completes -> OFF
# ============================================================

import os
import shutil
import time
from datetime import datetime


# ============================================================
# SETTINGS
# ============================================================

LOG_FOLDER = r"C:\Users\pcherupr\OneDrive - Clarios\Documents\PCAN-Explorer 7\PCAN-Testing\Separate_Trace_Logs_CAN1-6"

CYCLES = 5

SIX_HOURS = 6 * 60 * 60
FLOAT_SECONDS = 18 * 60 * 60
STANDBY_SECONDS = 6 * 60 * 60

OFF_WAIT = 1
INITIAL_STANDBY_WAIT = 1

FLOAT_WAIT = 1
ISOLATION_OPEN_TIME = 1


# ============================================================
# TRACE DOCUMENTS
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
# SAVED LOG NAMES
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
# RENAMED CONNECTIONS
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
# STATUS FILE
# ============================================================

STATUS_FILE = os.path.join(
    LOG_FOLDER,
    "5_DAY_TEST_STATUS.txt"
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
# FIND SIX TRACE DOCUMENTS
# ============================================================

trace_docs = []

for doc in App.Documents:

    try:

        if doc.Name in TRACE_NAMES:

            tracer = doc.Tracer
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
        "STOPPED - Expected 6 trace files, found {}".format(
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

            name = msg.Connection.Name

            if name in SYSTEMS:

                if msg.ID == SYSTEMS[name]["id"]:

                    found[name] = msg

        except:
            pass

    return found


targets = {}


# Retry up to 10 sec
for attempt in range(10):

    targets = find_targets()

    if len(targets) == 6:
        break

    log_status(
        "FOUND {}/6 SYSTEM COMMANDS - RETRY".format(
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

def set_mode(value, text):

    for name in SYSTEMS:

        try:

            targets[name].SetSignalValue(
                SYSTEMS[name]["mode"],
                value
            )

            log_status(
                "{} -> {}".format(
                    name,
                    text
                )
            )

        except Exception as e:

            log_status(
                "ERROR {} {} : {}".format(
                    name,
                    text,
                    e
                )
            )


# ============================================================
# ISOLATION OPEN
# ============================================================

def isolation_open():

    for name in SYSTEMS:

        signal = SYSTEMS[name]["isolation"]

        # 48V has no isolation
        if signal is None:

            log_status(
                "{} -> NO ISOLATION".format(
                    name
                )
            )

            continue

        try:

            targets[name].SetSignalValue(
                signal,
                0
            )

            log_status(
                "{} Isolation -> OPEN".format(
                    name
                )
            )

        except Exception as e:

            log_status(
                "ERROR OPEN {} : {}".format(
                    name,
                    e
                )
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

            log_status(
                "{} Isolation -> CLOSE".format(
                    name
                )
            )

        except Exception as e:

            log_status(
                "ERROR CLOSE {} : {}".format(
                    name,
                    e
                )
            )


# ============================================================
# FLOAT TRANSITION
# ============================================================

def enter_float():

    set_mode(
        3,
        "FLOAT"
    )

    # Wait 1 sec
    wait_seconds(
        FLOAT_WAIT
    )

    # Isolation OPEN
    isolation_open()

    # Keep OPEN for 1 sec
    wait_seconds(
        ISOLATION_OPEN_TIME
    )

    # Isolation CLOSE
    isolation_close()

    log_status(
        "FLOAT TRANSITION COMPLETE"
    )


# ============================================================
# SAVE SIX SEPARATE TRACE FILES
# ============================================================

def save_log_set(log_number, restart=True):

    log_status("")
    log_status(
        "=============================="
    )

    log_status(
        "SAVING LOG{}".format(
            log_number
        )
    )

    log_status(
        "=============================="
    )


    # STOP ALL TRACES
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


    # SAVE SIX SEPARATE FILES
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


    # RESTART TRACES
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


    log_status(
        "LOG{} COMPLETE".format(
            log_number
        )
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
# START ALL SIX TRACES
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
# START TEST
# ============================================================

log_status(
    "=================================="
)

log_status(
    "5-DAY MASTER TEST STARTING"
)

log_status(
    "=================================="
)


# ============================================================
# INITIAL STARTUP ONLY ONCE
#
# OFF 1 SEC
# STANDBY 1 SEC
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
    INITIAL_STANDBY_WAIT
)


# ============================================================
# FIVE 24-HOUR CYCLES
# ============================================================

log_number = 1


for cycle in range(1, CYCLES + 1):

    log_status("")
    log_status(
        "=================================="
    )

    log_status(
        "DAY {} / CYCLE {} START".format(
            cycle,
            cycle
        )
    )

    log_status(
        "=================================="
    )


    # ========================================================
    # FLOAT TRANSITION
    # ========================================================

    enter_float()


    # ========================================================
    # FLOAT = 18 HOURS
    #
    # Save at:
    # 6 hours
    # 12 hours
    # 18 hours
    # ========================================================

    for block in range(1, 4):

        log_status(
            "DAY {} FLOAT BLOCK {} START".format(
                cycle,
                block
            )
        )


        wait_seconds(
            SIX_HOURS
        )


        save_log_set(
            log_number,
            restart=True
        )


        log_number += 1


        log_status(
            "DAY {} FLOAT {} HOURS COMPLETE".format(
                cycle,
                block * 6
            )
        )


    # ========================================================
    # STANDBY = 6 HOURS
    # ========================================================

    set_mode(
        1,
        "STANDBY"
    )


    log_status(
        "DAY {} -> STANDBY 6 HOURS START".format(
            cycle
        )
    )


    wait_seconds(
        STANDBY_SECONDS
    )


    # ========================================================
    # SAVE 24-HOUR LOG
    # ========================================================

    final_cycle = (
        cycle == CYCLES
    )


    save_log_set(
        log_number,
        restart=not final_cycle
    )


    log_number += 1


    log_status(
        "DAY {} / 24 HOURS COMPLETE".format(
            cycle
        )
    )


# ============================================================
# FINAL OFF AFTER DAY 5 STANDBY
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
    "5-DAY TEST COMPLETED"
)

log_status(
    "20 LOG SETS CREATED"
)

log_status(
    "120 TOTAL TRACE FILES"
)

log_status(
    "ALL SYSTEMS OFF"
)

log_status(
    "=================================="
)