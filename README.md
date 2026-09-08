
import os
import shutil
import time
from datetime import datetime


# =================================================
# SETTINGS
# =================================================

LOG_FOLDER = r"C:\Users\pcherupr\OneDrive - Clarios\Documents\PCAN-Explorer 7\PCAN-Testing\Separate_Trace_Logs_CAN1-6"

TRACE_NAMES = [
    "CAN_1_FILTER.trc",
    "CAN_2_FILTER.trc",
    "CAN_3_FILTER.trc",
    "CAN_4_FILTER.trc",
    "CAN_5_FILTER.trc",
    "CAN_6_FILTER.trc"
]

PLOT_NAMES = [
    "Plot1_Final.plt",
    "Plot2_Final.plt",
    "Plot3_Final.plt",
    "Plot4_Final.plt",
    "Plot5_Final.plt",
    "Plot6_Final.plt"
]

PANEL_NAME = "Panel1.ipf"

TOTAL_TEST_SECONDS = 300       # 5 minutes
LOG_INTERVAL_SECONDS = 60      # every 1 minute

os.makedirs(LOG_FOLDER, exist_ok=True)

STATUS_FILE = os.path.join(
    LOG_FOLDER,
    "status_log.txt"
)


# =================================================
# STATUS LOG
# =================================================

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


# =================================================
# SYSTEM MAPPING
# =================================================

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


# =================================================
# FIND TRACE DOCUMENTS
# =================================================

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


# =================================================
# FIND MODE TRANSMIT MESSAGES
# =================================================

targets = {}

for msg in App.TransmitMessages:

    try:

        can_name = msg.Connection.Name

        if can_name in SYSTEMS:

            expected_id = SYSTEMS[can_name]["id"]

            if msg.ID == expected_id:
                targets[can_name] = msg

    except:
        pass


log_status(
    "FOUND {} SYSTEM COMMAND MESSAGES".format(
        len(targets)
    )
)


# =================================================
# START ALL 6 PLOTS
# =================================================

plotters = []

for plot_name in PLOT_NAMES:

    try:

        doc = App.Documents.Item(plot_name)

        plotter = doc.ActiveWindow.Object

        plotter.Start()

        plotters.append(plotter)

        log_status(
            "{} -> PLOT STARTED".format(
                plot_name
            )
        )

    except Exception as e:

        log_status(
            "WARNING: Could not start {} : {}".format(
                plot_name,
                e
            )
        )


# =================================================
# PANEL -> RUN MODE
# =================================================

panel = None

try:

    panel_doc = App.Documents.Item(PANEL_NAME)

    panel = panel_doc.ActiveWindow.Object

    panel.RunMode = True

    log_status(
        "Panel1.ipf -> RUN MODE"
    )

except Exception as e:

    log_status(
        "WARNING: Panel Run Mode failed: {}".format(
            e
        )
    )


# =================================================
# SAFE MODE CHANGE
# =================================================

def set_mode(value, mode_name):

    for can_name in [
        "CAN1",
        "CAN2",
        "CAN3",
        "CAN4",
        "CAN5",
        "CAN6"
    ]:

        if can_name not in targets:

            log_status(
                "WARNING: {} mode message unavailable".format(
                    can_name
                )
            )

            continue

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
                "ERROR {} MODE: {}".format(
                    can_name,
                    e
                )
            )


# =================================================
# ISOLATION OPEN
# =================================================

def isolation_open():

    for can_name in targets:

        signal_name = SYSTEMS[can_name]["isolation"]

        # CAN5 has no isolation command
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
                "ERROR {} ISOLATION OPEN: {}".format(
                    can_name,
                    e
                )
            )


# =================================================
# ISOLATION CLOSE
# =================================================

def isolation_close():

    for can_name in targets:

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
                "ERROR {} ISOLATION CLOSE: {}".format(
                    can_name,
                    e
                )
            )


# =================================================
# SAVE TRACE LOG SET
# =================================================

def save_log_set(log_number, restart=True):

    log_status(
        "SAVING LOG SET {}".format(
            log_number
        )
    )

    for doc in trace_docs:

        try:
            doc.Tracer.Stop()

        except Exception as e:

            log_status(
                "TRACE STOP ERROR {}: {}".format(
                    doc.Name,
                    e
                )
            )


    timestamp = datetime.now().strftime(
        "%Y-%m-%d_%H-%M-%S"
    )


    for doc in trace_docs:

        try:

            filename = os.path.join(
                LOG_FOLDER,
                "{}_1MIN_LOG{}_{}.trc".format(
                    os.path.splitext(doc.Name)[0],
                    log_number,
                    timestamp
                )
            )

            shutil.copy2(
                doc.FullName,
                filename
            )

            log_status(
                "SAVED: {}".format(
                    filename
                )
            )

        except Exception as e:

            log_status(
                "TRACE SAVE ERROR {}: {}".format(
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
                    "TRACE RESTART ERROR {}: {}".format(
                        doc.Name,
                        e
                    )
                )

        log_status(
            "ALL AVAILABLE TRACERS RESTARTED"
        )


# =================================================
# START TRACING
# =================================================

for doc in trace_docs:

    try:
        doc.Tracer.Start()

    except Exception as e:

        log_status(
            "TRACE START ERROR {}: {}".format(
                doc.Name,
                e
            )
        )


log_status(
    "5-MINUTE MASTER TEST STARTED"
)


# =================================================
# INITIAL MODE
# =================================================

set_mode(
    1,
    "STANDBY"
)

start_time = time.monotonic()


# =================================================
# EVENT FLAGS
# =================================================

event_5 = False
event_7 = False

event_60 = False

event_90 = False
event_92 = False

event_150 = False

event_180 = False
event_182 = False

event_240 = False

event_295 = False


next_log_time = 60
log_number = 1


# =================================================
# MASTER LOOP
# =================================================

while True:

    elapsed = time.monotonic() - start_time


    # ---------------------------------------------
    # 00:05 FLOAT + OPEN
    # ---------------------------------------------

    if elapsed >= 5 and not event_5:

        set_mode(
            3,
            "FLOAT"
        )

        isolation_open()

        event_5 = True


    # ---------------------------------------------
    # 00:07 CLOSE
    # ---------------------------------------------

    if elapsed >= 7 and not event_7:

        isolation_close()

        event_7 = True


    # ---------------------------------------------
    # 01:00 STANDBY
    # ---------------------------------------------

    if elapsed >= 60 and not event_60:

        set_mode(
            1,
            "STANDBY"
        )

        event_60 = True


    # ---------------------------------------------
    # 01:30 FLOAT + OPEN
    # ---------------------------------------------

    if elapsed >= 90 and not event_90:

        set_mode(
            3,
            "FLOAT"
        )

        isolation_open()

        event_90 = True


    # ---------------------------------------------
    # 01:32 CLOSE
    # ---------------------------------------------

    if elapsed >= 92 and not event_92:

        isolation_close()

        event_92 = True


    # ---------------------------------------------
    # 02:30 STANDBY
    # ---------------------------------------------

    if elapsed >= 150 and not event_150:

        set_mode(
            1,
            "STANDBY"
        )

        event_150 = True


    # ---------------------------------------------
    # 03:00 FLOAT + OPEN
    # ---------------------------------------------

    if elapsed >= 180 and not event_180:

        set_mode(
            3,
            "FLOAT"
        )

        isolation_open()

        event_180 = True


    # ---------------------------------------------
    # 03:02 CLOSE
    # ---------------------------------------------

    if elapsed >= 182 and not event_182:

        isolation_close()

        event_182 = True


    # ---------------------------------------------
    # 04:00 STANDBY
    # ---------------------------------------------

    if elapsed >= 240 and not event_240:

        set_mode(
            1,
            "STANDBY"
        )

        event_240 = True


    # ---------------------------------------------
    # 04:55 FINAL OFF
    # ---------------------------------------------

    if elapsed >= 295 and not event_295:

        set_mode(
            0,
            "OFF"
        )

        event_295 = True


    # =================================================
    # SAVE LOG EVERY 1 MINUTE
    # =================================================

    if elapsed >= next_log_time:

        final_log = (
            next_log_time >= TOTAL_TEST_SECONDS
        )

        save_log_set(
            log_number,
            restart=not final_log
        )

        log_number += 1

        next_log_time += LOG_INTERVAL_SECONDS


    # =================================================
    # END AT 5 MINUTES
    # =================================================

    if elapsed >= TOTAL_TEST_SECONDS:
        break


    App.Wait(200)


# =================================================
# STOP PLOTS
# =================================================

for plotter in plotters:

    try:
        plotter.Stop()

    except:
        pass


log_status(
    "ALL PLOTS STOPPED"
)


# =================================================
# PANEL -> DESIGN MODE
# =================================================

if panel is not None:

    try:

        panel.RunMode = False

        log_status(
            "Panel1.ipf -> DESIGN MODE"
        )

    except Exception as e:

        log_status(
            "Panel Design Mode error: {}".format(
                e
            )
        )


# =================================================
# FINISHED
# =================================================

log_status(
    "5-MINUTE MASTER TEST FINISHED"
)
