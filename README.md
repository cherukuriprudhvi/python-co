

# ============================================================
# 5-DAY MASTER TEST - PCAN EXPLORER
#
# INITIAL STARTUP:
# OFF -> wait 5 sec
# STANDBY -> wait 5 sec
# FLOAT -> wait 5 sec
# Isolation OPEN -> wait 2 sec
# Isolation CLOSE -> wait 2 sec
#
# TEST:
# FLOAT = 18 hours
# STANDBY = 6 hours
# 5 cycles = approximately 5 days
#
# TRACE:
# 6 separate trace files
# Save every 6 hours
# 4 log sets/day
# 20 log sets total
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
FLOAT_BLOCKS = 3                 # 3 x 6h = 18 hours
STANDBY_BLOCKS = 1               # 1 x 6h = 6 hours

OFF_WAIT_SECONDS = 5
STANDBY_START_WAIT_SECONDS = 5

FLOAT_BEFORE_ISOLATION_SECONDS = 5

ISOLATION_OPEN_SECONDS = 2
AFTER_ISOLATION_CLOSE_SECONDS = 2


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


# Saved files use actual system names
LOG_NAMES = {

    "CAN_1_FILTER.trc": "EPAS_CAN1",
    "CAN_2_FILTER.trc": "EPAS_CAN2",

    "CAN_3_FILTER.trc": "EBB_CAN3",

    "CAN_4_FILTER.trc": "EMB_CAN4",

    "CAN_5_FILTER.trc": "48V_EPAS_CAN5",

    "CAN_6_FILTER.trc": "EBB_CAN6"
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
# SYSTEM COMMAND MAPPING
# ============================================================

SYSTEMS = {

    # EPAS
    "CAN1": {
        "id": 0x213,
        "mode": "EMduleMde_D_Rq3",
        "isolation": "IsolSwtch_B_Cmd3"
    },

    "CAN2": {
        "id": 0x213,
        "mode": "EMduleMde_D_Rq3",
        "isolation": "IsolSwtch_B_Cmd3"
    },

    # EBB
    "CAN3": {
        "id": 0x210,
        "mode": "EMduleMde_D_Rq",
        "isolation": "IsolSwtch_B_Cmd"
    },

    # EMB
    "CAN4": {
        "id": 0x211,
        "mode": "EMduleMde_D_Rq2",
        "isolation": "IsolSwtch_B_Cmd2"
    },

    # 48V EPAS
    "CAN5": {
        "id": 0x212,
        "mode": "UCapMduleMde_D_Rq",
        "isolation": None
    },

    # EBB
    "CAN6": {
        "id": 0x210,
        "mode": "EMduleMde_D_Rq",
        "isolation": "IsolSwtch_B_Cmd"
    }
}


os.makedirs(LOG_FOLDER, exist_ok=True)

STATUS_FILE = os.path.join(
    LOG_FOLDER,
    "5_DAY_TEST_STATUS.txt"
)


# ============================================================
# STATUS LOG
# ============================================================

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
#
# Do NOT use time.sleep() for long waits.
# App.Wait keeps PCAN responsive.
# ============================================================

def wait_seconds(seconds):

    end_time = time.monotonic() + seconds

    while time.monotonic() < end_time:

        remaining = end_time - time.monotonic()

        if remaining <= 0:
            break

        App.Wait(200)


# ============================================================
# FIND ALL 6 TRACE FILES
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
    "FOUND {} TRACE DOCUMENTS".format(
        len(trace_docs)
    )
)


# REAL TEST REQUIRES ALL 6
if len(trace_docs) != 6:

    raise Exception(
        "STOPPED: Expected 6 trace files, found {}".format(
            len(trace_docs)
        )
    )


# ============================================================
# FIND COMMAND MESSAGES
# ============================================================

def find_targets():

    found = {}

    for msg in App.TransmitMessages:

        try:

            can_name = msg.Connection.Name

            if can_name in SYSTEMS:

                if msg.ID == SYSTEMS[can_name]["id"]:

                    found[can_name] = msg

        except:
            pass

    return found


targets = {}


# Retry for 10 seconds before giving up
for attempt in range(10):

    targets = find_targets()

    if len(targets) == 6:
        break

    log_status(
        "Waiting for CAN command messages... found {}/6".format(
            len(targets)
        )
    )

    wait_seconds(1)


log_status(
    "FOUND {} SYSTEM COMMAND MESSAGES".format(
        len(targets)
    )
)


# REAL TEST REQUIRES ALL 6
if len(targets) != 6:

    missing = []

    for can_name in SYSTEMS:

        if can_name not in targets:
            missing.append(can_name)

    raise Exception(
        "STOPPED: Missing command messages: {}".format(
            ", ".join(missing)
        )
    )


# ============================================================
# MODE CONTROL
# ============================================================

def set_mode(value, mode_name):

    for can_name in [
        "CAN1",
        "CAN2",
        "CAN3",
        "CAN4",
        "CAN5",
        "CAN6"
    ]:

        try:

            msg = targets[can_name]

            signal_name = SYSTEMS[can_name]["mode"]

            msg.SetSignalValue(
                signal_name,
                value
            )

            log_status(
                "{} -> {}".format(
                    can_name,
                    mode_name
                )
            )

        except Exception as e:

            log_status(
                "ERROR {} MODE {} : {}".format(
                    can_name,
                    mode_name,
                    e
                )
            )


# ============================================================
# ISOLATION OPEN
# ============================================================

def isolation_open():

    for can_name in [
        "CAN1",
        "CAN2",
        "CAN3",
        "CAN4",
        "CAN5",
        "CAN6"
    ]:

        signal_name = SYSTEMS[can_name]["isolation"]

        # 48V CAN5 has no isolation command
        if signal_name is None:

            log_status(
                "{} -> NO ISOLATION COMMAND".format(
                    can_name
                )
            )

            continue

        try:

            targets[can_name].SetSignalValue(
                signal_name,
                0
            )

            log_status(
                "{} Isolation -> OPEN".format(
                    can_name
                )
            )

        except Exception as e:

            log_status(
                "ERROR {} Isolation OPEN : {}".format(
                    can_name,
                    e
                )
            )


# ============================================================
# ISOLATION CLOSE
# ============================================================

def isolation_close():

    for can_name in [
        "CAN1",
        "CAN2",
        "CAN3",
        "CAN4",
        "CAN5",
        "CAN6"
    ]:

        signal_name = SYSTEMS[can_name]["isolation"]

        if signal_name is None:
            continue

        try:

            targets[can_name].SetSignalValue(
                signal_name,
                1
            )

            log_status(
                "{} Isolation -> CLOSE".format(
                    can_name
                )
            )

        except Exception as e:

            log_status(
                "ERROR {} Isolation CLOSE : {}".format(
                    can_name,
                    e
                )
            )


# ============================================================
# FLOAT TRANSITION
#
# FLOAT
# wait 5 sec
# OPEN
# wait 2 sec
# CLOSE
# wait 2 sec
# ============================================================

def enter_float():

    log_status(
        "STARTING FLOAT TRANSITION"
    )

    set_mode(
        3,
        "FLOAT"
    )

    wait_seconds(
        FLOAT_BEFORE_ISOLATION_SECONDS
    )


    isolation_open()

    wait_seconds(
        ISOLATION_OPEN_SECONDS
    )


    isolation_close()

    wait_seconds(
        AFTER_ISOLATION_CLOSE_SECONDS
    )


    log_status(
        "FLOAT TRANSITION COMPLETE"
    )


# ============================================================
# START ALL PLOTS
# ============================================================

plotters = []

for plot_name in PLOT_NAMES:

    try:

        doc = App.Documents.Item(
            plot_name
        )

        plotter = doc.ActiveWindow.Object

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
            "WARNING: {} plot start failed: {}".format(
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
        "Panel1.ipf -> RUN MODE"
    )

except Exception as e:

    log_status(
        "WARNING: Panel RUN MODE failed: {}".format(
            e
        )
    )


# ============================================================
# START ALL 6 TRACES
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
            "ERROR starting {} : {}".format(
                doc.Name,
                e
            )
        )


# ============================================================
# SAVE SIX SEPARATE TRACE FILES
# ============================================================

def save_log_set(log_number, cycle_number, restart=True):

    log_status(
        "SAVING 6-HOUR LOG SET {}".format(
            log_number
        )
    )


    # -----------------------------------------
    # STOP ALL TRACES
    # -----------------------------------------

    for doc in trace_docs:

        try:

            doc.Tracer.Stop()

        except Exception as e:

            log_status(
                "ERROR stopping {} : {}".format(
                    doc.Name,
                    e
                )
            )


    timestamp = datetime.now().strftime(
        "%Y-%m-%d_%H-%M-%S"
    )


    # -----------------------------------------
    # SAVE SIX INDIVIDUAL FILES
    # -----------------------------------------

    for doc in trace_docs:

        try:

            system_name = LOG_NAMES[
                doc.Name
            ]

            filename = os.path.join(
                LOG_FOLDER,
                "{}_CYCLE{}_6H_LOG{}_{}.trc".format(
                    system_name,
                    cycle_number,
                    log_number,
                    timestamp
                )
            )

            shutil.copy2(
                doc.FullName,
                filename
            )

            log_status(
                "SAVED -> {}".format(
                    filename
                )
            )

        except Exception as e:

            log_status(
                "ERROR saving {} : {}".format(
                    doc.Name,
                    e
                )
            )


    # -----------------------------------------
    # RESTART FOR NEXT 6 HOURS
    # -----------------------------------------

    if restart:

        for doc in trace_docs:

            try:

                doc.Tracer.Start()

            except Exception as e:

                log_status(
                    "ERROR restarting {} : {}".format(
                        doc.Name,
                        e
                    )
                )

        log_status(
            "ALL 6 TRACES RESTARTED"
        )


# ============================================================
# TEST START
# ============================================================

log_status(
    "====================================="
)

log_status(
    "5-DAY MASTER TEST STARTING"
)

log_status(
    "====================================="
)


# ============================================================
# INITIAL STARTUP - ONLY ONCE
#
# OFF 5 SEC
# STANDBY 5 SEC
# THEN FLOAT TRANSITION
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
    STANDBY_START_WAIT_SECONDS
)


# ============================================================
# MAIN 5-DAY TEST
# ============================================================

log_number = 1


for cycle in range(1, CYCLES + 1):

    log_status(
        "====================================="
    )

    log_status(
        "STARTING CYCLE {} OF {}".format(
            cycle,
            CYCLES
        )
    )

    log_status(
        "====================================="
    )


    # --------------------------------------------------------
    # ENTER FLOAT
    # --------------------------------------------------------

    enter_float()


    # --------------------------------------------------------
    # FLOAT = 18 HOURS
    #
    # Save trace after:
    # 6 hours
    # 12 hours
    # 18 hours
    # --------------------------------------------------------

    for float_block in range(
        1,
        FLOAT_BLOCKS + 1
    ):

        log_status(
            "CYCLE {} FLOAT BLOCK {}/3 STARTED".format(
                cycle,
                float_block
            )
        )

        wait_seconds(
            SIX_HOURS
        )


        save_log_set(
            log_number,
            cycle,
            restart=True
        )

        log_number += 1


        log_status(
            "CYCLE {} FLOAT {} HOURS COMPLETE".format(
                cycle,
                float_block * 6
            )
        )


    # --------------------------------------------------------
    # AFTER 18 HOURS -> STANDBY
    # --------------------------------------------------------

    set_mode(
        1,
        "STANDBY"
    )

    log_status(
        "CYCLE {} -> 6-HOUR STANDBY STARTED".format(
            cycle
        )
    )


    # --------------------------------------------------------
    # STANDBY = 6 HOURS
    # --------------------------------------------------------

    wait_seconds(
        SIX_HOURS
    )


    # Final cycle: do not restart tracing after final save
    final_cycle = (
        cycle == CYCLES
    )


    save_log_set(
        log_number,
        cycle,
        restart=not final_cycle
    )

    log_number += 1


    log_status(
        "CYCLE {} COMPLETE - 24 HOURS".format(
            cycle
        )
    )


# ============================================================
# TEST COMPLETE
#
# System remains in STANDBY.
# ============================================================

log_status(
    "====================================="
)

log_status(
    "ALL 5 CYCLES COMPLETE"
)

log_status(
    "SYSTEMS REMAIN IN STANDBY"
)

log_status(
    "TOTAL 6-HOUR LOG SETS = 20"
)

log_status(
    "====================================="
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
            "Panel1.ipf -> DESIGN MODE"
        )

    except Exception as e:

        log_status(
            "Panel DESIGN MODE error: {}".format(
                e
            )
        )


# ============================================================
# FINISHED
# ============================================================

log_status(
    "5-DAY MASTER TEST FINISHED"
)