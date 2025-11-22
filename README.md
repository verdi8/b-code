# `b-code` specification

## Current Version
Current version of the document is **`0.0.1-draft`**.

## Welcome

> **Hello, visitor! If you've come across this document, it likely holds your interest. We encourage you to read on.**

## Introduction
### Purpose of `b-code`
This document presents a proposal for a specification of a text-based protocol called `b-code`, designed to control small **robots** such as Otto, Zumo, or mBot.
`b-code` aims to serve DIY and toy **robots** in a similar manner to how G-code functions for 3D printers and CNC machines.

One of the objectives of `b-code` is to permit the development of reusable remote control apps compatible with different **robot** models.

Even though `b-code` is intended to cover a wide range of tiny **robots** capabilities, it is extensible to accommodate specific features of certain **robots**.

## Cheat Sheet

For a quick overview of the `b-code` commands, responses, and examples, see the [CHEATSHEET.md](CHEATSHEET.md) document. It provides a concise summary of the protocol's key elements for easy access during development.

## Related projects
### `b-code` descriptor
The `b-code` protocol is minimalistic, to be implemented on **robots** with limited sowftare resources like Arduino controllers.

For example, the command `A 10` could make a **robot** sing happy birthday while weaving its arms. It actually depends on the **robot** implementation.

That's why this protocol is completed by the specification of a descriptor file (in YAML format) which describes the capabilities of a specific **robot** model (catalogs of actions, units of movement, list of sensors, units, id card of the **robot**, etc.). 

The descriptor file specification has its own Github repository: https://github.com/verdi8/b-code-desc

### `b-code` web remote control app


## Specification
### Generalities
`b-code` is the normalized name of the protocol. It is lowercase with a hyphen.

The protocol is designed for communication between a **controller** (e.g., a computer, smartphone, or tablet) and a **robot** (the device being controlled).

### Robot concepts implied by the specification
The `b-code` specification is designed around several fundamental concepts and terms. These terms are defined below for a better understanding of the specification.

#### Robot
A **robot** is a physical device capable of moving and performing actions. It typically includes components such as motors, sensors, and actuators that enable it to interact with its environment.

#### Movements
A movement is a change in the robot's location or orientation, executed by the robot's locomotion system. 
Movements are quantified by a **unit of movement** and are categorized as:
- **Translation**: A **movement** that changes the **robot**'s location without altering its orientation (e.g. moving forward, backward, left, or right).
- **Rotation**: A **movement** that changes the **robot**'s orientation without altering its location (e.g. turning left or right).

A **movement** is also defined by a value called a **unit of movement**.

There are two types of **units of movement**: one for **translation** and another for **rotation**.

The **unit of movement** can represent various types of measurements, such as:
- a standardized unit, such as centimeters, millimeters or inches for a distance, degrees or radians for an angle
- a number of wheel turns for a wheeled **robot**, both sides for **translation**, one side for **rotation**
- the number of steps for a walking **robot**

The **unit of movement** is a value that depends on the **robot**'s hardware and software design. 

#### Actions
Actions are high-level operations that allow robots to interact with their environment or express themselves through a combination of gestures, sounds, and displays.

A **gesture** is a sequence of movements of parts of the **robot** that - in opposition to a **movement** - does not change the **robot**'s location or orientation. Examples include waving an arm, nodding a head, or blinking an eye.

**Sounds** are audio signals produced by the **robot**. Examples include beeps, melodies, or voice messages. **Sounds** are human-hearable.

**Displays** are visual outputs produced by the **robot**. Examples include LED matrixes, screen messages, or animations.

Gestures, sounds, and displays are considered primary actions, meaning they represent the basic building blocks of robot operations. However, the term **action** in the rest of this specification refers to either:
- a combination of these primary actions (e.g., a robot waving its arm while playing a melody and showing an animation), or
- a high-level operation that cannot be described as merely a gesture, sound, or display (e.g., a robot performing a complex task like "nod" or "greet").

**Actions** are used for purposes like expressing emotions, providing feedback, or interacting with the **robot**'s environment.

### General protocol rules
`b-code` operates as a request-response protocol. The **controller** sends a **command** to the **robot**, 
which processes the **command** and returns a **response**.

The **robot** does not send any messages without a prior request from the **controller**.

The **controller** must wait for the **robot**'s complete multiline **response** before issuing another **command**.

Although the specification primarily utilizes ASCII, UTF-8 (without BOM) is the default encoding for future extensions.

### Syntax
#### Commands
The general format of a **command** consists of a single line of text structured as follows:
```
<command code> <arg1> <arg2> ... <argN>
```

The **command** :
- begins with a **command code**, which is a sequence of ASCII uppercase letters and digits (no spaces, no special characters, a type named `[CODE]` later in this document),
- no spaces are allowed before the **command code**,
- is followed by a list of **arguments**, separated by one or more spaces. **Arguments** may vary in **type** depending on the **command**.
- ends with a single newline character (`\n` or `0x0A`).
- when more **arguments** are provided than required for a specific **command**, the extra **arguments** are ignored.

The maximum length of a **command** line is 64 characters, including the newline character.

**Note:** The types mentioned in the syntax (e.g., `[INTEGER]`, `[FLOAT]`, or `[CODE]`) are described in detail in the **Types** section at the end of this document.

#### Responses
##### Response structure
There can be multiple **responses** to a **command**, depending on the **command**.
Each **response** is a single line of text, with the following structure:
```
<response type code> <complement1> <complement2> ... <complementN>
```
A **response** line:
* starts with a **response code**, which is of type `[CODE]` (a sequence of ASCII letters and digits, no spaces, no special characters),
* is followed by a list of **complements**, separated by a single or multiple spaces. **Complements** can be of different **types** (depending on the **response**).
* is ended by a single newline character (`\n` or `0x0A`).

##### Terminating response line
A sequence of **response** lines is ended by a **terminating response**, with a **response type code** `OK` or `ERR`.
A **terminating response** means that no more **responses** will be sent by the **robot** for the current **command**, and the **controller** can send another **command**.

The `OK` **response type** has no complement.

The `ERR` **response** has the following structure:
```
ERR <error code>
```
where `<error code>` is an integer (type `[INTEGER]`) that identifies the specific error.

#### Sequence diagram

![Command-Response sequence diagram](http://www.plantuml.com/plantuml/proxy?cache=no&src=https://raw.githubusercontent.com/verdi8/b-code/refs/heads/feature/init/diagrams/command_response_sequence.puml)

#### Examples
In the follwing document, **commands** are represented by a line starting with `>>` and **responses** by lines starting with `<<`.

**Example 1: Successful command execution**
```
>> M F 10
<< OK
```
**Example 2: Command execution with an error**
```
>> M X 10
<< ERR 2
```

**Example 3: Command with multiple responses**
```
>> S TEMP
<< TEMP 25.3
<< OK
```

### Movements

As explained above, a **movement** is a change in the **robot**'s location or orientation.

#### Direction

The following section describes **commands** related to **movements** of the **robot** in a given direction.

These directions imply the **robot** to have a front, a back, a left and/or a right side. Even though not all **robots** may have these sides physically defined, they should be logically defined for the purpose of movement commands.

#### Unit of Movement
In the following **commands**, some **command arguments** are described as a `<unit of movement>`.

For a transltion, the **unit of movement** is a `[FLOAT]` value representing the distance the **robot** should move. But for a rotation, the **unit of movement** is representing the angle the **robot** should rotate.

> [!NOTE]
> As the **unit of movement** is a `[FLOAT]` value, it can be a partial value, such as `0.5` or `1.7`.
> It is up to the **robot** to determine how to handle partial values. It must **round them to the upper value it can handle**. For example:
> - for a **robot** with standardized units:
    >   - a value of `0.7` could be rounded to 1 cm wth a 1 cm precision for a distance
>   - a value of `1.7` could be rounded to 2 degrees with a 1 degree precision for an angle
> - for a wheeled **robot** with a 5 degrees precision, a value of `1.2` could be rounded to a 435 degree wheel rotation (1.2 * 360 / 5)
> - for a walking **robot**, a value of `O.1` could be rounded to 1 step
>
> Also, the **unit of movement** can be negative, positive or zero:
> - a positive value indicates **movement** in the requested direction.
> - a negative value indicates **movement** in the opposite direction.
> - a zero value indicates no **movement**.


#### Translation
A translation is a **movement** that changes the **robot**'s location without altering its orientation.

The distance of a translation is specified using a **unit of movement**.

**Command:** 

`M <direction> <unit of movement>` 

with
`<direction>` (type `[CODE]`) is the direction of the **movement**. It can be one of the following letters:
- `F`: Forward
- `B`: Backward
- `L`: Left
- `R`: Right
- `U`: Up
- `D`: Down

It can also be a combination of these letters, such as `FL` for Forward-Left or `BR` for Backward-Right.

These codes are predefined values for these specific directions. Other values are allowed, but it is up to the **robot** implementation to interpret them.

> [!NOTE]
> If the **robot** cannot move in one of these directions, it should ignore the **command**.

`<unit of movement>` (type `[FLOAT]`), see the **Unit of Movement** section above.

**Response:**

The only **response** to this **command** is a terminating `OK` or `ERR <error code>`.


**Examples:**

Move forward by 10 units of movement:
```
>> M F 10
<< OK
```

Move backward on the left by 3.5 units of movement:
```
>> M BL 5.5
<< OK
```

#### Rotation
A rotation is a **movement** that changes the **robot**'s orientation without altering its location.

The angle of a rotation is specified using a **unit of movement**.

**Command:**

`R <direction> <unit of movement>`
with
`<direction>` (type `[CODE]`) is the direction of the **movement**. It can be one of the following letters:
- `L`: Left
- `R`: Right
- `U`: Up
- `D`: Down

It can also be a combination of these letters, such as `UL` for Up-Left or `DR` for Down-Right.

These codes are predefined values for these specific directions. Other values are allowed, but it is up to the **robot** implementation to interpret them.

> [!NOTE]
> If the **robot** cannot rotate in one of these directions, it should ignore the **command**.

`<unit of movement>` (type `[FLOAT]`), see the **Unit of Movement** section above.

**Response:**

The only **response** to this **command** is a terminating `OK` or `ERR <error code>`.

**Examples:**

Rotate left by 90 units of movement:
```
>> R L 90
<< OK
```
Rotate up-right by 45.5 units of movement:
```
>> R UR 45.5
<< OK
```

### Gestures, sounds, displays and actions

Gestures, sounds, displays and action are concepts described above in the **Robot concepts implied by the specification** section.

#### Gestures

**Command:** 

`G <gesture code>` with
`<gesture code>` (type `[INTEGER]`) is an integer that identifies the specific gesture to be performed by the **robot**.

It is up to the **robot** implementation to define the mapping between gesture codes and actual gestures.

> [!NOTE]
> The gesture code `0` is usually reserved for stopping any ongoing gesture and returning the **robot** to a neutral position.

**Response:**

A terminating `OK` or `ERR <error code>`.

#### Sounds

**Command:**

`S <sound code> [<sound volume>]` with
`<sound code>` (type `[INTEGER]`) is an integer that identifies the specific sound to be played by the **robot**.

> [!NOTE]
> The sound code `0` is usually reserved for stopping any ongoing sound.

`<sound volume>` (type `[FLOAT]`, optional) is a floating-point value representing the volume at which the sound should be played. It typically ranges from `0.0` (mute) to `1.0` (maximum volume). If not provided, the default volume level is used. If provided but not supported by the **robot**, it should be ignored. Out of range values should be clamped to the nearest valid value (e.g., a value of `1.5` should be treated as `1.0`).

**Response:**

A terminating `OK` or `ERR <error code>`.

#### Displays

**Command:**

`D <display code> [<display number>] ` with
`<display code>` (type `[INTEGER]`) is an integer that identifies the specific display to be shown by the **robot**.

> [!NOTE]
> The display code 0 is usually reserved for turning off the display.

`<display number>` (type `[INTEGER]`, optional) is an integer that can be used to identify a specific instance of the display, if the **robot** supports multiple displays. If not provided, the default display instance is used (conventionally `0`).

**Response:**

A terminating `OK` or `ERR <error code>`.

#### Actions

**Command:**

`A <action code>` with
`<action code>` (type `[INTEGER]`) is an integer that identifies the specific composite action to be performed by the **robot**.

**Specific action:**

`A 0` is a reserved action code that makes the **robot** stop all its ongoing actions immediately and return to a neutral state. It is an equivalent a `0` command for gestures, sounds, and displays.

**Response:**

A terminating `OK` or `ERR <error code>`.

### Special commands

#### Stop

The stop **command** is used to immediately halt all ongoing actions and movements of the **robot**.

It is a safety feature that allows the **controller** to quickly stop the **robot** in case of an emergency or unexpected behavior.

**Command:**

`0`

**Response:**

A terminating `OK`.

#### NOP
The NOP (No Operation) **command** is used to check or maintain the communication between the **controller** and the **robot** without performing any action.

**Command:** 

`Z`

**Response:**

A terminating `OK`.


### Appendices

#### Errors
##### Error ranges

Error codes are strictly positive integers (greater than or equal to `1`).
Error codes are organized into ranges based on their categories:

| Error code range | Description                        |
|------------------|------------------------------------|
| `0`              | Forbidden value. Error codes must be strictly positive integers. |
| `1` to `99`      | Parsing errors. These errors imply that the **command** could not be understood and could not be submitted to the **robot**'s logic. |
| `100` to `199`   | Action errors. These errors relate to issues encountered while executing **actions**. |



##### Generic errors

| Error code | Message                          | Description                      |
|------------|----------------------------------|----------------------------------|
| `1`        | Parsing error                    | The command could not simply be understood. |
| `2`        | Invalid command code             | The command code is not a supported value. |

#### Types

##### Types specification
| Type          | Description                                                                                                                                                                                                                                                                                   | Examples                     |
|---------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------|
| `[CHARACTER]` | The `[CHARACTER]` type represents a single ASCII character. It is used to denote individual characters in commands or responses.                                                                                                                                                              | `A`, `Z`, `0`, `9`, `!`     |
| `[CODE]`      | The `[CODE]` type represents a string of ASCII letters and digits (no spaces, no special characters). A code has a maximum length of 16 characters.                                                                                                                                           | `ERROR`, `SENSOR1`          |
| `[INTEGER]`   | A signed integer type that can represent whole numbers, both positive and negative. An integer is a 16-bit value, ranging from -32,768 to 32,767. Error codes are a specific use case of `[INTEGER]`.                                                                                              | `0`, `42`, `-7`             |
| `[FLOAT]`     | The `[FLOAT]` type corresponds to the `float` type in the C language, specifically a single-precision 32-bit IEEE 754 floating-point number. It can represent a wide range of values with a precision of approximately 6 to 7 decimal digits. Note that scientific notation is not supported.   | `5`, `3.14`, `-10.00`       |


### Versioning
#### Versioning scheme
The versioning of the `b-code` specification follows the Semantic Versioning principles (SemVer 2.0.0).

#### Branching model
The `b-code` specification repository follows the following branching model:
- The `main` branch contains the latest stable release of the specification.
- The `draft` branch is used for ongoing development and integration of evolutions to the specification.

### Contribution
Contributions to the `b-code` specification are welcome! If you have proposals for improvements, feel free to submit an issue on the GitHub repository.

