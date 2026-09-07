

doc = App.Documents.Add(peDocumentKindPlotter)
plotter = doc.ActiveWindow.Object

channel = plotter.Channels.Add()
channel.Signal = Signals("EMduleInCrct_U_Actl3")

print("ONE SIGNAL ADDED")