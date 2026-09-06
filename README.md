
traceDoc = None
tracer = None

for doc in App.Documents:
    try:
        tracer = doc.Tracer
        traceDoc = doc
        break
    except:
        pass

if traceDoc is None:
    raise Exception("No Trace document is open")
