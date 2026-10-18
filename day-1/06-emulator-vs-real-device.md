# Emulator vs real device

Emulators and simulators are good for layout, navigation logic, and fast iteration. They **cannot** tell you:

- **Real performance.** The emulator runs on your laptop's CPU and GPU. A smooth emulator says nothing about a 2019 TV.
- **Memory pressure.** Real devices kill or reload apps the emulator keeps alive.
- **The real remote.** Key repeat speed, long presses, brand-specific keys, OS-reserved keys, the LG Magic Remote pointer, the Siri Remote touch surface.
- **The real browser engine.** Chrome on your laptop is far newer than the TV's engine. The Tizen and webOS simulators don't match every model year either.
- **Video playback.** Hardware decoders, DRM, codecs, and HDR behave differently on each chipset.
- **The ten-foot experience.** Text that looks fine 50 cm from a monitor can be unreadable from the couch.
- **Overscan, picture modes, and slow startup** on real panels.

Rule: develop on the emulator, **verify every feature on at least one real, low-end device per family** before calling it done.

---

Next: [Remote and focus](./02-remote-and-focus.md)
