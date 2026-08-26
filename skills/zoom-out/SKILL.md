---
name: zoom-out
description: Use when the user is unfamiliar with an area of the codebase and needs the broader context before details. Map relevant modules, callers, dependencies, data flow, and responsibilities using the project's own vocabulary.
disable-model-invocation: true
---

# Zoom out

Go one level of abstraction above the file, class, function, or error currently being discussed.

Before diving into implementation details, build a compact map of the relevant area:

- entry points
- modules or bounded contexts
- main services/components
- callers and callees
- persistence
- queues/events/workers
- external integrations
- configuration that materially affects the flow

Use terminology already present in the repository and project documentation.

Show the main control/data flow, for example:

```text
Controller / Event / Worker
          ↓
     Application service
          ↓
   Domain / orchestration
          ↓
Repository / external adapter
          ↓
       DB / API
```

Do not enumerate the entire repository. Include only components that help explain how the current area fits into the larger system.

After the map, explain where the current file/class/function sits in that flow and which neighboring components are most important to inspect next.

Do not modify code unless the user separately asked for implementation.
