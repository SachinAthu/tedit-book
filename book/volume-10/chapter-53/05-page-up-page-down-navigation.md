# 53.5 Page Up and Page Down Navigation

## Goal

Connect Page Up and Page Down terminal events to document navigation. A page is the number of document rows currently visible, not a hard-coded line count or the full terminal height.

## Changed files

- `src/cursor/cursor.h` — public Page Up/Page Down operations.
- `src/cursor/cursor.c` — page-sized vertical movement with boundary clamping and preferred-column preservation.
- `src/command/command_engine.h` — page navigation commands and `page_rows` metadata.
- `src/input/input_engine.c` — map Page Up/Page Down events to commands.
- `src/command/command_engine.c` — execute page navigation commands.
- `src/main.c` — pass the Renderer’s visible document height to each command.
- `tests/test_input_engine.c` — verify event-to-command mapping.
- `tests/test_command_engine.c` — verify page movement, boundaries, preferred-column recovery, and Dirty Flag behavior.
- `tests/test_input_pipeline.c` — verify Page Up/Page Down terminal sequences reach navigation behavior.

## Complete updated source

### `src/cursor/cursor.h`

```c
#ifndef CURSOR_CURSOR_H
#define CURSOR_CURSOR_H

#include "document/document.h"

#include <stdbool.h>
#include <stddef.h>

typedef struct
{
    size_t row;
    size_t column;
    size_t preferred_column;
} Cursor;

void cursor_init(Cursor *cursor);

bool cursor_set_position(Cursor *cursor, const Document *document, size_t row, size_t column);

bool cursor_advance(Cursor *cursor, const Document *document);

bool cursor_move_left(Cursor *cursor);

bool cursor_move_right(Cursor *cursor, const Document *document);

bool cursor_move_home(Cursor *cursor);

bool cursor_move_end(Cursor *cursor, const Document *document);

bool cursor_move_up(Cursor *cursor, const Document *document);

bool cursor_move_page_up(Cursor *cursor, const Document *document, size_t page_rows);

bool cursor_move_page_down(Cursor *cursor, const Document *document, size_t page_rows);

bool cursor_move_down(Cursor *cursor, const Document *document);

#endif
```

### `src/cursor/cursor.c`

```c
#include "cursor.h"

void cursor_init(Cursor *cursor)
{
    if (cursor == NULL)
    {
        return;
    }

    cursor->row = 0;
    cursor->column = 0;
    cursor->preferred_column = 0;
}

bool cursor_set_position(Cursor *cursor, const Document *document, size_t row, size_t column)
{
    if (cursor == NULL || document == NULL || row >= document->line_count)
    {
        return false;
    }

    const DocumentLine *line = &document->lines[row];

    if (column > line->length)
    {
        return false;
    }

    cursor->row = row;
    cursor->column = column;
    cursor->preferred_column = column;

    return true;
}

bool cursor_advance(Cursor *cursor, const Document *document)
{
    const DocumentLine *line;

    if (cursor == NULL || document == NULL ||
        cursor->row >= document->line_count)
    {
        return false;
    }

    line = &document->lines[cursor->row];

    if (cursor->column >= line->length)
    {
        return false;
    }

    cursor->column += 1;
    cursor->preferred_column = cursor->column;

    return true;
}

bool cursor_move_left(Cursor *cursor)
{
    if (cursor == NULL || cursor->column == 0)
    {
        return false;
    }

    cursor->column -= 1;
    cursor->preferred_column = cursor->column;
    return true;
}

bool cursor_move_right(Cursor *cursor, const Document *document)
{
    const DocumentLine *line;

    if (cursor == NULL || document == NULL || cursor->row >= document->line_count)
    {
        return false;
    }

    line = &document->lines[cursor->row];
    if (cursor->column >= line->length)
    {
        return false;
    }

    cursor->column += 1;
    cursor->preferred_column = cursor->column;
    return true;
}

bool cursor_move_home(Cursor *cursor)
{
    if (cursor == NULL || cursor->column == 0)
    {
        return false;
    }

    cursor->column = 0;
    cursor->preferred_column = 0;
    return true;
}

bool cursor_move_end(Cursor *cursor, const Document *document)
{
    const DocumentLine *line;

    if (cursor == NULL || document == NULL || cursor->row >= document->line_count)
    {
        return false;
    }

    line = &document->lines[cursor->row];
    if (cursor->column == line->length)
    {
        return false;
    }

    cursor->column = line->length;
    cursor->preferred_column = cursor->column;
    return true;
}

static bool cursor_move_vertical(Cursor *cursor, const Document *document,
                                 size_t target_row)
{
    const DocumentLine *target_line;

    if (cursor == NULL || document == NULL ||
        target_row >= document->line_count)
    {
        return false;
    }

    target_line = &document->lines[target_row];
    cursor->row = target_row;
    cursor->column = cursor->preferred_column;

    if (cursor->column > target_line->length)
    {
        cursor->column = target_line->length;
    }

    return true;
}

bool cursor_move_up(Cursor *cursor, const Document *document)
{
    if (cursor == NULL || document == NULL || cursor->row == 0 ||
        cursor->row >= document->line_count)
    {
        return false;
    }

    return cursor_move_vertical(cursor, document, cursor->row - 1);
}

bool cursor_move_down(Cursor *cursor, const Document *document)
{
    if (cursor == NULL || document == NULL ||
        cursor->row >= document->line_count ||
        cursor->row + 1 >= document->line_count)
    {
        return false;
    }

    return cursor_move_vertical(cursor, document, cursor->row + 1);
}

bool cursor_move_page_up(Cursor *cursor, const Document *document, size_t page_rows)
{
    size_t target_row;

    if (cursor == NULL || document == NULL || page_rows == 0 ||
        cursor->row >= document->line_count || cursor->row == 0)
    {
        return false;
    }

    target_row = cursor->row > page_rows ? cursor->row - page_rows : 0;
    if (target_row == cursor->row)
    {
        return false;
    }

    return cursor_move_vertical(cursor, document, target_row);
}

bool cursor_move_page_down(Cursor *cursor, const Document *document, size_t page_rows)
{
    size_t remaining_rows;
    size_t target_row;

    if (cursor == NULL || document == NULL || page_rows == 0 ||
        cursor->row >= document->line_count ||
        cursor->row + 1 >= document->line_count)
    {
        return false;
    }

    remaining_rows = document->line_count - 1 - cursor->row;
    if (page_rows >= remaining_rows)
    {
        target_row = document->line_count - 1;
    }
    else
    {
        target_row = cursor->row + page_rows;
    }

    return cursor_move_vertical(cursor, document, target_row);
}
```

### `src/input/input_engine.c`

```c
#include "input/input_engine.h"

static void command_set(EditorCommand *command, EditorCommandType type, uint32_t character)
{
    command->type = type;
    command->character = character;
    command->page_rows = 0;
}

bool input_engine_translate(const InputEvent *event, EditorCommand *command)
{
    if (event == NULL || command == NULL)
    {
        return false;
    }

    command_set(command, EDITOR_COMMAND_NONE, 0);

    switch (event->type)
    {
        case INPUT_EVENT_NONE:
            break;
        case INPUT_EVENT_CHARACTER:
            command_set(command, EDITOR_COMMAND_INSERT_CHARACTER, event->character);
            break;
        case INPUT_EVENT_CTRL_Q:
            command_set(command, EDITOR_COMMAND_QUIT, 0);
            break;
        case INPUT_EVENT_CTRL_S:
            command_set(command, EDITOR_COMMAND_SAVE, 0);
            break;
        case INPUT_EVENT_BACKSPACE:
            command_set(command, EDITOR_COMMAND_BACKSPACE, 0);
            break;
        case INPUT_EVENT_ENTER:
            command_set(command, EDITOR_COMMAND_INSERT_NEWLINE, 0);
            break;
        case INPUT_EVENT_ESCAPE:
            command_set(command, EDITOR_COMMAND_CANCEL, 0);
            break;
        case INPUT_EVENT_ARROW_LEFT:
            command_set(command, EDITOR_COMMAND_MOVE_LEFT, 0);
            break;
        case INPUT_EVENT_ARROW_RIGHT:
            command_set(command, EDITOR_COMMAND_MOVE_RIGHT, 0);
            break;
        case INPUT_EVENT_ARROW_UP:
            command_set(command, EDITOR_COMMAND_MOVE_UP, 0);
            break;
        case INPUT_EVENT_ARROW_DOWN:
            command_set(command, EDITOR_COMMAND_MOVE_DOWN, 0);
            break;
        case INPUT_EVENT_HOME:
            command_set(command, EDITOR_COMMAND_MOVE_HOME, 0);
            break;
        case INPUT_EVENT_END:
            command_set(command, EDITOR_COMMAND_MOVE_END, 0);
            break;
        case INPUT_EVENT_PAGE_UP:
            command_set(command, EDITOR_COMMAND_PAGE_UP, 0);
            break;
        case INPUT_EVENT_PAGE_DOWN:
            command_set(command, EDITOR_COMMAND_PAGE_DOWN, 0);
            break;
        case INPUT_EVENT_TAB:
            break;
        case INPUT_EVENT_DELETE:
            command_set(command, EDITOR_COMMAND_DELETE_CHARACTER, 0);
            break;
        default:
            return false;
    }

    return true;
}
```

### `src/command/command_engine.h`

```c
#ifndef COMMAND_COMMAND_ENGINE_H
#define COMMAND_COMMAND_ENGINE_H

#include <stdbool.h>
#include <stdint.h>

#include "state/editor_state.h"

typedef enum
{
    EDITOR_COMMAND_NONE = 0,
    EDITOR_COMMAND_INSERT_CHARACTER,
    EDITOR_COMMAND_BACKSPACE,
    EDITOR_COMMAND_INSERT_NEWLINE,
    EDITOR_COMMAND_CANCEL,
    EDITOR_COMMAND_MOVE_LEFT,
    EDITOR_COMMAND_MOVE_RIGHT,
    EDITOR_COMMAND_MOVE_UP,
    EDITOR_COMMAND_MOVE_DOWN,
    EDITOR_COMMAND_PAGE_UP,
    EDITOR_COMMAND_PAGE_DOWN,
    EDITOR_COMMAND_MOVE_HOME,
    EDITOR_COMMAND_MOVE_END,
    EDITOR_COMMAND_DELETE_CHARACTER,
    EDITOR_COMMAND_SAVE,
    EDITOR_COMMAND_QUIT
} EditorCommandType;

typedef struct
{
    EditorCommandType type;
    uint32_t character;
    size_t page_rows;
} EditorCommand;

bool command_engine_execute(EditorState *state,
                            const EditorCommand *command,
                            const char *save_path,
                            bool *redraw,
                            bool *should_quit);

#endif
```

### `src/command/command_engine.c`

```c
#include "command/command_engine.h"

#include "cursor/cursor.h"
#include "document/document.h"
#include "state/editor_state.h"

#include <string.h>

static bool command_backspace(EditorState *state, bool *changed)
{
    if (state->cursor.column > 0)
    {
        const size_t delete_column = state->cursor.column - 1;
        if (!document_delete_character(&state->document, state->cursor.row, delete_column))
        {
            return true;
        }
        if (!cursor_set_position(&state->cursor, &state->document,
                                 state->cursor.row, delete_column))
        {
            return false;
        }
    }
    else if (state->cursor.row > 0)
    {
        const size_t previous_row = state->cursor.row - 1;
        const DocumentLine *previous_line = document_get_line(&state->document, previous_row);
        if (previous_line == NULL)
        {
            return true;
        }

        const size_t previous_length = previous_line->length;
        if (!document_merge_lines(&state->document, previous_row))
        {
            return true;
        }
        if (!cursor_set_position(&state->cursor, &state->document, previous_row, previous_length))
        {
            return false;
        }
    }
    else
    {
        return true;
    }

    state->is_dirty = true;
    *changed = true;
    return true;
}

static bool command_delete_character(EditorState *state, bool *changed)
{
    const DocumentLine *line = document_get_line(&state->document, state->cursor.row);

    if (line == NULL)
    {
        return true;
    }

    if (state->cursor.column < line->length)
    {
        if (!document_delete_character(&state->document, state->cursor.row,
                                       state->cursor.column))
        {
            return true;
        }
    }
    else if (state->cursor.row + 1 < state->document.line_count)
    {
        if (!document_merge_lines(&state->document, state->cursor.row))
        {
            return true;
        }
    }
    else
    {
        return true;
    }

    if (!cursor_set_position(&state->cursor, &state->document,
                             state->cursor.row, state->cursor.column))
    {
        return false;
    }

    state->is_dirty = true;
    *changed = true;
    return true;
}

static size_t encode_utf8(uint32_t character, unsigned char bytes[4])
{
    if (character <= 0x7F)
    {
        bytes[0] = (unsigned char)character;
        return 1;
    }
    if (character <= 0x7FF)
    {
        bytes[0] = (unsigned char)(0xC0 | (character >> 6));
        bytes[1] = (unsigned char)(0x80 | (character & 0x3F));
        return 2;
    }
    if (character <= 0xFFFF && !(character >= 0xD800 && character <= 0xDFFF))
    {
        bytes[0] = (unsigned char)(0xE0 | (character >> 12));
        bytes[1] = (unsigned char)(0x80 | ((character >> 6) & 0x3F));
        bytes[2] = (unsigned char)(0x80 | (character & 0x3F));
        return 3;
    }
    if (character <= 0x10FFFF)
    {
        bytes[0] = (unsigned char)(0xF0 | (character >> 18));
        bytes[1] = (unsigned char)(0x80 | ((character >> 12) & 0x3F));
        bytes[2] = (unsigned char)(0x80 | ((character >> 6) & 0x3F));
        bytes[3] = (unsigned char)(0x80 | (character & 0x3F));
        return 4;
    }
    return 0;
}

static size_t filename_previous_boundary(const char *text, size_t position)
{
    if (position == 0)
    {
        return 0;
    }

    --position;
    while (position > 0 && (((unsigned char)text[position] & 0xC0) == 0x80))
    {
        --position;
    }
    return position;
}

static size_t filename_next_boundary(const char *text, size_t length, size_t position)
{
    if (position >= length)
    {
        return length;
    }

    ++position;
    while (position < length && (((unsigned char)text[position] & 0xC0) == 0x80))
    {
        ++position;
    }
    return position;
}

static void filename_entry_clear(EditorState *state)
{
    state->filename_entry_active = false;
    state->filename_entry_length = 0;
    state->filename_entry_cursor = 0;
    state->filename_entry[0] = '\0';
}

static void filename_entry_start(EditorState *state, bool quit_after_save)
{
    state->filename_entry_active = true;
    state->filename_entry_length = 0;
    state->filename_entry_cursor = 0;
    state->filename_entry[0] = '\0';
    state->quit_after_save = quit_after_save;
    (void)editor_state_set_message(state, "Enter: save  Esc: cancel");
}

static bool command_save(EditorState *state, const char *save_path, bool quitting, bool *redraw, bool *should_quit)
{
    const char *current_path = state->file_path != NULL ? state->file_path : save_path;

    if (current_path == NULL)
    {
        filename_entry_start(state, quitting);
        *redraw = true;
        return true;
    }

    if (editor_state_save_file(state, current_path))
    {
        (void)editor_state_set_message(state, "Saved successfully.");
        state->quit_after_save = false;
        if (quitting)
        {
            *should_quit = true;
        }
    }
    else
    {
        (void)editor_state_set_message(state, "Save failed; document remains modified.");
        state->quit_after_save = false;
    }

    *redraw = true;
    return true;
}

static bool filename_entry_execute(EditorState *state, const EditorCommand *command, bool *redraw, bool *should_quit)
{
    if (command->type == EDITOR_COMMAND_CANCEL)
    {
        const bool was_quitting = state->quit_after_save;
        filename_entry_clear(state);
        state->quit_after_save = false;
        (void)editor_state_set_message(state, was_quitting ? "Quit cancelled." : "Save As cancelled.");
        *redraw = true;
        return true;
    }

    if (command->type == EDITOR_COMMAND_MOVE_LEFT)
    {
        const size_t previous = filename_previous_boundary(state->filename_entry,
                                                           state->filename_entry_cursor);
        if (previous != state->filename_entry_cursor)
        {
            state->filename_entry_cursor = previous;
            *redraw = true;
        }
        return true;
    }

    if (command->type == EDITOR_COMMAND_MOVE_RIGHT)
    {
        const size_t next = filename_next_boundary(state->filename_entry,
                                                   state->filename_entry_length,
                                                   state->filename_entry_cursor);
        if (next != state->filename_entry_cursor)
        {
            state->filename_entry_cursor = next;
            *redraw = true;
        }
        return true;
    }

    if (command->type == EDITOR_COMMAND_BACKSPACE)
    {
        if (state->filename_entry_cursor > 0)
        {
            const size_t previous = filename_previous_boundary(state->filename_entry, state->filename_entry_cursor);
            const size_t removed = state->filename_entry_cursor - previous;
            memmove(state->filename_entry + previous,
                    state->filename_entry + state->filename_entry_cursor,
                    state->filename_entry_length - state->filename_entry_cursor + 1);
            state->filename_entry_length -= removed;
            state->filename_entry_cursor = previous;
            *redraw = true;
        }
        return true;
    }

    if (command->type == EDITOR_COMMAND_DELETE_CHARACTER)
    {
        if (state->filename_entry_cursor < state->filename_entry_length)
        {
            const size_t next = filename_next_boundary(state->filename_entry,
                                                       state->filename_entry_length,
                                                       state->filename_entry_cursor);
            const size_t removed = next - state->filename_entry_cursor;
            memmove(state->filename_entry + state->filename_entry_cursor,
                    state->filename_entry + next,
                    state->filename_entry_length - next + 1);
            state->filename_entry_length -= removed;
            *redraw = true;
        }
        return true;
    }

    if (command->type == EDITOR_COMMAND_INSERT_CHARACTER)
    {
        unsigned char encoded[4];
        const size_t encoded_length = encode_utf8(command->character, encoded);

        if (encoded_length == 0)
        {
            return true;
        }
        if (encoded_length > EDITOR_FILENAME_CAPACITY - 1 - state->filename_entry_length)
        {
            (void)editor_state_set_message(state, "Filename is too long.");
            *redraw = true;
            return true;
        }

        memmove(state->filename_entry + state->filename_entry_cursor + encoded_length,
                state->filename_entry + state->filename_entry_cursor,
                state->filename_entry_length - state->filename_entry_cursor + 1);
        memcpy(state->filename_entry + state->filename_entry_cursor, encoded, encoded_length);
        state->filename_entry_length += encoded_length;
        state->filename_entry_cursor += encoded_length;
        (void)editor_state_set_message(state, "Enter: save  Esc: cancel");
        *redraw = true;
        return true;
    }

    if (command->type == EDITOR_COMMAND_INSERT_NEWLINE)
    {
        if (state->filename_entry_length == 0)
        {
            (void)editor_state_set_message(state, "Enter a file name.");
            *redraw = true;
            return true;
        }

        if (editor_state_save_as(state, state->filename_entry))
        {
            const bool should_quit_after_save = state->quit_after_save;
            filename_entry_clear(state);
            state->quit_after_save = false;
            (void)editor_state_set_message(state, "Saved successfully.");
            if (should_quit_after_save)
            {
                *should_quit = true;
            }
        }
        else
        {
            (void)editor_state_set_message(
                state, "Save failed; check path and try again, or Esc to cancel.");
        }
        *redraw = true;
        return true;
    }

    /* Ignore commands not meaningful while a filename is being entered. */
    return true;
}

static bool quit_confirmation_execute(EditorState *state, const EditorCommand *command, const char *save_path, bool *redraw, bool *should_quit)
{
    if (command->type == EDITOR_COMMAND_CANCEL)
    {
        state->quit_confirmation_active = false;
        (void)editor_state_set_message(state, "Quit cancelled.");
        *redraw = true;
        return true;
    }

    if (command->type != EDITOR_COMMAND_INSERT_CHARACTER)
    {
        return true;
    }

    if (command->character == 'n' || command->character == 'N')
    {
        state->quit_confirmation_active = false;
        state->quit_after_save = false;
        *should_quit = true;
        return true;
    }

    if (command->character == 'y' || command->character == 'Y')
    {
        state->quit_confirmation_active = false;
        return command_save(state, save_path, true, redraw, should_quit);
    }

    return true;
}

bool command_engine_execute(EditorState *state, const EditorCommand *command, const char *save_path, bool *redraw, bool *should_quit)
{
    bool changed = false;

    if (state == NULL || command == NULL || redraw == NULL || should_quit == NULL)
    {
        return false;
    }

    *redraw = false;
    *should_quit = false;

    if (state->filename_entry_active)
    {
        return filename_entry_execute(state, command, redraw, should_quit);
    }

    if (state->quit_confirmation_active)
    {
        return quit_confirmation_execute(state, command, save_path, redraw, should_quit);
    }

    switch (command->type)
    {
    case EDITOR_COMMAND_NONE:
        break;
    case EDITOR_COMMAND_QUIT:
        if (state->is_dirty)
        {
            state->quit_confirmation_active = true;
            state->quit_after_save = false;
            (void)editor_state_set_message(state, "Save changes before quitting?");
            *redraw = true;
        }
        else
        {
            *should_quit = true;
        }
        break;
    case EDITOR_COMMAND_SAVE:
        return command_save(state, save_path, false, redraw, should_quit);
    case EDITOR_COMMAND_CANCEL:
        break;
    case EDITOR_COMMAND_INSERT_CHARACTER:
        if (!document_insert_character(&state->document, state->cursor.row, state->cursor.column, command->character))
        {
            return true;
        }
        if (!cursor_advance(&state->cursor, &state->document))
        {
            return false;
        }
        state->is_dirty = true;
        changed = true;
        break;
    case EDITOR_COMMAND_BACKSPACE:
        if (!command_backspace(state, &changed))
        {
            return false;
        }
        break;
    case EDITOR_COMMAND_INSERT_NEWLINE:
        if (!document_split_line(&state->document, state->cursor.row, state->cursor.column))
        {
            return true;
        }
        if (!cursor_set_position(&state->cursor, &state->document, state->cursor.row + 1, 0))
        {
            return false;
        }
        state->is_dirty = true;
        changed = true;
        break;
    case EDITOR_COMMAND_MOVE_LEFT:
        changed = cursor_move_left(&state->cursor);
        break;
    case EDITOR_COMMAND_MOVE_RIGHT:
        changed = cursor_move_right(&state->cursor, &state->document);
        break;
    case EDITOR_COMMAND_MOVE_UP:
        changed = cursor_move_up(&state->cursor, &state->document);
        break;
    case EDITOR_COMMAND_MOVE_DOWN:
        changed = cursor_move_down(&state->cursor, &state->document);
        break;
    case EDITOR_COMMAND_PAGE_UP:
        changed = cursor_move_page_up(&state->cursor, &state->document, command->page_rows);
        break;
    case EDITOR_COMMAND_PAGE_DOWN:
        changed = cursor_move_page_down(&state->cursor, &state->document, command->page_rows);
        break;
    case EDITOR_COMMAND_MOVE_HOME:
        changed = cursor_move_home(&state->cursor);
        break;
    case EDITOR_COMMAND_MOVE_END:
        changed = cursor_move_end(&state->cursor, &state->document);
        break;
    case EDITOR_COMMAND_DELETE_CHARACTER:
        if (!command_delete_character(state, &changed))
        {
            return false;
        }
        break;
    default:
        return false;
    }

    *redraw = *redraw || changed;
    return true;
}
```

### `src/main.c`

```c
#include "command/command_engine.h"
#include "cursor/cursor.h"
#include "document/document.h"
#include "input/input_engine.h"
#include "input/input_parser.h"
#include "renderer/renderer.h"
#include "state/editor_state.h"
#include "terminal/terminal.h"

#include <errno.h>
#include <stdbool.h>
#include <stdio.h>
#include <sys/select.h>
#include <unistd.h>

#define ESCAPE_TIMEOUT_MICROSECONDS 50000

static int read_escape_byte(unsigned char *byte)
{
    fd_set read_set;
    struct timeval timeout;
    int result;

    FD_ZERO(&read_set);
    FD_SET(STDIN_FILENO, &read_set);
    timeout.tv_sec = 0;
    timeout.tv_usec = ESCAPE_TIMEOUT_MICROSECONDS;

    do
    {
        result = select(STDIN_FILENO + 1, &read_set, NULL, NULL, &timeout);
    } while (result == -1 && errno == EINTR);

    if (result <= 0)
    {
        return result;
    }

    for (;;)
    {
        const ssize_t bytes_read = read(STDIN_FILENO, byte, sizeof(*byte));

        if (bytes_read == 1)
        {
            return 1;
        }
        if (bytes_read == 0)
        {
            return 0;
        }
        if (errno != EINTR)
        {
            return -1;
        }
    }
}

static bool redraw_after_action(Renderer *renderer, EditorState *state)
{
    if (renderer == NULL || state == NULL)
    {
        return false;
    }

    if (!viewport_ensure_cursor_visible(
            &state->viewport,
            state->cursor.row,
            state->cursor.column,
            renderer->document_rows,
            renderer->screen_buffer.columns))
    {
        return false;
    }

    return renderer_render(renderer, state);
}

static bool application_event_loop(EditorState *state, Renderer *renderer, const char *save_path)
{
    InputParser parser;
    input_parser_init(&parser);

    while (true)
    {
        unsigned char byte;
        int read_result;
        InputEvent event;
        bool should_quit;
        bool event_ready = false;

        /*
         * While the parser is in the middle of an Escape sequence, the next
         * byte must arrive quickly or not at all: read with a timeout so a
         * lone ESC press is handled promptly. Otherwise a plain blocking
         * read is enough.
         */
        if (input_parser_is_waiting(&parser))
        {
            read_result = read_escape_byte(&byte);

            if (read_result == 0)
            {
                if (!input_parser_timeout(&parser, &event))
                {
                    return false;
                }

                if (event.type == INPUT_EVENT_NONE)
                {
                    continue;
                }

                event_ready = true;
                read_result = 1;
            }
        }
        else
        {
            const ssize_t bytes_read = read(STDIN_FILENO, &byte, sizeof(byte));

            if (bytes_read < 0)
            {
                if (errno == EINTR)
                {
                    continue;
                }
                return false;
            }

            if (bytes_read == 0)
            {
                return true;
            }
            read_result = 1;
        }

        if (read_result < 0)
        {
            return false;
        }
        if (read_result == 0)
        {
            return true;
        }

        /*
         * Feed the byte into the parser; it returns true once a complete key
         * event is ready (or when a timeout already produced one above).
         */
        if (!event_ready && !input_parser_feed(&parser, byte, &event))
        {
            return false;
        }

        EditorCommand command;
        bool redraw;

        /* Pipeline step by step: event -> command -> state change -> redraw. */
        if (!input_engine_translate(&event, &command))
        {
            return false;
        }

        command.page_rows = renderer->document_rows;

        if (!command_engine_execute(state, &command, save_path, &redraw, &should_quit))
        {
            return false;
        }
        if (redraw && !redraw_after_action(renderer, state))
        {
            return false;
        }
        if (should_quit)
        {
            return true;
        }
    }
}

int main(int argc, char *argv[])
{
    if (argc > 2)
    {
        fprintf(stderr, "Usage: %s [filename]\n", argv[0]);
        return 1;
    }

    const char *startup_path = argc == 2 ? argv[1] : NULL;
    bool startup_load_failed = false;
    TerminalState terminal;
    EditorState state;
    Renderer renderer;
    bool state_initialized = false;
    bool renderer_initialized = false;
    int exit_status = 1;

    if (!terminal_init(&terminal))
    {
        return 1;
    }
    if (!terminal_enable_raw_mode(&terminal))
    {
        terminal_restore(&terminal);
        return 1;
    }
    if (!terminal_enter_alternate_screen(&terminal))
    {
        terminal_restore(&terminal);
        return 1;
    }
    if (!editor_state_init(&state))
    {
        goto cleanup;
    }
    state_initialized = true;

    if (startup_path != NULL && !editor_state_load_file(&state, startup_path))
    {
        startup_load_failed = true;
        goto cleanup;
    }
    if (terminal.columns < 2 || !renderer_init(&renderer, terminal.rows, terminal.columns - 1))
    {
        goto cleanup;
    }
    renderer_initialized = true;

    if (!redraw_after_action(&renderer, &state) || !application_event_loop(&state, &renderer, startup_path))
    {
        goto cleanup;
    }

    exit_status = 0;

cleanup:
    if (renderer_initialized)
    {
        renderer_destroy(&renderer);
    }
    if (state_initialized)
    {
        editor_state_destroy(&state);
    }
    terminal_restore(&terminal);
    terminal_leave_alternate_screen(&terminal);

    if (startup_load_failed)
    {
        fprintf(stderr, "tedit: unable to open or decode '%s'.\n", startup_path);
    }

    return exit_status;
}
```

## Behavior

- Page Up moves upward by `Renderer.document_rows`, clamped to row zero.
- Page Down moves downward by the same distance, clamped to the final logical line.
- The Cursor preserves `preferred_column`. If a destination line is shorter, `column` is clamped to that line’s length; a later longer line restores the preferred column.
- Navigation does not change document contents or the Dirty Flag.
- Redraw is requested only when the cursor position changes.
- Filename-entry and quit-confirmation modes retain their existing command handling.

## Next section

Chapter 53.6 — Tab Insertion and Configured Tab Width.
