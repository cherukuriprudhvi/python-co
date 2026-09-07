

# TEST - CREATE ONE PCAN PLOTTER

doc = App.Documents.Add(peDocumentKindPlotter)

if doc is None:
    print("PLOT CREATION FAILED")
else:
    print("PLOT CREATED SUCCESSFULLY")
    print("Document:", doc.Name)