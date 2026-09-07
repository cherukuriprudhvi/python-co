

for doc in App.Documents:
    try:
        print(doc.Name, " | KIND:", doc.Kind, " | OBJECT:", doc.Object)
    except Exception as e:
        print(doc.Name, " | ERROR:", e)