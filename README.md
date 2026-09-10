

# ============================================================
# 5-MINUTE VALIDATION OF 5-DAY MASTER TEST
#
# SCALE:
# 18 hours FLOAT   -> 45 seconds
# 6 hours STANDBY  -> 15 seconds
# 6-hour trace log -> every 15 seconds
# 5 days           -> 5 minutes
#
# INITIAL:
# OFF -> 5 sec
# STANDBY -> 5 sec
#
# EACH FLOAT ENTRY:
# FLOAT
# wait 5 sec
# Isolation OPEN
# wait 2 sec
# Isolation CLOSE
# wait 2 sec
# Continue FLOAT until total FLOAT time = 45 sec
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

FLOAT_SECONDS = 45
STANDBY_SECONDS = 15
LOG_INTERVAL_SECONDS = 15

OFF_WAIT_SECONDS = 5
INITIAL_STANDBY_WAIT_SECONDS = 5

FLOAT_BEFORE_ISOLATION_SECONDS = 5
ISOLATION_OPEN_SECONDS = 2
AFTER_ISOLATION_CLOSE_SECONDS = 2


# ============================================================
# TRACE FILES
# ============================================================

TRACE_NAMES = [
    "CAN_1_FILTER.trc",
    "CAN_2_FILTER.trc",
    "CAN_3_FILTER.trc",
    "CAN_4_FILTER.trc",
    "CAN_5_FILTER.trc",
    "CAN_6_FILTER.trc"
]


# Saved logs use SYSTEM NAMES
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
# CAN / SYSTEM MAPPING
# ============================================================

SYSTEMS = {

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

    "CAN3": {
        "id": 0x210,
        "mode": "EMduleMde_D_Rq",
        "isolation": "IsolSwtch_B_Cmd"
    },

    "CAN4": {
        "id": 0x211,
        "mode": "EMduleMde_D_Rq2",
        "isolation": "IsolSwtch_B_Cmd2"
    },

    "CAN5": {
        "id": 0x212,
        "mode": "UCapMduleMde_D_Rq",
        "isolation": None
    },

    "CAN6": {
        "id": 0x210,
        "mode": "EMduleMde_D_Rq",
        "isolation": "IsolSwtch_B_Cmd"
    }
}


os.makedirs(LOG_FOLDER, exist_ok=True)

STATUS_FILE = os.path.join(
    LOG_FOLDER,
    "5_MIN_TEST_STATUS.txt"
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
# ============================================================

def wait_seconds(seconds):

    end_time = time.monotonic() + seconds

    while time.monotonic() < end_time:
        App.Wait(200)


# ============================================================
# FIND SIX TRACES
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
    "FOUND {} TRACE FILES".format(len(trace_docs))
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

            can_name = msg.Connection.Name

            if can_name in SYSTEMS:

                if msg.ID == SYSTEMS[can_name]["id"]:
                    found[can_name] = msg

        except:
            pass

    return found


targets = {}

for attempt in range(10):

    targets = find_targets()

    if len(targets) == 6:
        break

    log_status(
        "Waiting for CANs - FOUND {}/6".format(
            len(targets)
        )
    )

    wait_seconds(1)


if len(targets) != 6:

    raise Exception(
        "STOPPED - All 6 CAN command messages not found"
    )


log_status("ALL 6 CAN COMMAND MESSAGES FOUND")


# ============================================================
# MODE
# ============================================================

def set_mode(value, mode_name):

    for can_name in [
        "CAN1", "CAN2", "CAN3",
        "CAN4", "CAN5", "CAN6"
    ]:

        try:

            signal_name = SYSTEMS[can_name]["mode"]

            targets[can_name].SetSignalValue(
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
                "ERROR {} -> {} : {}".format(
                    can_name,
                    mode_name,
                    e
                )
            )


# ============================================================
# ISOLATION OPEN
# ============================================================

def isolation_open():

    for can_name in SYSTEMS:

        signal_name = SYSTEMS[can_name]["isolation"]

        # CAN5 does not have isolation
        if signal_name is None:
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

    for can_name in SYSTEMS:

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
# START PLOTS
# ============================================================

plotters = []

for plot_name in PLOT_NAMES:

    try:

        plot_doc = App.Documents.Item(plot_name)
        plotter = plot_doc.ActiveWindow.Object

        plotter.Start()

        plotters.append(plotter)

        log_status(
            "{} -> STARTED".format(plot_name)
        )

    except Exception as e:

        log_status(
            "PLOT ERROR {} : {}".format(
                plot_name,
                e
            )
        )


# ============================================================
# PANEL RUN MODE
# ============================================================

panel = None

try:

    panel_doc = App.Documents.Item(PANEL_NAME)
    panel = panel_doc.ActiveWindow.Object

    panel.RunMode = True

    log_status("Panel1.ipf -> RUN MODE")

except Exception as e:

    log_status(
        "PANEL ERROR: {}".format(e)
    )


# ============================================================
# START SIX TRACE LOGS
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
# SAVE SIX SEPARATE LOGS
# ============================================================

def save_log_set(log_number, cycle_number, restart=True):

    log_status(
        "SAVING LOG SET {}".format(log_number)
    )


    # Stop all six
    for doc in trace_docs:

        try:
            doc.Tracer.Stop()
        except Exception as e:
            log_status(
                "STOP ERROR {} : {}".format(
                    doc.Name,
                    e
                )
            )


    timestamp = datetime.now().strftime(
        "%Y-%m-%d_%H-%M-%S"
    )


    # Save six different files
    for doc in trace_docs:

        try:

            system_name = LOG_NAMES[doc.Name]

            filename = os.path.join(
                LOG_FOLDER,
                "{}_CYCLE{}_15SEC_LOG{}_{}.trc".format(
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
                "SAVED -> {}".format(filename)
            )

        except Exception as e:

            log_status(
                "SAVE ERROR {} : {}".format(
                    doc.Name,
                    e
                )
            )


    if restart:

        for doc in trace_docs:

            try:
                doc.Tracer.Start()
            except Exception as e:
                log_status(
                    "RESTART ERROR {} : {}".format(
                        doc.Name,
                        e
                    )
                )

        log_status(
            "ALL 6 TRACES RESTARTED"
        )


# ============================================================
# INITIAL STARTUP - ONCE
# ============================================================

log_status("==============================")
log_status("5-MINUTE TEST STARTING")
log_status("==============================")


# OFF
set_mode(0, "OFF")

wait_seconds(
    OFF_WAIT_SECONDS
)


# STANDBY
set_mode(1, "STANDBY")

wait_seconds(
    INITIAL_STANDBY_WAIT_SECONDS
)


# ============================================================
# 5 CYCLES
# ============================================================

log_number = 1


for cycle in range(1, CYCLES + 1):

    log_status(
        "=============================="
    )

    log_status(
        "CYCLE {} OF 5".format(cycle)
    )

    log_status(
        "=============================="
    )


    # ========================================================
    # FLOAT START
    # ========================================================

    float_start = time.monotonic()

    set_mode(3, "FLOAT")


    # Wait 5 sec after entering FLOAT
    wait_seconds(
        FLOAT_BEFORE_ISOLATION_SECONDS
    )


    # OPEN
    isolation_open()

    wait_seconds(
        ISOLATION_OPEN_SECONDS
    )


    # CLOSE
    isolation_close()

    wait_seconds(
        AFTER_ISOLATION_CLOSE_SECONDS
    )


    # ========================================================
    # COMPLETE TOTAL 45 SECOND FLOAT PERIOD
    #
    # Logs at 15, 30, 45 sec
    # ========================================================

    next_float_log = 15

    while True:

        float_elapsed = (
            time.monotonic() - float_start
        )


        if float_elapsed >= next_float_log:

            save_log_set(
                log_number,
                cycle,
                restart=True
            )

            log_number += 1
            next_float_log += 15


        if float_elapsed >= FLOAT_SECONDS:
            break


        App.Wait(200)


    log_status(
        "CYCLE {} -> 45 SEC FLOAT COMPLETE".format(
            cycle
        )
    )


    # ========================================================
    # STANDBY 15 SEC
    # ========================================================

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


        App.Wait(200)


    # 60-sec / simulated 24-hour log
    final_test = (
        cycle == CYCLES
    )


    save_log_set(
        log_number,
        cycle,
        restart=not final_test
    )

    log_number += 1


    log_status(
        "CYCLE {} COMPLETE".format(
            cycle
        )
    )


# ============================================================
# FINISHED
# ============================================================

log_status(
    "ALL 5 TEST CYCLES COMPLETE"
)

log_status(
    "20 SEPARATE LOG SETS CREATED"
)


# ============================================================
# STOP PLOTS
# ============================================================

for plotter in plotters:

    try:
        plotter.Stop()
    except:
        pass


log_status("ALL PLOTS STOPPED")


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
            "PANEL DESIGN ERROR: {}".format(e)
        )


log_status("==============================")
log_status("5-MINUTE VALIDATION FINISHED")
log_status("==============================")