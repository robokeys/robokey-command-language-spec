# RoboKey Command Language (RKCL) Specification

Version 0.2 Draft

Author: Rob Deas, 2024

License: CC BY 4.0 license (spec only; reference implementation is GPL)

## Introduction

RoboKey Command Language (RKCL) is a universal, human- and machine-readable protocol for representing keyboard actions, macros, and control commands. RKCL is designed for use with programmable keyboards, automation agents, accessibility tools, testing harnesses, and virtual devices.

RKCL aims to be simple for humans to write and debug, but robust and structured enough for automation, AI, and integration with modern tools.

### 1. Command Formats

RKCL supports two command formats:



Text format (easy for humans, legacy code, and scripts)

JSONL format (newline-delimited JSON, best for automation and AI integration)

Commands in either format may be sent over serial, network, or other channels.

#### 1.1 Text Command Format

Each command is a single line of text.

Optional UUID prefix: id=<uuid>,

Command and parameter are separated by any of: :, -, _, ., ,, or space.

Case-insensitive, extra whitespace is ignored.

Examples:

HELPKEY:ENTER

id=abc123,COMBO:CTRL-ALT-DELCMD_SET_DELAY 75text:Hello, world!

### 1.2 JSONL Command Format

Each command is a valid JSON object, one per line (JSONL).

Use fields: command, parameter, and optionally uuid, timestamp, etc.

Recommended for agents, GUIs, bulk input, and robust communication.

Examples:

{"command": "combo", "parameter": "ctrl-alt-del", "uuid": "abc123"}{"command": "text", "parameter": "Hello, world!"}{"command": "key", "parameter": "f12"}

### 2. Command Normalization

Case-insensitive: KEY:ENTER == key-enter == Key Enter

Flexible delimiters: :, -, _, ., ,, and space all act as separators.

Parameter parsing: Everything after the first separator is the parameter.

Whitespace: Leading/trailing whitespace is ignored.

### 3. UUIDs: Correlating Requests and Responses

You can include a UUID for each command, which will always be returned in responses.

In text: use prefix id=<uuid>,

In JSONL: use field "uuid": "<uuid>".

If no UUID is supplied, the device or tool will generate one and include it in its responses.

Example:

id=12345,KEY:ENTER


{"command": "key", "parameter": "enter", "uuid": "12345"}

All responses related to a command will echo the same UUID for easy matching.

### 4. Command Reference

CommandParameter     Type                       Description
HELP                 none                       Print help text
TYPE_HELP            none                       Print help as keystrokes
PING                 none                       Device responsiveness test
STATUS               none                       Query device status
LOREM                none                       Type Lorem Ipsum text (one block)
LOREM_LINES          none                       Type Lorem Ipsum as four lines
TEXT                 string                     Type message as keystrokes
LINE                 string                     Type message plus newline
KEY                  key token                  Press/release a single key
COMBO                combo spec                 Press combination (e.g., CTRL-ALT-DEL)
EDIT                 cut/copy/paste/selectall   Perform edit action
HOLD                 key token                  Hold a key down
RELEASE_KEY          key token                  Release a specific key
RELEASE:ALL          none                       Release all held keys
PASSWORD             base64 string              Securely type decoded text
PRIVATE:TEXT         base64 string              Type decoded private text (not logged)see below...
Plus many "CMD:"-prefixed control commands



#### 4.1 Control/Administrative Commands (always start with CMD:)

CMD:OUTPUT:OFF / ON - Enable/disable keyboard output

CMD:SET:DELAY:<ms> - Set delay between keystrokes

CMD:SET:PRESS_LENGTH:<ms> - Set key press time

CMD:RESET - Reset device

CMD:ECHO:ON / OFF - Toggle serial echo

CMD:DEBUG:ON / OFF - Toggle debug messages

CMD:JITTER:ON / OFF - Enable/disable random timing jitter

CMD:KEY_JITTER:ON / OFF - Enable/disable key press jitter

CMD:DELAY_JITTER:ON / OFF - Enable/disable inter-key delay jitter

CMD:CONNECT, CMD:DISCONNECT, CMD:RECONNECT - Manage device state

CMD:STOP / CMD:PAUSE / CMD:RESUME - Control action state (these always work, even if busy)

### 5. Key and Combo Syntax

Special tokens: @C/CTRL, @S/SHIFT, @A/ALT, GUI, ALT_GR, etc.

Aliases: ENTER, RETURN, \N, \R = Enter/Return; BKSP, BS, \B = Backspace; \T = Tab; F1-F24 = Function keys.

Combos: e.g., COMBO:@C-@A-DEL or combo:ctrl,alt,del

Custom keys: Any token in the provided mapping.

### 6. Secure/Sensitive Commands

PASSWORD: and PRIVATE:TEXT: commands expect base64-encoded input.

Sensitive input is never logged, only the UUID is referenced in responses/logs.

### 7. Responses (Always JSONL)

All output is JSONL, regardless of input format.

Each line is a valid JSON object with at least:

level (INFO, ERROR, STATUS, etc.)

message

command (if available)

uuid (always present if supplied or generated)

plus any other relevant fields.

Examples:

{"level": "INFO", "message": "Command received: TEXT:Hello", "command": "TEXT", "parameter": "Hello", "uuid": "abc123"}{"level": "STATUS", "status": "connected", "ready": true, "paused": false, "uuid": "abc123"}{"level": "ERROR", "message": "Unknown key: QWERTY", "command": "KEY", "parameter": "QWERTY", "uuid": "abc123"}

### 8. Best Practices

Always supply a UUID if you want to reliably match responses.

Use JSONL for programmatic/agent integration.

Do not log or display parameters for sensitive commands.

Batch commands by sending multiple lines (text or JSONL).

### 9. Extensibility and Future-proofing

Unknown commands are ignored or result in a JSONL error response.

New commands should use descriptive names, optionally namespaced.

Future versions may add explicit keymap negotiation, Unicode input, multi-key macros, and more fields in JSONL.

### 10. Example Session

Text input:

id=abc123,combo:ctrl-alt-deltext:Hello, world!edit:copycmd:pausecmd:resumepassword:U2VjcmV0IQ==

Equivalent JSONL:

{"command": "combo", "parameter": "ctrl-alt-del", "uuid": "abc123"}{"command": "text", "parameter": "Hello, world!"}{"command": "edit", "parameter": "copy"}{"command": "cmd:pause"}{"command": "cmd:resume"}{"command": "password", "parameter": "U2VjcmV0IQ=="}

Sample responses:

{"level": "INFO", "command": "combo", "parameter": "ctrl-alt-del", "uuid": "abc123", "message": "Combo executed"}{"level": "INFO", "command": "text", "parameter": "Hello, world!", "message": "Typed text"}{"level": "INFO", "command": "edit", "parameter": "copy", "message": "Copy sent"}{"level": "STATUS", "paused": true, "message": "Device paused"}{"level": "INFO", "command": "password", "uuid": "auto-xyz", "message": "Sensitive information (password) typed (redacted)"}

### 11. Full Key Token List

(See your project’s key mapping array for the full, up-to-date set. Document here as needed for users.)

### 12. License

This specification is released under CC BY 4.0 license.

The reference implementation is licensed under the GNU General Public License v3 (or later).

13. Reference Implementation

See: RoboKeysDuino on GitHub for the canonical Arduino implementation.
