
variables
{
  msTimer stepTimer;
  msTimer txTimer;

  int detectedSystem = 0;
  int step = 0;

  // CAN1 / DBC1 control messages
  message DBC1::EnergyMgmtBodyCtrl_1 ebbMsg;
  message DBC1::EnergyMgmtBodyCtrl_2 embMsg;
  message DBC1::EnergyMgmtBodyCtrl_3 v48Msg;
  message DBC1::EnergyMgmtBodyCtrl_4 epasMsg;
}


/* =========================
   AUTO DETECTION - CAN1
   ========================= */

on message CAN1.*
{
  if (detectedSystem != 0)
    return;

  // EBB
  if (this.id == 273 ||
      this.id == 304 ||
      this.id == 544 ||
      this.id == 560 ||
      this.id == 1024)
  {
    detectedSystem = 1;
    write("CAN1 DETECTED: EBB");
    startSequence();
  }

  // EMB
  else if (this.id == 272 ||
           this.id == 305 ||
           this.id == 545 ||
           this.id == 561 ||
           this.id == 769 ||
           this.id == 1025)
  {
    detectedSystem = 2;
    write("CAN1 DETECTED: EMB");
    startSequence();
  }

  // 48V EPAS
  else if (this.id == 256 ||
           this.id == 306 ||
           this.id == 770)
  {
    detectedSystem = 3;
    write("CAN1 DETECTED: 48V EPAS");
    startSequence();
  }

  // EPAS
  else if (this.id == 309 ||
           this.id == 1026)
  {
    detectedSystem = 4;
    write("CAN1 DETECTED: EPAS");
    startSequence();
  }
}


/* =========================
   START WITH OFF
   ========================= */

void startSequence()
{
  step = 0;

  setMode(0);      // OFF
  setTimer(txTimer, 100);
  setTimer(stepTimer, 5000);

  write("CAN1: OFF");
}


/* =========================
   CYCLIC TRANSMISSION
   ========================= */

on timer txTimer
{
  sendControlMessage();
  setTimer(txTimer, 100);
}


/* =========================
   STARTUP SEQUENCE
   ========================= */

on timer stepTimer
{
  if (step == 0)
  {
    setMode(1);            // STANDBY
    step = 1;

    write("CAN1: STANDBY");
    setTimer(stepTimer, 5000);
  }

  else if (step == 1)
  {
    setMode(3);            // FLOAT
    step = 2;

    write("CAN1: FLOAT");

    // 48V has no isolation
    if (detectedSystem == 3)
    {
      write("CAN1 48V EPAS startup complete");
    }
    else
    {
      setTimer(stepTimer, 4000);
    }
  }

  else if (step == 2)
  {
    setIsolation(0);       // OPEN
    step = 3;

    write("CAN1: Isolation OPEN");
    setTimer(stepTimer, 2000);
  }

  else if (step == 3)
  {
    setIsolation(1);       // CLOSE
    step = 4;

    write("CAN1: Isolation CLOSE");
    write("CAN1 startup sequence complete");
  }
}


/* =========================
   MODE SELECTION
   ========================= */

void setMode(int mode)
{
  if (detectedSystem == 1)
    ebbMsg.EMduleMde_D_Rq = mode;

  else if (detectedSystem == 2)
    embMsg.EMduleMde_D_Rq2 = mode;

  else if (detectedSystem == 3)
    v48Msg.UCapMduleMde_D_Rq = mode;

  else if (detectedSystem == 4)
    epasMsg.EMduleMde_D_Rq3 = mode;

  sendControlMessage();
}


/* =========================
   ISOLATION
   ========================= */

void setIsolation(int value)
{
  if (detectedSystem == 1)
    ebbMsg.IsolSwtch_B_Cmd = value;

  else if (detectedSystem == 2)
    embMsg.IsolSwtch_B_Cmd2 = value;

  else if (detectedSystem == 4)
    epasMsg.IsolSwtch_B_Cmd3 = value;

  sendControlMessage();
}


/* =========================
   SEND CORRECT MESSAGE ONLY
   ========================= */

void sendControlMessage()
{
  if (detectedSystem == 1)
    output(ebbMsg);       // 528 / 0x210

  else if (detectedSystem == 2)
    output(embMsg);       // 529 / 0x211

  else if (detectedSystem == 3)
    output(v48Msg);       // 530 / 0x212

  else if (detectedSystem == 4)
    output(epasMsg);      // 531 / 0x213
}
