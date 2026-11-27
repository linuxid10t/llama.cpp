IMPORTANT: Ensure you’ve thoroughly reviewed the [AGENTS.md](AGENTS.md) file before beginning any work.

---

## Haiku OS port (this fork)

This fork carries a Haiku OS build patch on top of upstream; the same patch is proposed upstream in ggml-org/llama.cpp#33168. See [AGENTS.md > Haiku port](AGENTS.md#haiku-port) for the full list of modified files and the reasoning behind each change. Key constraints when working in this fork:

- Haiku has no `<sys/syslimits.h>`; use `<limits.h>` for `PATH_MAX`.
- The BSD sockets API lives in `libnetwork`, not `libc`. Any new target using sockets must link `network` under a `CMAKE_SYSTEM_NAME STREQUAL "Haiku"` guard.
- `pthread_attr_setinheritsched` is declared but unimplemented on Haiku (wrapped in `#if 0` in `pthread.h`). Avoid code paths that call it: mtmd sets `MA_NO_THREADING` so miniaudio never does. Prefer disabling a feature over patching vendored code; if a direct call is unavoidable, guard it with `!defined(__HAIKU__)`.
- `fs_get_cache_directory()` in `common/common.cpp` needs no patch: its `#else` branch uses the XDG `~/.cache` path, which covers Haiku. If upstream ever adds an `#error` fallback there again, add `__HAIKU__` to the XDG branch.
