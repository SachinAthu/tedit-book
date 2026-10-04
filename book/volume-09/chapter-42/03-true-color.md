# 42.3 True Color

Traditional terminals commonly represented colors using a limited palette. An application could request a predefined color, but the number of available colors was constrained by the terminal's color model.

Modern terminal emulators can support **true color**, also called **24-bit color**. Instead of selecting a color from a small predefined palette, the application can specify a color using separate red, green, and blue components.

Conceptually:

```text
Red   ──┐
Green ──┼──► RGB Color
Blue  ──┘
```

Each component can represent a value from 0 through 255:

```text
RGB = (R, G, B)
```

This provides:

```text
256 × 256 × 256
```

possible RGB combinations, or approximately 16.7 million colors.

The important architectural point is that true color **extends the attributes of a terminal cell**. It does not turn the cell into a graphical object.

## Color in the Screen Buffer

The baseline Screen Buffer can conceptually represent a cell as:

```text
+----------------------+
| Character            |
| Foreground           |
| Background           |
| Attributes           |
+----------------------+
```

With true color, the foreground and background values can contain RGB colors rather than merely references to a small fixed palette.

For example:

```text
Cell
├── character = 'A'
├── foreground = RGB(255, 120, 40)
└── background = RGB(20, 20, 20)
```

The renderer can then translate these attributes into the appropriate terminal output.

This is fundamentally different from terminal images. The cell remains a cell; only the precision of its visual attributes has increased.

## Indexed Color and True Color

It is useful to distinguish several common color models.

A simplified progression is:

```text
Basic colors
     ↓
Indexed colors
     ↓
True color
```

In a basic color model, the application selects from a small predefined set.

An indexed-color model provides a larger palette, where a numeric value identifies a palette entry.

True color instead specifies the RGB components directly.

Conceptually:

```text
Indexed:

Color ID
   │
   ▼
Palette Entry
   │
   ▼
RGB Color


True Color:

Red ──┐
Green ┼──► RGB Color
Blue ─┘
```

True color therefore removes the need for the application to select a nearby color from a fixed palette.

## Why True Color Matters to an Editor

A text editor can use color for many presentation purposes:

```text
Syntax highlighting
Status indicators
Selection
Search matches
Diagnostics
UI elements
```

A limited palette may be sufficient for a simple editor, but richer color makes it possible to represent more distinct visual roles.

For example:

```text
keyword       → one color
string        → another
comment       → another
number        → another
diagnostic    → another
selection     → another
```

The important distinction is between **semantic styling** and **terminal color encoding**.

The syntax-highlighting system should not need to know how a particular terminal represents RGB values.

Instead:

```text
Syntax Information
        ↓
Semantic Style
        ↓
Theme
        ↓
Visual Style
        ↓
Screen Cell
        ↓
Terminal Output
```

The terminal-specific representation remains at the presentation boundary.

## True Color Does Not Change the Document Model

A document containing:

```text
int main(void)
{
    return 0;
}
```

does not need to store RGB values simply because the renderer displays the text using colors.

The Document still represents the text.

The color is derived presentation information:

```text
Document
   │
   ▼
Syntax / Presentation Analysis
   │
   ▼
Style
   │
   ▼
Renderer
   │
   ▼
Colored Screen Cells
```

This distinction prevents presentation concerns from leaking into the core document representation.

## True Color and the Renderer

The renderer ultimately needs to convert the visual style of each cell into terminal output.

Conceptually:

```text
Cell
├── character
├── foreground RGB
├── background RGB
└── attributes
       │
       ▼
Terminal Output
```

The Terminal Interface can then encode those values using the terminal's supported color protocol.

The renderer therefore works with **visual properties**, while the Terminal Interface handles their **terminal-specific encoding**.

This follows the same separation established throughout the architecture:

> Higher-level components describe what should be displayed; platform components determine how the target terminal represents it.

## Color Is Still Capability-Dependent

True color should not automatically be treated as universally available.

A terminal environment may support:

- basic colors,
- indexed colors,
- true color,
- or some other level of color capability.

Therefore, an editor that wants to use true color must consider what happens when the capability is unavailable.

A useful conceptual model is:

```text
Desired Color
     │
     ▼
True Color Available?
   /           \
 yes            no
  │              │
 RGB          Fallback
 output       color model
```

The fallback does not change the semantic meaning of the style.

For example, the editor might decide that a particular piece of text represents a keyword. The terminal's capabilities determine how closely that semantic style can be represented visually.

## Semantic Style Versus Exact Color

This distinction becomes particularly important for themes.

An editor should conceptually think in terms of:

```text
keyword
comment
string
error
selection
```

rather than scattering literal RGB values throughout the rendering code.

A theme can map those semantic roles to visual styles:

```text
Keyword → RGB(...)
Comment → RGB(...)
String  → RGB(...)
Error   → RGB(...)
```

The renderer consumes those styles.

This allows the theme system and terminal capability handling to remain separate.

## True Color and Portability

True color also illustrates a broader principle of terminal portability.

The editor may have a preferred visual representation:

```text
Preferred:
RGB(220, 180, 80)
```

But the actual terminal may not be able to reproduce that exact color.

Therefore:

```text
Semantic Intent
      ↓
Preferred Visual Style
      ↓
Terminal Capability
      ↓
Representable Visual Style
```

The goal is not necessarily to guarantee identical pixels on every terminal.

The goal is to preserve the **meaning and hierarchy of the interface** while using the best representation available.

## Architectural Boundary

For our editor, true color should therefore remain primarily a presentation concern.

A clean responsibility division is:

```text
Core
  │
  │ semantic/editor state
  ▼
Renderer
  │
  │ visual style
  ▼
Terminal Interface
  │
  │ terminal-specific color encoding
  ▼
Terminal Emulator
```

The Core does not need to know whether the terminal supports 24-bit RGB.

The Renderer can work with visual styles.

The Terminal Interface handles the terminal-specific representation and any required fallback.

This keeps true color from becoming a dependency throughout the editor.

## The Broader Lesson

True color demonstrates an important property of terminal graphics:

> **A richer terminal capability does not necessarily require a richer application model.**

The editor can continue to reason about text and semantic presentation while the platform layer determines how those concepts are encoded for a particular terminal.

This principle becomes more important when we move from colors to **images**, because images introduce graphical data that cannot naturally be represented as attributes of individual character cells.
