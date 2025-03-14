
# b-code Specification Proposal

# Welcome

> **Hello, visitor! If you've come across this document, it likely holds your interest. We encourage you to read on.**

## Introduction
This document presents a proposal for a specification of a text-based protocol called b-code, designed to control small robots such as Otto, Zumo, or mBot.
b-code aims to serve DIY and toy robots in a similar manner to how G-code functions for 3D printers and CNC machines.

One of the objectives of b-code is permit the development of reusable remote control apps compatible with different robot models.

## Specification
### Generalities
"b-code" is the normalized name of the protocol. It is case-insensitive with a dash as separator.

b-code operates as a request-response protocol. The controller sends a **command** to the robot, which processes the command and returns a response.
The robot does not send any messages without a prior request from the controller.
The controller must wait for the robot's complete response before issuing another command.

Although the specification primarily utilizes ASCII, UTF-8 (without BOM) is the default encoding,
particularly for commands or responses that include human-readable messages.

### Syntax
#### Commands
The general format of a **command** consists of a single line of text structured as follows:
```
<command code> <arg1> <arg2> ... <argN>
```

The **command** line:
- begins with a **command code**, which is a string of ASCII letters and digits (no spaces or special characters).
- is followed by a list of **arguments**, separated by one or more spaces. **Arguments** may vary in **type** depending on the command.
- ends with a single newline character (`\n` or `0x0A`).

#### Responses

##### Response structure
There can be multiple **responses** to a command, depending on the command.
Each **response** is a single line of text, with the following structure:
```
<response type code> <complement1> <complement2> ... <complementN>
```
A **response** line :
* starts with a **response code**, which is a string of ASCII letters and digits (no spaces, no special characters),
* is followed by a list of **complements**, separated by a single or multiple spaces. **complements** can be of different **types** (depending on the response).
* is ended by a single newline character (`\n` or `0x0A`).

##### Terminating response line
A sequence of response lines is ended by a **terminating response**, with a response type code *OK* or *ERROR*.
A **terminating response** means that no more response will be sent by the robot for the current command, and the client can send another command.

The `OK` response type has no complement.

The `ERROR` response has the following structure :
```
ERROR <message> <code>
```

#####

### Movements

#### Unit of Movement
In the following commands, some command arguments  argument are described as a `<unit of movement>`.

The **unit of movement** is a `FLOAT` value representing the distance the robot should move.

**Units of movement** are used for two types of movements:
- translation, which is the distance the robot should move in a straight line
- rotation, which is the angle the robot should rotate

The **unit of movement** is a value that depends on the robot's hardware and software design. The **unit of movement** can represent various types of measurements, such as:
- a standardized unit, such as centimeters, millimeters or inches for a distance, degrees or radians for an angle
- a number of wheel turns for a wheeled robot, both sides for translation, one side for rotation
- the number of steps for a walking robot

> [!NOTE]
> As the **unit of movement** is a `FLOAT` value, it can be a partial value, such as `0.5` or `1.7`.
> It is up to the robot to determine how to handle partial values. It must **round them to the upper value it can handle**. For example:
> - for a robot with standardized units:
    >   - a value of `0.7` could be rounded to 1 cm wth a 1 cm precision for a distance
>   - a value of `1.7` could be rounded to 2 degrees with a 1 degree precision for an angle
> - for a wheeled robot with a 5 degrees precision, a value of `1.2` could be rounded to a 435 degree wheel rotation (1.2 * 360 / 5)
> - for a walking robot, a value of `O.1` could be rounded to 1 step
>
> Also, the **unit of movement** can be negative, positive or zero:
> - a positive value indicates movement in the requested direction.
> - a negative value indicates movement in the opposite direction.
> - a zero value indicates no movement.

#### Translation
**Command:** `M <direction> <unit of movement>` with
<direction> (type `CHARACTER`) is the direction of the movement. It can be one of the following:
- `F`: Forward
- `B`: Backward
- `L`: Left
- `R`: Right
  <unit of movement> (type `FLOAT`), see the **Unit of Movement** section above.

> [!NOTE]
> These directions imply the robot to have a front, a back, a left and a right side.
> If the robot cannot move in one of these directions, it should ignore the command.

**Response:** `M <direction> <unit>`

**Examples:**
>> `M F `
<< `OK`




#### Rotation

### Control commands

#### NOP
** Command:** `X`

#### End of transmission
**Command**
When receiving the
**Response**

#### Encoded commands

### Annexes

#### Generic errors



#### Types

##### Types specification
| Type      | Description                                                                                                                                                                                                                                                                                   | Examples                     |
|-----------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------|
| `TEXT`    | See below for the detailed description of this type                                                                                                                                                                                                                                           | `[This is a "TEXT" message]` |
| `INTEGER` | A signed integer number                                                                                                                                                                                                                                                                       | `90`, `-42`                  |
| `FLOAT`   | The `FLOAT` type corresponds to the `float` type in the C language, specifically a single-precision 32-bit IEEE 754 floating-point number. It can represent a wide range of values with a precision of approximately 6 to 7 decimal digits. Note that scientific notation is not supported.   | `5`, `3.14`, `-10.00`        |

##### TEXT type
The `TEXT` type is used for multiple words sequences in certain commands and responses (e.g., error messages).

The text value must always be surrounded by square brackets, even if it contains no space. The square brackets are not part of the text value.

If the text value contains square brackets, they must be escaped with a backslash. The backslash character must also be escaped with another backslash.

Encoding should be UTF-8 without BOM.

Example values:
- `[ThisIsASimpleTextMessageWithNoSpace]`
- `[This is a simple text message]`
- `[This is a text message with a \[quote\]]`
- `[This is a text message with a \\backslash]`
- `[This is a text message with a €uro symbol]`

## License
b-code © 2025 by Verdi8 is licensed under CC BY-SA 4.0. To view a copy of this license, visit https://creativecommons.org/licenses/by-sa/4.0/
