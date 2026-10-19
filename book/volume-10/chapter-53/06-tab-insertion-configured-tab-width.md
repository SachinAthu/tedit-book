# 53.6 Tab Insertion and Configured Tab Width

## Implementation goal

Make Tab a real document-editing command and render stored tab characters according to the tab-width setting while preserving the distinction between logical document positions and visual terminal columns.

## Changed files

- `src/state/editor_state.h`
- `src/state/editor_state.c`
- `src/input/input_engine.c`
- `src/command/command_engine.h`
- `src/command/command_engine.c`
- `src/renderer/renderer.h`
- `src/renderer/renderer.c`
- `src/main.c`
- `tests/test_input_engine.c`
- `tests/test_command_engine.c`
- `tests/test_input_pipeline.c`
- `tests/test_editing.c`

## Complete updated source

### `src/state/editor_state.h`

```c
#ifndef STATE_EDITOR_STATE_H
#define STATE_EDITOR_STATE_H

#include "cursor/cursor.h"
#include "document/document.h"
#include "viewport/viewport.h"

#include <stdbool.h>
#include <stddef.h>

#define EDITOR_MESSAGE_CAPACITY 256
#define EDITOR_FILENAME_CAPACITY 4096
#define DEFAULT_TAB_WIDTH 4

typedef struct
{
    size_t tab_width;
} EditorConfiguration;

typedef struct
{
    Document document;
    Cursor cursor;
    Viewport viewport;
    EditorConfiguration configuration;
    bool is_dirty;
    char *file_path;
    char message[EDITOR_MESSAGE_CAPACITY];
    bool filename_entry_active;
    char filename_entry[EDITOR_FILENAME_CAPACITY];
    size_t filename_entry_length;
    size_t filename_entry_cursor;
    bool quit_confirmation_active;
    bool quit_after_save;
} EditorState;

bool editor_state_init(EditorState *state);

bool editor_state_load_file(EditorState *state, const char *path);

bool editor_state_save_file(EditorState *state, const char *path);

bool editor_state_save_as(EditorState *state, const char *path);

bool editor_state_set_message(EditorState *state, const char *message);

void editor_state_destroy(EditorState *state);

#endif
```

### `src/state/editor_state.c`

```c
#include "editor_state.h"
#include "file/file.h"

#include <string.h>
#include <stdlib.h>

bool editor_state_init(EditorState *state)
{
    if (state == NULL)
    {
        return false;
    }

    /* Zero every field first so partial failures leave a clean state. */
    memset(state, 0, sizeof(*state));

    if (!document_init(&state->document))
    {
        return false;
    }

    cursor_init(&state->cursor);
    viewport_init(&state->viewport);
    state->configuration.tab_width = DEFAULT_TAB_WIDTH;
    state->is_dirty = false;
    state->file_path = NULL;
    state->message[0] = '\0';
    state->filename_entry_active = false;
    state->filename_entry[0] = '\0';
    state->filename_entry_length = 0;
    state->filename_entry_cursor = 0;
    state->quit_confirmation_active = false;
    state->quit_after_save = false;

    return true;
}

bool editor_state_load_file(EditorState *state, const char *path)
{
    if (state == NULL || path == NULL)
    {
        return false;
    }

    char *new_path;
    size_t path_length = strlen(path);

    if (path_length == SIZE_MAX)
    {
        return false;
    }

    new_path = malloc(path_length + 1);
    if (new_path == NULL)
    {
        return false;
    }
    memcpy(new_path, path, path_length + 1);

    /* Do not change the current state or path unless loading succeeds. */
    if (file_load_document(path, &state->document) != FILE_LOAD_OK)
    {
        free(new_path);
        return false;
    }

    free(state->file_path);
    state->file_path = new_path;
    cursor_init(&state->cursor);
    viewport_init(&state->viewport);
    state->is_dirty = false;
    state->filename_entry_active = false;
    state->filename_entry[0] = '\0';
    state->filename_entry_length = 0;
    state->filename_entry_cursor = 0;
    state->quit_confirmation_active = false;
    state->quit_after_save = false;

    return true;
}

bool editor_state_save_file(EditorState *state, const char *path)
{
    if (state == NULL || path == NULL)
    {
        return false;
    }

    if (file_save_document(path, &state->document) != FILE_SAVE_OK)
    {
        return false;
    }

    state->is_dirty = false;
    return true;
}

bool editor_state_save_as(EditorState *state, const char *path)
{
    char *new_path;
    size_t path_length;

    if (state == NULL || path == NULL || path[0] == '\0')
    {
        return false;
    }

    path_length = strlen(path);
    if (path_length == SIZE_MAX)
    {
        return false;
    }

    new_path = malloc(path_length + 1);
    if (new_path == NULL)
    {
        return false;
    }
    memcpy(new_path, path, path_length + 1);

    if (file_save_document(path, &state->document) != FILE_SAVE_OK)
    {
        free(new_path);
        return false;
    }

    free(state->file_path);
    state->file_path = new_path;
    state->is_dirty = false;
    return true;
}

bool editor_state_set_message(EditorState *state, const char *message)
{
    size_t length;

    if (state == NULL || message == NULL)
    {
        return false;
    }

    /* Copy at most a full buffer's worth, leaving room for the terminator. */
    length = strlen(message);
    if (length >= sizeof(state->message))
    {
        length = sizeof(state->message) - 1;
    }

    memmove(state->message, message, length);
    state->message[length] = '\0';
    return true;
}

void editor_state_destroy(EditorState *state)
{
    if (state == NULL)
    {
        return;
    }

    document_destroy(&state->document);
    free(state->file_path);
    state->file_path = NULL;
    cursor_init(&state->cursor);
    viewport_init(&state->viewport);
    state->is_dirty = false;
    state->message[0] = '\0';
    state->filename_entry_active = false;
    state->filename_entry[0] = '\0';
    state->filename_entry_length = 0;
    state->filename_entry_cursor = 0;
    state->quit_confirmation_active = false;
    state->quit_after_save = false;
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
            command_set(command, EDITOR_COMMAND_INSERT_TAB, 0);
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
    EDITOR_COMMAND_INSERT_TAB,
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
    case EDITOR_COMMAND_INSERT_TAB:
    {
        const uint32_t character = command->type == EDITOR_COMMAND_INSERT_TAB
                                       ? (uint32_t)'\t'
                                       : command->character;
        if (!document_insert_character(&state->document, state->cursor.row, state->cursor.column, character))
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
    }
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

### `src/renderer/renderer.h`

```c
#ifndef RENDERER_RENDERER_H
#define RENDERER_RENDERER_H

#include <stdbool.h>
#include <stddef.h>

#include "state/editor_state.h"
#include "screen_buffer.h"

typedef struct
{
    ScreenBuffer screen_buffer;
    size_t document_rows;
    char *output_buffer;
    size_t output_capacity;
} Renderer;


bool renderer_init(Renderer *renderer, size_t rows, size_t columns);

void renderer_destroy(Renderer *renderer);

bool renderer_render(Renderer *renderer, const EditorState *state);

bool renderer_cursor_visual_column(const EditorState *state, size_t *visual_column);

#endif
```

### `src/renderer/renderer.c`

```c
#include "renderer.h"

#include <errno.h>
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

/* Fill an in-memory ScreenBuffer with the visible part of the document. */
static bool renderer_build_frame(Renderer *renderer, const EditorState *state);

static bool renderer_draw_status_bar(Renderer *renderer, const EditorState *state);

static bool renderer_draw_message_bar(Renderer *renderer, const EditorState *state);

static const char *renderer_filename_prompt_prefix(const EditorState *state);

/* Send a finished ScreenBuffer to the terminal. */
static bool renderer_write_frame(Renderer *renderer);

/* Grow the output buffer so it can hold at least 'required' bytes. */
static bool renderer_ensure_output_capacity(Renderer *renderer, size_t required);

/* Write all bytes to the terminal, retrying if the write is interrupted. */
static bool renderer_write_all(const char *data, size_t length);

/* Move the terminal's visible cursor to a given row and column. */
static bool renderer_write_cursor_position(size_t row, size_t column);

static size_t renderer_utf8_length(uint32_t code_point);

static size_t renderer_encode_utf8(uint32_t code_point, char output[4]);

static size_t renderer_tab_advance(size_t visual_column, size_t tab_width);

bool renderer_init(Renderer *renderer, size_t rows, size_t columns)
{
    if (renderer == NULL)
    {
        return false;
    }

    renderer->screen_buffer.cells = NULL;
    renderer->screen_buffer.rows = 0;
    renderer->screen_buffer.columns = 0;
    renderer->document_rows = 0;
    renderer->output_buffer = NULL;
    renderer->output_capacity = 0;

    if (rows < 4 || columns == 0 || !screen_buffer_init(&renderer->screen_buffer, rows, columns))
    {
        return false;
    }

    renderer->document_rows = rows - 3;
    return true;
}

void renderer_destroy(Renderer *renderer)
{
    if (renderer == NULL)
    {
        return;
    }

    screen_buffer_destroy(&renderer->screen_buffer);

    free(renderer->output_buffer);
    renderer->output_buffer = NULL;
    renderer->output_capacity = 0;
}

bool renderer_cursor_visual_column(const EditorState *state, size_t *visual_column)
{
    const DocumentLine *line;
    size_t column = 0;

    if (state == NULL || visual_column == NULL || state->configuration.tab_width == 0)
    {
        return false;
    }

    if (state->document.line_count == 0)
    {
        if (state->cursor.row != 0 || state->cursor.column != 0)
        {
            return false;
        }
        *visual_column = 0;
        return true;
    }
    if (state->cursor.row >= state->document.line_count)
    {
        return false;
    }

    line = &state->document.lines[state->cursor.row];

    if (state->cursor.column > line->length)
    {
        return false;
    }

    for (size_t index = 0; index < state->cursor.column; ++index)
    {
        size_t advance = line->data[index] == (uint32_t)'\t'
                             ? renderer_tab_advance(column, state->configuration.tab_width)
                             : 1;
        if (column > SIZE_MAX - advance)
        {
            return false;
        }
        column += advance;
    }

    *visual_column = column;
    return true;
}

bool renderer_render(Renderer *renderer, const EditorState *state)
{
    bool success;

    if (renderer == NULL || state == NULL || renderer->screen_buffer.cells == NULL)
    {
        return false;
    }
    if (!state->filename_entry_active)
    {
        size_t cursor_visual_column;
        if (!renderer_cursor_visual_column(state, &cursor_visual_column) ||
            state->cursor.row < state->viewport.viewport_row ||
            cursor_visual_column < state->viewport.viewport_column ||
            state->cursor.row - state->viewport.viewport_row >= renderer->document_rows ||
            cursor_visual_column - state->viewport.viewport_column >= renderer->screen_buffer.columns)
        {
            return false;
        }
    }

    if (!renderer_build_frame(renderer, state))
    {
        return false;
    }

    /*
     * Hide the terminal text cursor while drawing.
     */
    if (!renderer_write_all("\x1b[?25l", 6))
    {
        return false;
    }

    success = renderer_write_frame(renderer);

    if (success)
    {
        if (state->filename_entry_active)
        {
            const size_t prompt_length = strlen(renderer_filename_prompt_prefix(state));
            size_t entry_column = prompt_length + state->filename_entry_cursor;
            if (entry_column >= renderer->screen_buffer.columns)
            {
                entry_column = renderer->screen_buffer.columns - 1;
            }
            success = renderer_write_all("\x1b[30;47m", 8) &&
                      renderer_write_cursor_position(renderer->screen_buffer.rows - 2, entry_column);
        }
        else
        {
            size_t cursor_visual_column;
            success = renderer_cursor_visual_column(state, &cursor_visual_column) &&
                      renderer_write_all("\x1b[0m", 4) &&
                      renderer_write_cursor_position(
                          state->cursor.row - state->viewport.viewport_row,
                          cursor_visual_column - state->viewport.viewport_column);
        }
    }

    /*
     * Always attempt to restore cursor visibility,
     * even if frame rendering fails.
     */
    if (!renderer_write_all("\x1b[?25h", 6))
    {
        success = false;
    }

    return success;
}

static bool renderer_build_frame(Renderer *renderer, const EditorState *state)
{
    if (renderer == NULL || state == NULL)
    {
        return false;
    }

    if (renderer->screen_buffer.cells == NULL)
    {
        return false;
    }

    /*
     * Start from a clean slate: every cell becomes a blank space. A
     * zero-line Document (the representation of an empty file) therefore
     * renders as a blank document area without requiring a synthetic line.
     */
    screen_buffer_clear(&renderer->screen_buffer, (uint32_t)' ');

    /* Map each screen row to its corresponding logical document row. */
    for (size_t screen_row = 0; screen_row < renderer->document_rows; screen_row++)
    {
        const size_t document_row = state->viewport.viewport_row + screen_row;
        const DocumentLine *line;

        if (document_row >= state->document.line_count)
        {
            break;
        }

        line = &state->document.lines[document_row];

        /* Expand logical characters into visual cells before horizontal clipping. */
        size_t visual_column = 0;
        for (size_t document_column = 0; document_column < line->length; ++document_column)
        {
            const uint32_t code_point = line->data[document_column];
            const size_t advance = code_point == (uint32_t)'\t'
                                       ? renderer_tab_advance(visual_column, state->configuration.tab_width)
                                       : 1;

            if (code_point == (uint32_t)'\t')
            {
                for (size_t offset = 0; offset < advance; ++offset)
                {
                    const size_t cell_column = visual_column + offset;
                    if (cell_column >= state->viewport.viewport_column &&
                        cell_column - state->viewport.viewport_column < renderer->screen_buffer.columns)
                    {
                        if (!screen_buffer_set(&renderer->screen_buffer, screen_row,
                                               cell_column - state->viewport.viewport_column,
                                               (uint32_t)' '))
                        {
                            return false;
                        }
                    }
                }
            }
            else if (visual_column >= state->viewport.viewport_column &&
                     visual_column - state->viewport.viewport_column < renderer->screen_buffer.columns)
            {
                if (!screen_buffer_set(&renderer->screen_buffer, screen_row,
                                       visual_column - state->viewport.viewport_column, code_point))
                {
                    return false;
                }
            }

            if (visual_column > SIZE_MAX - advance)
            {
                return false;
            }
            visual_column += advance;
            if (visual_column >= state->viewport.viewport_column &&
                visual_column - state->viewport.viewport_column >= renderer->screen_buffer.columns)
            {
                break;
            }
        }
    }

    return renderer_draw_status_bar(renderer, state) &&
           renderer_draw_message_bar(renderer, state);
}

static bool renderer_draw_status_bar(Renderer *renderer, const EditorState *state)
{
    char right[96];
    const char *path;
    const char *suffix;
    const char *hint;
    size_t path_length;
    size_t suffix_length;
    int right_length;
    size_t status_row;
    size_t left_limit;
    size_t path_limit;
    size_t path_position = 0;
    size_t right_position;

    if (renderer == NULL || state == NULL || renderer->document_rows == 0)
    {
        return false;
    }

    status_row = renderer->screen_buffer.rows - 1;

    /*
     * While a prompt is capturing keys, the Status Bar row shows the prompt's
     * key hints instead of the file name and cursor position, which do not
     * change while the prompt is open.
     */
    if (state->filename_entry_active || state->quit_confirmation_active)
    {
        hint = state->filename_entry_active ? "Enter: save  Esc: cancel"
                                            : "Y=save N=discard Esc=cancel";
        for (size_t column = 0; column < renderer->screen_buffer.columns; ++column)
        {
            if (!screen_buffer_set(&renderer->screen_buffer, status_row, column, (uint32_t)' '))
            {
                return false;
            }
        }
        for (size_t index = 0; hint[index] != '\0' && index < renderer->screen_buffer.columns; ++index)
        {
            if (!screen_buffer_set(&renderer->screen_buffer, status_row, index,
                                   (uint32_t)(unsigned char)hint[index]))
            {
                return false;
            }
        }
        return true;
    }

    path = state->file_path != NULL ? state->file_path : "[No Name]";
    path_length = strlen(path);
    suffix = state->is_dirty ? " [+]" : "";
    suffix_length = strlen(suffix);
    right_length = snprintf(right, sizeof(right), "Ln %zu, Col %zu",
                            state->cursor.row + 1, state->cursor.column + 1);
    if (right_length < 0 || (size_t)right_length >= sizeof(right))
    {
        return false;
    }

    left_limit = renderer->screen_buffer.columns > (size_t)right_length + 2
                     ? renderer->screen_buffer.columns - (size_t)right_length - 2
                     : 0;
    path_limit = left_limit > suffix_length ? left_limit - suffix_length : 0;

    for (size_t column = 0; column < renderer->screen_buffer.columns; ++column)
    {
        if (!screen_buffer_set(&renderer->screen_buffer, status_row, column, (uint32_t)' '))
        {
            return false;
        }
    }

    /* Render a bounded path prefix; arbitrary filesystem bytes stay printable. */
    while (path_position < path_length && path_position < path_limit)
    {
        unsigned char byte = (unsigned char)path[path_position];
        uint32_t code_point = byte >= 0x20 && byte < 0x7F
                                  ? byte
                                  : (byte >= 0x80 ? (uint32_t)'?' : (uint32_t)' ');
        if (!screen_buffer_set(&renderer->screen_buffer, status_row, path_position, code_point))
        {
            return false;
        }
        ++path_position;
    }

    for (size_t index = 0; index < suffix_length && path_position + index < left_limit; ++index)
    {
        if (!screen_buffer_set(&renderer->screen_buffer, status_row, path_position + index,
                               (uint32_t)(unsigned char)suffix[index]))
        {
            return false;
        }
    }

    if ((size_t)right_length < renderer->screen_buffer.columns)
    {
        right_position = renderer->screen_buffer.columns - (size_t)right_length;
        for (size_t index = 0; index < (size_t)right_length; ++index)
        {
            if (!screen_buffer_set(&renderer->screen_buffer, status_row, right_position + index,
                                   (uint32_t)(unsigned char)right[index]))
            {
                return false;
            }
        }
    }

    return true;
}

static const char *renderer_filename_prompt_prefix(const EditorState *state)
{
    if (state == NULL)
    {
        return "Save as: ";
    }
    if (strncmp(state->message, "Save failed", sizeof("Save failed") - 1) == 0)
    {
        return "Save failed: ";
    }
    if (strncmp(state->message, "Filename is too long", sizeof("Filename is too long") - 1) == 0)
    {
        return "Filename too long: ";
    }
    if (strncmp(state->message, "Enter a file name", sizeof("Enter a file name") - 1) == 0)
    {
        return "Enter a file name: ";
    }
    return "Save as: ";
}

static bool renderer_draw_message_bar(Renderer *renderer, const EditorState *state)
{
    const char *prompt;
    size_t prompt_length;
    size_t message_row;
    size_t column = 0;

    if (renderer == NULL || state == NULL || renderer->screen_buffer.cells == NULL ||
        renderer->screen_buffer.rows < 4)
    {
        return false;
    }

    message_row = renderer->screen_buffer.rows - 2;
    prompt = renderer_filename_prompt_prefix(state);
    prompt_length = strlen(prompt);
    for (column = 0; column < renderer->screen_buffer.columns; ++column)
    {
        if (!screen_buffer_set(&renderer->screen_buffer, message_row, column, (uint32_t)' '))
        {
            return false;
        }
    }

    if (state->filename_entry_active)
    {
        for (size_t index = 0; index < prompt_length && index < renderer->screen_buffer.columns; ++index)
        {
            if (!screen_buffer_set(&renderer->screen_buffer, message_row, index, (uint32_t)(unsigned char)prompt[index]))
            {
                return false;
            }
        }

        column = prompt_length;
        for (size_t index = 0; index < state->filename_entry_length && column < renderer->screen_buffer.columns; ++index, ++column)
        {
            const unsigned char byte = (unsigned char)state->filename_entry[index];
            const uint32_t code_point = byte >= 0x20 && byte < 0x7F
                                            ? byte
                                            : (byte >= 0x80 ? (uint32_t)'?' : (uint32_t)' ');
            if (!screen_buffer_set(&renderer->screen_buffer, message_row, column, code_point))
            {
                return false;
            }
        }
        return true;
    }

    const size_t message_length = strlen(state->message);
    const size_t visible_length = message_length < renderer->screen_buffer.columns
                                      ? message_length
                                      : renderer->screen_buffer.columns;
    for (column = 0; column < visible_length; ++column)
    {
        const unsigned char byte = (unsigned char)state->message[column];
        const uint32_t code_point = byte >= 0x20 && byte < 0x7F
                                        ? byte
                                        : (byte >= 0x80 ? (uint32_t)'?' : (uint32_t)' ');
        if (!screen_buffer_set(&renderer->screen_buffer, message_row, column, code_point))
        {
            return false;
        }
    }

    return true;
}

static bool renderer_ensure_output_capacity(Renderer *renderer, size_t required)
{
    if (renderer == NULL)
    {
        return false;
    }

    if (required <= renderer->output_capacity)
    {
        return true;
    }

    char *larger = realloc(renderer->output_buffer, required);
    if (larger == NULL)
    {
        return false;
    }

    renderer->output_buffer = larger;
    renderer->output_capacity = required;

    return true;
}

static bool renderer_write_frame(Renderer *renderer)
{
    const ScreenBuffer *buffer;
    char sequence[64];

    if (renderer == NULL)
    {
        return false;
    }

    buffer = &renderer->screen_buffer;

    if (buffer->cells == NULL)
    {
        return false;
    }

    /*
     * First pass: measure the ANSI cursor-position sequences and
     * UTF-8-encoded cell contents.
     */
    size_t frame_length = 0;

    for (size_t row = 0; row < buffer->rows; row++)
    {
        int sequence_length = snprintf(sequence, sizeof(sequence), "\x1b[%zu;1H", row + 1);

        if (sequence_length < 0 || (size_t)sequence_length >= sizeof(sequence))
        {
            return false;
        }

        if (frame_length > SIZE_MAX - (size_t)sequence_length)
        {
            return false;
        }

        frame_length += (size_t)sequence_length;

        const size_t message_row = buffer->rows - 2;
        const size_t style_length = row == message_row ? sizeof("\x1b[30;47m") - 1
                                                       : sizeof("\x1b[0m") - 1;
        if (frame_length > SIZE_MAX - style_length)
        {
            return false;
        }
        frame_length += style_length;

        for (size_t column = 0; column < buffer->columns; ++column)
        {
            const ScreenCell *cell = screen_buffer_get(buffer, row, column);

            if (cell == NULL)
            {
                return false;
            }

            size_t encoded_length = renderer_utf8_length(cell->code_point);

            if (encoded_length == 0 || frame_length > SIZE_MAX - encoded_length)
            {
                return false;
            }

            frame_length += encoded_length;
        }
    }

    if (!renderer_ensure_output_capacity(renderer, frame_length))
    {
        return false;
    }

    /*
     * Second pass: construct the complete terminal output.
     */
    size_t position = 0;

    for (size_t row = 0; row < buffer->rows; ++row)
    {
        int sequence_length = snprintf(sequence, sizeof(sequence), "\x1b[%zu;1H", row + 1);

        if (sequence_length < 0 || (size_t)sequence_length >= sizeof(sequence))
        {
            return false;
        }

        memcpy(renderer->output_buffer + position, sequence, (size_t)sequence_length);
        position += (size_t)sequence_length;

        if (row == buffer->rows - 2)
        {
            static const char message_style[] = "\x1b[30;47m";
            memcpy(renderer->output_buffer + position, message_style, sizeof(message_style) - 1);
            position += sizeof(message_style) - 1;
        }
        else
        {
            static const char reset_style[] = "\x1b[0m";
            memcpy(renderer->output_buffer + position, reset_style, sizeof(reset_style) - 1);
            position += sizeof(reset_style) - 1;
        }

        for (size_t column = 0; column < buffer->columns; ++column)
        {
            const ScreenCell *cell = screen_buffer_get(buffer, row, column);

            if (cell == NULL)
            {
                return false;
            }

            char encoded[4];
            size_t encoded_length = renderer_encode_utf8(cell->code_point, encoded);

            if (encoded_length == 0)
            {
                return false;
            }

            memcpy(renderer->output_buffer + position, encoded, encoded_length);

            position += encoded_length;
        }
    }

    return renderer_write_all(renderer->output_buffer, frame_length);
}

static bool renderer_write_all(const char *data, size_t length)
{
    if (data == NULL)
    {
        return false;
    }

    size_t written = 0;

    while (written < length)
    {
        ssize_t result;

        /* Ask the OS to write the remaining bytes. */
        result = write(STDOUT_FILENO, data + written, length - written);

        if (result < 0)
        {
            /* EINTR means a signal interrupted us; retrying is safe. */
            if (errno == EINTR)
            {
                continue;
            }

            return false;
        }

        /* A return value of 0 means nothing could be written; give up. */
        if (result == 0)
        {
            return false;
        }

        written += (size_t)result;
    }

    return true;
}

static bool renderer_write_cursor_position(size_t row, size_t column)
{
    char sequence[64];
    int sequence_length = snprintf(sequence, sizeof(sequence), "\x1b[%zu;%zuH", row + 1, column + 1);

    if (sequence_length < 0 || (size_t)sequence_length >= sizeof(sequence))
    {
        return false;
    }

    return renderer_write_all(sequence, (size_t)sequence_length);
}

static size_t renderer_utf8_length(uint32_t code_point)
{
    if (code_point <= UINT32_C(0x7F))
    {
        return 1;
    }

    if (code_point <= UINT32_C(0x7FF))
    {
        return 2;
    }

    if (code_point >= UINT32_C(0xD800) && code_point <= UINT32_C(0xDFFF))
    {
        return 0;
    }

    if (code_point <= UINT32_C(0xFFFF))
    {
        return 3;
    }

    if (code_point <= UINT32_C(0x10FFFF))
    {
        return 4;
    }

    return 0;
}

static size_t renderer_encode_utf8(uint32_t code_point, char output[4])
{
    size_t length = renderer_utf8_length(code_point);

    if (length == 0 || output == NULL)
    {
        return 0;
    }

    if (length == 1)
    {
        output[0] = (char)code_point;
    }
    else if (length == 2)
    {
        output[0] = (char)(UINT32_C(0xC0) | (code_point >> 6));

        output[1] = (char)(UINT32_C(0x80) | (code_point & UINT32_C(0x3F)));
    }
    else if (length == 3)
    {
        output[0] = (char)(UINT32_C(0xE0) | (code_point >> 12));

        output[1] = (char)(UINT32_C(0x80) | ((code_point >> 6) & UINT32_C(0x3F)));

        output[2] = (char)(UINT32_C(0x80) | (code_point & UINT32_C(0x3F)));
    }
    else
    {
        output[0] = (char)(UINT32_C(0xF0) | (code_point >> 18));

        output[1] = (char)(UINT32_C(0x80) | ((code_point >> 12) & UINT32_C(0x3F)));

        output[2] = (char)(UINT32_C(0x80) | ((code_point >> 6) & UINT32_C(0x3F)));

        output[3] = (char)(UINT32_C(0x80) | (code_point & UINT32_C(0x3F)));
    }

    return length;
}

static size_t renderer_tab_advance(size_t visual_column, size_t tab_width)
{
    return tab_width - (visual_column % tab_width);
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

    size_t cursor_visual_column;
    if (!renderer_cursor_visual_column(state, &cursor_visual_column) ||
        !viewport_ensure_cursor_visible(
            &state->viewport,
            state->cursor.row,
            cursor_visual_column,
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

- `EditorState` owns an `EditorConfiguration` with `tab_width` initialized to `DEFAULT_TAB_WIDTH` (4).
- Tab maps to the dedicated `EDITOR_COMMAND_INSERT_TAB` command and inserts U+0009 as one logical Document code point.
- The command marks the Document dirty and requests redraw only after successful insertion.
- Filename-entry mode ignores the dedicated Tab command rather than inserting a control byte into a path.
- The Renderer expands tabs into spaces through the next configured tab stop.
- `renderer_cursor_visual_column()` maps the logical cursor position to a visual column.
- The Event Loop uses visual columns when ensuring the cursor remains visible; the Renderer uses the same mapping for cursor placement and visibility validation.
- `Cursor.column` remains a logical code-point column. `Viewport.viewport_column` is a visual display column.
- Other Unicode code points still occupy one display cell in this milestone. Full Unicode display-width handling remains for Sections 53.7–53.8.
- Configuration file loading and command-line overrides are not added here; the built-in configuration value and consumer are established.

## Next section

Chapter 53.7 — UTF-8 Keyboard Input and Character-Boundary Editing

Next, we will verify and complete UTF-8 input integration so cursor movement and editing operate on complete Unicode code points rather than arbitrary bytes, while preserving the existing Document representation.
