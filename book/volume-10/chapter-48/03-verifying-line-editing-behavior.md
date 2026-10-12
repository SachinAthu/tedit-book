# 48.3 Verifying Line-Editing Behavior

## Verify Document Boundaries

Run the existing tests after the Chapter 48 changes:

```
cmake --build build
ctest --test-dir build --output-on-failure
```

The current implementation passes both registered test targets: `smoke` and `editing`.

The line-editing tests must verify the following conditions:

- Splitting a line preserves every byte and its original order.
- Splitting at column zero creates an empty line before the original contents.
- Splitting at the end creates an empty line after the original contents.
- Merging lines preserves the text order and reduces the line count by one.
- Invalid operations leave the Document unchanged.
- Empty lines can be split and merged without losing the remaining document contents.

## Verify the Tests

Run:

```
cmake --build build
ctest --test-dir build --output-on-failure
```

Both registered test targets should pass. The new test is part of the existing `editing` target, so no CMake changes are required.

## Verify Interactive Behavior

Run the editor in a Linux or WSL terminal:

```
./build/tedit
```

Check these editing sequences manually:

1. Type `hello`, move the Cursor between `e` and `l`, and press Enter. Confirm that the text becomes `he` followed by `llo`.
2. Press Backspace at the beginning of the second line. Confirm that the lines merge into `hello` and the Cursor returns to the joining position.
3. Press Enter at the beginning and end of a line. Confirm that empty lines are created at the expected positions.
4. Press Backspace at the beginning of the first line. Confirm that the Document remains unchanged.
5. Press `Ctrl-Q` and confirm that the terminal returns to its normal state.

These checks validate the interactive input path as well as the Document operations. Automated tests alone do not confirm terminal key delivery or rendering behavior.

Chapter 48 verification status: the build and automated tests pass. Interactive terminal checks remain to be performed in the target environment. Vertical Cursor navigation, viewport adjustments, and file saving are outside this section's scope.
