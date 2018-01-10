- Feature Name: Flipper Daemon
- Start Date: 2018-01-10
- RFC PR:
- Issue:

# Summary
[summary]: #summary

This RFCS proposes that a daemon be created to manage libflipper state that needs to be maintained across applications that are using a flipper device.

- Two applications cannot connect to the same flipper device at the same time since only one process can own the libusb connection to the device.
- If an application is forcably terminated, the connection to the device is never closed and could cause problems.
- The event system is currently not possible without having the user spin up a thread to handle the run loop needed to poll the attached flipper devices and dispatch events to an application.
- If you attach to a device using the console on the command line, the connection to the device is not maintained between commands.

A daemon would solve these issue.

# Motivation
[motivation]: #motivation

If a daemon is implemented, it would solve many of the global state problems that libflipper has. Libflipper can become a stateless perfectly reentrant and thread safe library. When global state needs to be accessed, such as the currently attached device, or the libusb endpoint to that device, an IPC message would be sent to the daemon to handle interacting with the stateful device, letting libflipper remain stateless.

# Detailed design
[design]: #detailed-design

Create an application that starts a run loop, polling incoming data from all of the attached devices in a loop. If an application that is using libflipper needs to interact with a device, it will send a message to the daemon, it will be processed by the event loop, and the result will be returned to the libflipper instance that requested it. I imagine that the connection between libflipper and the daemon would be the message runtime, and functions could be called the same way they are on the device. The message runtime is robust, and this is a great use case.

# How We Teach This
[how-we-teach-this]: #how-we-teach-this

This is a feature that would exist largely behind the scenes. Libflipper would continue to behave as it does currently with the added bonus that is is thread safe.

# Drawbacks
[drawbacks]: #drawbacks

It would require the first instance of libflipper to spawn a process, and every subsequent instance of libflipper to attach to that process.

It may not be cross platform for free. It should be on macOS and Linux, but Windows is the oddball per ususal.

# Alternatives
[alternatives]: #alternatives

Have a runloop per instance of libflipper, and instead of attaching to a global process, spawn a local thread. This would solve the event system issue, but require the user to call a blocking `lf_handle_events()` function and then transition their program to use event driven pub/sub techniques like iOS does.

# Unresolved questions
[unresolved]: #unresolved-questions

I can see a clear path to a simple implementation of this. Perhaps the transition to `libffi` should be made before we start performing function calls on the host, as that could be a possible security hole, especially if the daemon process is running with escalated privilages.
