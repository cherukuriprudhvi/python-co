

plotDoc = App.ActiveDocument
plotObj = plotDoc.Object

print("DOC:", plotDoc.Name)
print("OBJECT:", plotObj)

try:
    print("OBJECT KIND:", plotObj.ObjectKind)
except Exception as e:
    print("NO OBJECT KIND:", e)