# Troubleshooting workflow

TV bugs are often hard to see on your desk. They happen on one model, after holding a key for a while, after the network drops, or only in the release build. Guessing and changing three things at once wastes days. This file gives you a fixed order to work in: **reproduce, classify, collect, compare, one change**. The [debug exercises](./02-debug-exercises.md) use it for the most common TV bugs.

## The five steps

| Step | Question | Done when |
| --- | --- | --- |
| **1. Reproduce** | Can I make it happen when I want to? | You have written steps that trigger it every time (or "about 1 in N") |
| **2. Classify** | What kind of bug is it? | You can name one category: input, focus, Back, layout, performance, network, engine |
| **3. Collect** | What do I know about the device and the moment it broke? | Device, build, logs, and a video are attached to the ticket |
| **4. Compare** | Where does it work, and what is different there? | You have one working and one broken setup, and a short list of differences |
| **5. One change** | Does changing *this one thing* fix it? | The fix is confirmed on the device that failed, and nothing else changed |

Work in order. Most wasted time comes from jumping to step 5 before steps 1–4.

---

**How do you tell a layout bug from a focus bug?**

A focus bug means the wrong element has focus, or none does. A layout bug means the right element has focus but you can't see it properly. The quickest test is to press OK: if the item you expected opens, focus was correct and the problem is how it's drawn.