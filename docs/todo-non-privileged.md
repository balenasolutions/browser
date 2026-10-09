# TODO: Run the browser block without privileged mode

The examples run the browser with `privileged: true`. It would be nice to document a
non-privileged setup as an advanced option.

## What needs privileged mode now

Input doesn't. The display block's compositor handles input devices and Chromium gets
events over the Wayland socket, so the browser no longer runs udev.

From `src/start.sh`:

- `/dev/snd/*` for ALSA audio and `/dev/video*` for V4L2 decode on Pi 3/4. Only
  `/dev/dri` is mapped in the compose file, so these are only visible when privileged.
- `sysctl -w user.max_user_namespaces=10000` and the CPU governor write. Both fail
  silently (`|| true`) without privileges.

Not checked: Chromium's sandbox is still on (`src/server.js`) and uses user namespaces,
which the default seccomp profile may block.

## Things to try

- Pass `/dev/snd` and the Pi decoder nodes with `devices:`. The `/dev/video*` numbers
  differ between device types.
- See if `start.sh`'s GID mapping still works for nodes passed this way.
- If Chromium's sandbox fails, find the smallest `cap_add` / `security_opt` that fixes
  it. Don't use `--no-sandbox`.
- Check which of these compose fields the balena Supervisor supports.

USB sound cards plugged in after the container starts won't show up without privileged
mode. Probably fine for an opt-in setup, but worth noting in the readme.
