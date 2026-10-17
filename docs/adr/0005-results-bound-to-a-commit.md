# Every result is bound to the commit it ran on

Reviews, check verdicts and evidence each record the commit SHA they were produced at. A result recorded at an older commit is stale and cannot satisfy a step, so a new head voids earlier verdicts. We rejected carrying verdicts across rebases by patch identity because it trades a guarantee for speed. A package may later scope such an exception explicitly, but the engine default stays strict.
