# $AG_L$ - Animated Graphics Language

**$AG_L$** is a robust programming environment and language designed for the procedural generation of 2D graphics, animations, and interactive visualizations. It compiles directly to generic **Python3** using the *tkinter* library.

The language is built on the concept of an infinite **Canvas** where figures are instantiated, and **Views** which act as viewports capturing specific regions, zoom levels, and timeframes of that canvas. $AG_L$ supports primitive shapes, complex constructive geometry, custom model definitions with reactive properties, and a dedicated runtime scripting extension ($xAG_L$).

-----

## Demos

**Towers of Hanoi**
A demonstration of complex logic handling, recursion, and multi-view rendering. This simulation uses custom Models for the towers and disks, utilizing semantic actions to handle game rules.

![Intro.gif](doc/examples/demo/Intro.gif)

-----

## Architecture & Workflow

The $AG_L$ environment operates on a compilation-interpretation hybrid model. The core $AG_L$ code defines the static structure and logic of the program, which is compiled into . A secondary layer, **$xAG_L$**, serves as a runtime interpreted language for dynamic instructions (e.g., user input scripts or external animation files).

### Key Concepts

1.  **The Canvas**: An infinite drawing area.
2.  **The View**: A camera object defined by an origin (`Ox`, `Oy`) and dimensions.
3.  **The Model**: A blueprint for objects that can contain logic (Actions) triggered by property changes.

-----

## Language Specification

### Data Types

$AG_L$ is strongly typed. Instantiation uses the `(:)` operator, and assignment uses `(=)`.

  * **Primitives**: `Integer` (0), `Number` (0.0), `String` (""), `Boolean` (False).
  * **Geometry**:
      * `Point`: A coordinate tuple $(x, y)$.
      * `Vector`: A directional magnitude, supported in Cartesian $(x, y)$ or Polar $angle:length$ notation.
  * **Time**: Represents positive values for temporal operations.
  * **Collections**: `Array<Type>` for homogeneous lists.
  * **Enum**: Named constants.

**Examples:**

```
# Scalar Types
a : Integer = 3
b : Number = 2.0 * a
name : String = "AGL Demo"

# Geometry Math
p1 : Point = (1, 5)
v1 : Vector = (2, 3)
p2 : Point = p1 + v1      # Result is a Point
v2 : Vector = 50:10       # Polar coordinates (Angle:Length)

# Time
t1 : Time = 0.8
t2 : Time = 0.5
total : Time = t1 + t2

# Boolean Logic
isValid : Boolean = 1 == 1 and (2 == 1+1 or 2!=2)

# Arrays
matrix : Array<Array<Point>> = [[(1,2), (3,4)], [(5,6), (7,8)]]
val : Point = matrix[0][1] # Accessing (3,4)
```

### Views & Canvas

A **View** determines what is rendered. You can define the viewport center, dimensions, title, and background.

```
mainView : View with {
  Ox = 0.0;
  Oy = 0.0;
  width = 601;
  height = 601;
  title = "Main Display";
  background = "blue";
}

# View Operations
refresh mainView after 100 ms;    # Render frame after delay
p : Point = wait mouse click;     # Pause execution until click
close mainView;                   # Terminate window
```

### Graphic Objects

Objects are instantiated using the `Type at Position with { Properties }` syntax.

#### Basic Primitives

Supported primitives include **Dot**, **Line**, **Rectangle**, **Ellipse**, and **Text**.

```
# A rectangle defined by center point and a vector to the top-right corner
rect : Rectangle at (10, 20) with {
  length = (30, 40);
  fill = "blue";
}

# Text element
lbl : Text at (10, 20) with {
  text = "Hello World";
  fill = "red";
}
```

#### Arcs and Slices

Derived from the Ellipse, these require `start` (angle) and `extent` (sweep) properties.

  * **Arc**: Unclosed curve.
  * **ArcChord**: Connected directly start-to-end (flat edge).
  * **PieSlice**: Connected to the center (pie chart style).

<!-- end list -->

```
# A slice of a pie chart
slice : PieSlice at (10, 20) with {
  length = (30, 40);
  start = 0;
  extent = 90;
  fill = "purple";
}
```

#### Complex Shapes

  * **Polyline**: Connected straight lines.
  * **Spline**: Connected smooth curved lines.
  * **Polygon**: Closed Polyline.
  * **Blob**: Closed Spline.

<!-- end list -->

```
# A closed polygon with a blue outline
poly : Polygon at (10, 20) with {
  points = [(30, 40), (50, 60), (40, 70)];
  fill = "black";
  outline = "blue";
}
```

### Control Flow

$AG_L$ supports standard procedural control structures.

**Conditionals:**

```
var : Integer = 5;
if var < 10 do {
  print "Low";
} else do {
  print "High";
}
```

**Loops:**

```
# For Loop (Start .. End .. Step)
for i in 0..10..3 {
  print i;
}

# While Loop
while not (a + b >= 10) do {
    a = a + 1;
}

# Repeat-Until
repeat {
    a = a + 1;
} until (a >= 10);
```

-----

## Advanced Modeling

The `Model` keyword allows the creation of composite objects. Models can encapsulate multiple primitives, define internal properties, and define **Actions**.

### Actions & Reactivity

An `action` is a triggered function that executes automatically when a specific property changes. This is essential for creating interactive animations where changing a state variable (e.g., `open`) automatically updates the geometry.

**Example: A Pacman Model**

```
Pacman :: Model {
    # Internal geometry
    face : PieSlice at (0,0) with {
        length = (50,50);
        fill = "pink";
        start = 30;
        extent = 300;
    }

    # State property
    open : Boolean = False;

    # Reactive Logic
    action on open {
        if open do {
          with face do { start = 30; extent = 300; }
        } else do {
          with face do { start = 1; extent = 359; }
        }
    }
}

# Instantiation
player : Pacman;
player.open = True;     # Triggers the action immediately
player.face.fill = "yellow";
```

### Rotation & DeepCopy

$AG_L$ supports deep cloning of complex models and global rotation.

**DeepCopy Demo:**
The `deepcopy` command creates a completely independent instance of an object at a new location.

```
# Create an exact copy of 'ex1' at the origin of 'rec'
ellExtra : Ellipse = deepcopy ex1.ell to ex1.rec.origin;

for i in 1..360 do {
    rotate ex1 by 1;                  # Rotate original model
    move ellExtra to ex1.rec.origin;  # Move copy
    rotate ellExtra by -5;            # Rotate copy independently
    refresh view after 0.01 s;
}
```

Example:

![Rotate.gif](doc/examples/demo/Rotate.gif)

-----

## Scripting with xAGL

$xAG_L$ (extension `.xagl`) is an interpreted language used to control $AG_L$ programs at runtime. It is useful for separating animation logic from object definition.

You can load a script via file or user input and execute it using the `play` command.

**Scripting Demo:**

**Main Program (.agl):**

```
s1 : Script = load "assets/animations/move_logic.xagl";

play s1 with {
    m = object_name;  # Map local object to script variable 'm'
    v = view_name;    # Map local view to script variable 'v'
}
```

**Script File (.xagl):**

```
move m by (100, 0);
refresh v after 500 ms;
m.face.fill = "red"; # Dynamic property access
```

Example:

![Script.gif](doc/examples/demo/xagl1.gif)

-----

## Repository Structure

  * `src/`: Compiler source code and core logic.
  * `src/tests/`: Unit tests and semantic analysis verification.
  * `doc/`: General documentation.
  * `doc/examples/`: Source code for AGL and xAGL examples.
  * `doc/examples/demo/`: Visual output demonstrations (GIFs).
  * `doc/semantic_check.md`: Detailed breakdown of semantic rules.
  * `doc/running.md`: Execution tutorial.

-----

## Contributors

  * Pedro Pinto
  * Giovanni Santos
  * Guilherme Santos
  * João Pinto
  * João Monteiro
  * Jorge Domingues
