# SuperCollider style guidelines

When writing SuperCollider code for this series of tutorials, please follow the conventions described below. This is in order to make the coding style and syntax in the examples consistent, making them simpler for new users to follow.

The conventions are not intended to imply that one style or syntax is preferable to another. In some cases, the decision was fairly arbitrary, the goal being overall consistency.

## Use of interpreter variables
Do not use the interpreter variables `a` to `z`. Use a descriptive variable name preceded by a tilde (`~`) instead:

Don't do this:

```
h = 0.5;
```
Please do this instead:
```
~half = 0.5
```

Note that even though `s` is pre-assigned to `Server.default`, we would prefer `Server.default`.

## Assignments
Avoid the underscore syntax when making assignments, and use `=` instead:

Don't do this:
```
~midiOut.latency_(0.2);
```
Please do this instead:
```
~midiOut.latency = 0.2;
```