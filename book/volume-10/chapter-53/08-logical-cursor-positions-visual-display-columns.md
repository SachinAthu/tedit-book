# 53.8 Logical Cursor Positions and Visual Display Columns

## Implementation goal

Complete the mapping between logical Document positions and terminal display columns without changing the Document or Cursor coordinate model.

## Changed files

- `src/renderer/renderer.c`
- `src/renderer/screen_buffer.c`
- `src/renderer/screen_buffer.h`
- `src/main.c`
- `tests/test_editing.c`

## Complete updated source

### `src/renderer/renderer.c`

```c
#define _XOPEN_SOURCE 700
#include "renderer.h"

#include <errno.h>
#include <stdint.h>
#include <stdio.h>
#include <wchar.h>
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

static int renderer_code_point_width(uint32_t code_point);

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
        size_t advance;
        if (line->data[index] == (uint32_t)'\t')
        {
            advance = renderer_tab_advance(column, state->configuration.tab_width);
        }
        else
        {
            int width = renderer_code_point_width(line->data[index]);
            if (width < 0)
            {
                width = 1;
            }
            if (width == 0 && column == 0)
            {
                /* A leading combining mark is displayed with a dotted circle. */
                width = 1;
            }
            advance = (size_t)width;
        }
        if (column > SIZE_MAX - advance)
        {
            return false;
        }
        column += advance;
    }

    *visual_column = column;
    return true;
}

static size_t renderer_tab_advance(size_t visual_column, size_t tab_width)
{
    return tab_width - (visual_column % tab_width);
}

static bool renderer_code_point_in_range(uint32_t code_point, uint32_t first, uint32_t last)
{
    return code_point >= first && code_point <= last;
}

static int renderer_code_point_width(uint32_t code_point)
{
    int width;
    if (code_point > UINT32_C(0x10FFFF) ||
        (code_point >= UINT32_C(0xD800) && code_point <= UINT32_C(0xDFFF)))
    {
        return -1;
    }

    width = wcwidth((wchar_t)code_point);
    if (width >= 0)
    {
        return width;
    }

    /* Deterministic fallback for systems running in the POSIX/C locale. */
    if (renderer_code_point_in_range(code_point, 0x0300, 0x036F) ||
        renderer_code_point_in_range(code_point, 0x0483, 0x0489) ||
        renderer_code_point_in_range(code_point, 0x0591, 0x05BD) ||
        renderer_code_point_in_range(code_point, 0x05BF, 0x05BF) ||
        renderer_code_point_in_range(code_point, 0x05C1, 0x05C2) ||
        renderer_code_point_in_range(code_point, 0x0610, 0x061A) ||
        renderer_code_point_in_range(code_point, 0x064B, 0x065F) ||
        renderer_code_point_in_range(code_point, 0x0670, 0x0670) ||
        renderer_code_point_in_range(code_point, 0x06D6, 0x06ED) ||
        renderer_code_point_in_range(code_point, 0x1AB0, 0x1AFF) ||
        renderer_code_point_in_range(code_point, 0x1DC0, 0x1DFF) ||
        renderer_code_point_in_range(code_point, 0x20D0, 0x20FF) ||
        renderer_code_point_in_range(code_point, 0xFE00, 0xFE0F) ||
        renderer_code_point_in_range(code_point, 0xFE20, 0xFE2F) ||
        renderer_code_point_in_range(code_point, 0xE0100, 0xE01EF))
    {
        return 0;
    }

    if (renderer_code_point_in_range(code_point, 0x1100, 0x115F) ||
        renderer_code_point_in_range(code_point, 0x2329, 0x232A) ||
        renderer_code_point_in_range(code_point, 0x2E80, 0xA4CF) ||
        renderer_code_point_in_range(code_point, 0xAC00, 0xD7A3) ||
        renderer_code_point_in_range(code_point, 0xF900, 0xFAFF) ||
        renderer_code_point_in_range(code_point, 0xFE10, 0xFE19) ||
        renderer_code_point_in_range(code_point, 0xFE30, 0xFE6F) ||
        renderer_code_point_in_range(code_point, 0xFF00, 0xFF60) ||
        renderer_code_point_in_range(code_point, 0xFFE0, 0xFFE6) ||
        renderer_code_point_in_range(code_point, 0x1F300, 0x1FAFF) ||
        renderer_code_point_in_range(code_point, 0x20000, 0x3FFFD))
    {
        return 2;
    }

    return 1;
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

        /* Map logical code points to terminal cells before horizontal clipping. */
        size_t visual_column = 0;
        size_t previous_cell_column = 0;
        bool has_previous_cell = false;
        for (size_t document_column = 0; document_column < line->length; ++document_column)
        {
            const uint32_t code_point = line->data[document_column];
            size_t advance;

            if (code_point == (uint32_t)'\t')
            {
                advance = renderer_tab_advance(visual_column, state->configuration.tab_width);
                if (visual_column > SIZE_MAX - advance)
                {
                    return false;
                }
                for (size_t offset = 0; offset < advance; ++offset)
                {
                    const size_t cell_column = visual_column + offset;
                    if (cell_column >= state->viewport.viewport_column &&
                        cell_column - state->viewport.viewport_column < renderer->screen_buffer.columns &&
                        !screen_buffer_set(&renderer->screen_buffer, screen_row,
                                           cell_column - state->viewport.viewport_column, (uint32_t)' '))
                    {
                        return false;
                    }
                }
                previous_cell_column = visual_column + advance - 1;
                has_previous_cell = true;
            }
            else
            {
                int width = renderer_code_point_width(code_point);
                uint32_t display_code_point = code_point;
                if (width < 0)
                {
                    width = 1;
                    display_code_point = (uint32_t)'?';
                }
                if (width == 0)
                {
                    if (has_previous_cell)
                    {
                        if (previous_cell_column >= state->viewport.viewport_column &&
                            previous_cell_column - state->viewport.viewport_column < renderer->screen_buffer.columns &&
                            !screen_buffer_append_combining(&renderer->screen_buffer, screen_row,
                                                            previous_cell_column - state->viewport.viewport_column,
                                                            code_point))
                        {
                            return false;
                        }
                        continue;
                    }
                    display_code_point = UINT32_C(0x25CC); /* dotted circle */
                    width = 1;
                }

                advance = (size_t)width;
                if (visual_column > SIZE_MAX - advance)
                {
                    return false;
                }
                if (visual_column >= state->viewport.viewport_column &&
                    visual_column - state->viewport.viewport_column < renderer->screen_buffer.columns &&
                    advance <= renderer->screen_buffer.columns -
                                   (visual_column - state->viewport.viewport_column))
                {
                    const size_t cell_column = visual_column - state->viewport.viewport_column;
                    if (!screen_buffer_set(&renderer->screen_buffer, screen_row, cell_column, display_code_point))
                    {
                        return false;
                    }
                    if (advance == 2 &&
                        !screen_buffer_set_continuation(&renderer->screen_buffer, screen_row, cell_column + 1))
                    {
                        return false;
                    }
                    previous_cell_column = visual_column;
                    has_previous_cell = true;
                    if (display_code_point == UINT32_C(0x25CC) &&
                        !screen_buffer_append_combining(&renderer->screen_buffer, screen_row, cell_column, code_point))
                    {
                        return false;
                    }
                }
                else
                {
                    previous_cell_column = visual_column;
                    has_previous_cell = true;
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

            if (cell->continuation)
            {
                continue;
            }
            size_t encoded_length = renderer_utf8_length(cell->code_point);
            if (encoded_length == 0 || frame_length > SIZE_MAX - encoded_length)
            {
                return false;
            }
            frame_length += encoded_length;
            for (size_t mark = 0; mark < cell->combining_count; ++mark)
            {
                const size_t mark_length = renderer_utf8_length(cell->combining_code_points[mark]);
                if (mark_length == 0 || frame_length > SIZE_MAX - mark_length)
                {
                    return false;
                }
                frame_length += mark_length;
            }
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

            if (cell->continuation)
            {
                continue;
            }
            char encoded[4];
            size_t encoded_length = renderer_encode_utf8(cell->code_point, encoded);

            if (encoded_length == 0)
            {
                return false;
            }

            memcpy(renderer->output_buffer + position, encoded, encoded_length);
            position += encoded_length;
            for (size_t mark = 0; mark < cell->combining_count; ++mark)
            {
                char mark_bytes[4];
                const size_t mark_length = renderer_encode_utf8(cell->combining_code_points[mark], mark_bytes);
                if (mark_length == 0)
                {
                    return false;
                }
                memcpy(renderer->output_buffer + position, mark_bytes, mark_length);
                position += mark_length;
            }
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
```

### `src/renderer/screen_buffer.h`

```c
#ifndef RENDERER_SCREEN_BUFFER_H
#define RENDERER_SCREEN_BUFFER_H

#include <stdbool.h>
#include <stddef.h>
#include <stdint.h>

/*
 * A terminal display cell. Wide code points use a base cell plus a continuation
 * cell; zero-width combining code points are stored with their base cell.
 */
typedef struct
{
    uint32_t code_point;
    bool continuation;
    uint32_t *combining_code_points;
    size_t combining_count;
    size_t combining_capacity;
} ScreenCell;

typedef struct
{
    ScreenCell *cells;
    size_t rows;
    size_t columns;
} ScreenBuffer;

bool screen_buffer_init(ScreenBuffer *buffer, size_t rows, size_t columns);

void screen_buffer_destroy(ScreenBuffer *buffer);

void screen_buffer_clear(ScreenBuffer *buffer, uint32_t code_point);

bool screen_buffer_set(ScreenBuffer *buffer, size_t row, size_t column, uint32_t code_point);

const ScreenCell *screen_buffer_get(const ScreenBuffer *buffer, size_t row, size_t column);

/* Mark a cell as the second column occupied by a wide code point. */
bool screen_buffer_set_continuation(ScreenBuffer *buffer, size_t row, size_t column);

/* Append a zero-width combining code point to an existing base cell. */
bool screen_buffer_append_combining(ScreenBuffer *buffer, size_t row, size_t column, uint32_t code_point);

#endif
```

### `src/renderer/screen_buffer.c`

```c
#include "screen_buffer.h"

#include <stdint.h>
#include <stdlib.h>


bool screen_buffer_init(ScreenBuffer *buffer, size_t rows, size_t columns)
{
    if (buffer == NULL || rows == 0 || columns == 0)
    {
        return false;
    }

    /* Start from a known-empty state before allocating. */
    buffer->cells = NULL;
    buffer->rows = 0;
    buffer->columns = 0;

    /* Reject sizes where the cell count itself would overflow a size_t. */
    if (rows > SIZE_MAX / columns)
    {
        return false;
    }

    size_t cell_count = rows * columns;

    /* Reject sizes whose byte size would overflow a size_t. */
    if (cell_count > SIZE_MAX / sizeof(*buffer->cells))
    {
        return false;
    }

    /* Allocate one contiguous block holding every cell. */
    buffer->cells = calloc(cell_count, sizeof(*buffer->cells));
    if (buffer->cells == NULL)
    {
        return false;
    }

    buffer->rows = rows;
    buffer->columns = columns;

    /* Begin with a blank screen. */
    screen_buffer_clear(buffer, ' ');

    return true;
}

void screen_buffer_destroy(ScreenBuffer *buffer)
{
    if (buffer == NULL)
    {
        return;
    }

    if (buffer->cells != NULL)
    {
        const size_t cell_count = buffer->rows * buffer->columns;
        for (size_t index = 0; index < cell_count; ++index)
        {
            free(buffer->cells[index].combining_code_points);
        }
    }
    free(buffer->cells);

    buffer->cells = NULL;
    buffer->rows = 0;
    buffer->columns = 0;
}

void screen_buffer_clear(ScreenBuffer *buffer, uint32_t code_point)
{
    if (buffer == NULL || buffer->cells == NULL)
    {
        return;
    }

    size_t cell_count = buffer->rows * buffer->columns;

    /* Clear visible content while retaining allocated combining-mark capacity. */
    for (size_t index = 0; index < cell_count; ++index)
    {
        buffer->cells[index].code_point = code_point;
        buffer->cells[index].continuation = false;
        buffer->cells[index].combining_count = 0;
    }
}

bool screen_buffer_set(ScreenBuffer *buffer, size_t row, size_t column, uint32_t code_point)
{
    size_t index;

    if (buffer == NULL || buffer->cells == NULL || row >= buffer->rows || column >= buffer->columns)
    {
        return false;
    }

    /* Convert the 2D coordinate into a 1D block index. */
    index = row * buffer->columns + column;
    buffer->cells[index].code_point = code_point;
    buffer->cells[index].continuation = false;
    buffer->cells[index].combining_count = 0;

    return true;
}

const ScreenCell *screen_buffer_get(const ScreenBuffer *buffer, size_t row, size_t column)
{
    size_t index;

    if (buffer == NULL || buffer->cells == NULL || row >= buffer->rows || column >= buffer->columns)
    {
        return NULL;
    }

    index = row * buffer->columns + column;

    return &buffer->cells[index];
}


bool screen_buffer_set_continuation(ScreenBuffer *buffer, size_t row, size_t column)
{
    size_t index;
    if (buffer == NULL || buffer->cells == NULL || row >= buffer->rows || column >= buffer->columns)
    {
        return false;
    }
    index = row * buffer->columns + column;
    buffer->cells[index].code_point = (uint32_t)' ';
    buffer->cells[index].continuation = true;
    buffer->cells[index].combining_count = 0;
    return true;
}

bool screen_buffer_append_combining(ScreenBuffer *buffer, size_t row, size_t column, uint32_t code_point)
{
    size_t index;
    ScreenCell *cell;
    if (buffer == NULL || buffer->cells == NULL || row >= buffer->rows || column >= buffer->columns)
    {
        return false;
    }
    index = row * buffer->columns + column;
    cell = &buffer->cells[index];
    if (cell->combining_count == cell->combining_capacity)
    {
        size_t capacity = cell->combining_capacity == 0 ? 2 : cell->combining_capacity * 2;
        if (capacity < cell->combining_capacity || capacity > SIZE_MAX / sizeof(*cell->combining_code_points))
        {
            return false;
        }
        uint32_t *larger = realloc(cell->combining_code_points, capacity * sizeof(*larger));
        if (larger == NULL)
        {
            return false;
        }
        cell->combining_code_points = larger;
        cell->combining_capacity = capacity;
    }
    cell->combining_code_points[cell->combining_count++] = code_point;
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
#include <locale.h>
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
    (void)setlocale(LC_CTYPE, "");

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

## Implementation

- `Cursor.column` and `DocumentLine.length` remain logical code-point counts.
- `renderer_cursor_visual_column()` advances tabs to the next configured tab stop and uses Unicode display widths for other code points.
- POSIX `wcwidth()` is used when the active locale provides width information. Deterministic fallback ranges cover common combining and wide-character blocks in the C/POSIX locale.
- Wide code points occupy a base cell and a continuation cell in the Screen Buffer. The terminal output emits the wide code point once and skips the continuation cell so terminal columns stay aligned.
- Combining marks are stored with the base cell and emitted immediately after it without consuming another visual column. A leading combining mark is rendered with a dotted-circle base.
- Screen Buffer clearing resets visible continuation and combining counts while retaining allocations for reuse; destruction frees per-cell combining storage.
- The main program initializes `LC_CTYPE` from the environment so `wcwidth()` can use the user's configured locale.
- Viewport visibility and terminal cursor placement continue to consume visual columns, while the Status Bar reports the logical cursor position.

## Tests

Added regression coverage for a CJK wide code point, a combining acute accent, logical-to-visual cursor mapping, Screen Buffer continuation cells, attached combining data, emitted UTF-8 output, and terminal cursor placement.

## Remaining limitations

The fallback width ranges are not a complete, versioned Unicode East Asian Width database. Emoji ZWJ grapheme clusters, variation-selector presentation, and terminal-specific width differences may still vary. Non-ASCII filename rendering remains limited separately.

## Next milestone

Chapter 53.9 — Input Engine and Command Engine Integration.
