

import win32com.client
import time

# Connect to your active PCAN-Explorer 7 instance
try:
    pcan = win32com.client.GetActiveObject("PCANExplorer.Application")
except:
    pcan = win32com.client.Dispatch("PCANExplorer.Application")

# 1. Close existing messy plots to free screen space
for doc in list(pcan.Documents):
    if doc.Name.lower().endswith('.plt') or "plot" in doc.Name.lower():
        doc.Close()

def create_epas_plot(title, signals, y_min, y_max, autoscale, left, top, width, height):
    # Add a new Plotter document type (Kind = 5 or peDocumentKindPlotter)
    doc = pcan.Documents.Add(5) 
    doc.Name = title
    
    # Configure the screen window positioning
    win = doc.ActiveWindow
    win.Left = left
    win.Top = top
    win.Width = width
    win.Height = height
    
    # Access the base Plotter object
    plotter = win.Object
    
    # Add signals sequentially 
    for sig in signals:
        try:
            # PCAN-Explorer Python COM syntax for adding a variable to a plot
            plotter.AddVariable(sig)
        except Exception as e:
            print(f"Warning: Could not bind signal {sig}. Check your .sym/DBC paths. Error: {e}")
            
    # Apply Y-Axis scaling properties safely
    try:
        # Most versions expose YAxes directly on the base plotter view object
        plotter.YAxes(0).Autoscale = autoscale
        if not autoscale:
            plotter.YAxes(0).Min = y_min
            plotter.YAxes(0).Max = y_max
    except Exception as e:
        print(f"Note: Y-Axis scaling properties skipped due to object model variances: {e}")

# 2. Define your exact EPAS signal listings based on your live symbols
electrical_signals = [
    "EPAS_Volt.EMduleInCrct_U_Actl2",
    "EPAS_Volt.EMduleOutCrct_U_Actl2",
    "EPAS_Volt.Cell_U_Actl2",
    "EPAS_Current.EMduleInCrct_I_Actl2",
    "EPAS_Current.EMduleOutCrct_I_Actl2"
]

thermal_signals = [
    "EPAS_Temperature.FET_Te_Actl2",
    "EPAS_Temperature.Cel_Te_Actl2"
]

# 3. Launch and layout the windows side-by-side cleanly
print("Deploying optimized Python EPAS plots...")

create_epas_plot(
    title="EPAS_Electrical", 
    signals=electrical_signals, 
    y_min=0.0, y_max=16.0, autoscale=False,
    left=0, top=0, width=650, height=450
)

create_epas_plot(
    title="EPAS_Thermal", 
    signals=thermal_signals, 
    y_min=15.0, y_max=100.0, autoscale=False,
    left=655, top=0, width=650, height=450
)

# Enable timeline synchronization across windows
try:
    pcan.Commands.Execute("Plotter:EnableXAxisSync")
except:
    pass

print("Plots successfully configured via Python interface.")
