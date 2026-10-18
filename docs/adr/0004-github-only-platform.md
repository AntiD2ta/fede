# GitHub is the only platform fede talks to

fede integrates with GitHub (through `gh`, amended by ADR 0013 to `go-github` with `gh` optional) and nothing else. Issue trackers, notifications and evidence uploads are done by the coordinator with its own tools, prompted by fede items that the coordinator acknowledges. We rejected tracker and notifier adapters inside fede because the harness already holds those credentials and integrations, and each adapter would duplicate them.
