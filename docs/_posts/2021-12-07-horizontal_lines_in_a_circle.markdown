### Developing a Generative Art Drawing Software

I recently started creating generative art, and I thought it would be a great opportunity for me to improve my programming skills while becoming more comfortable with the OOP paradigm - Until discovering Luigi and Airflow, I worked, largely, with scripts chained together in a functional paradigm. The scripts were powered by a Makefile that tracked and generated/touched various empty .marker files.

My plan is to create a wrapper library over the python package, turtle, that is more intuitive. From the perspective of an artist, the turtle library is lacking. It is very difficult and/or non-intuitive to create many objects. The code reads as a long list of procedures that could or could not be related.

```python
def yin(radius, color1, color2):
    width(3)
    color("black", color1)
    begin_fill()
    circle(radius/2., 180)
    circle(radius, 180)
    left(180)
    circle(-radius/2., 180)
    end_fill()...
```

I'd also like to do things like fill the circle with horizontal lines, or other shapes, draw chords inside the circle, etc. Turtle doesn't have an easy and straightforward way to accomplish this goal. Although you can do it, doing so is a pain.



What I did to solve the problem, is to start creating my own shape classes that could be called with a draw() method. The circle is created by plotting n points from 0-2pi, i.e.

using the relation between Cartesian and polar coordinates, eq.
$$
x = rcos(\theta)
$$

$$
y = rsin(\theta)
$$

to return n (x, y) coordinate points. Below is an example of a simple code block that may be defined, outside of a class, to accomplish this goal.

#### Drawing the Circle with 1000 steps

```python
import numpy as np
import turtle

n_points = 1000
radius = 10
radian_values = np.linspace(0, 2*np.pi, n_points+1) # divides 2pi into 1000 pieces
xy_cors = [(radius*np.cos(rad), radius*np.sin(rad)) for rad in radian_values]
for x, y in xy_cors:
    turtle.goto(x, y)
```

This allows me to simply call each coordinate in a loop to direct the turtle to go to that point.

This also allows me to store a dictionary within the Circle class that contains all of the coordinates for the turtle. The Dictionaries keys correspond to the step int, while the values are another dictionary containing 'x' and 'y' values.

For example, we can update the code above to the following.

#### Storing the Coordinates of the Circle

```python
import numpy as np
import turtle


coordinates = {} # store xy points at each step.
n_points = 1000
radius = 10
radian_values = np.linspace(0, 2*np.pi, n_points+1) # divides 2pi into 1000 pieces
xy_cors = [(radius*np.cos(rad), radius*np.sin(rad)) for rad in radian_values]
for step, (x, y) in enumerate(xy_cors):
    turtle.goto(x, y)
    coordinates[step] = {'x': x, 'y': y}
```

Now we can access the x,y coordinates at any step along the circle. This also allows me to easily be able to connect two points within a circle, draw tangent lines at any point on the circle, and so on and so forth.

In practice the circle is created into a class that is instantiated as so.

```python
import numpy as np
import turtle
from turtle import Turtle
from circle_class import PolarCircle

my_turtle = Turtle()
circle = PolarCircle(turtle, radius, steps=1000, theta_range=2*np.pi, center=(0,0))
```

Some methods I would like to be able to call are

```python
circle.draw() # essentially calls the previous code block to draw/ store coordinates
circle.fill_with_horizontal_lines()
circle.translate()
circle.rotate()
```

The method **fill_with_horizontal_lines** method uses the coordinates of the circle to accomplish this goal. The key is to starting with the longest chord in the circle. How would you fill it will horizontal lines if you only knew the steps along the circle?

#### Filling the Circle with Horizontal Lines

```python
import numpy as np
import turtle


coordinates = {} # store xy points at each step.
n_points = 1000
radius = 10
radian_values = np.linspace(0, 2*np.pi, n_points+1) # divides 2pi into 1000 pieces
xy_cors = [(radius*np.cos(rad), radius*np.sin(rad)) for rad in radian_values]
for step, (x, y) in enumerate(xy_cors):
    turtle.goto(x, y)
    coordinates[step] = {'x': x, 'y': y}

top_half_circle = [step for step in range(0, int(n_points/2)+1)] # 0-500
top_half_circle_reversed = top_half_circle[::-1] # 500 - 0

for step, step_rev in list(zip(top_half_circle, top_half_circle_reversed)):
    x_one =  coordinates[step]['x']
    y_one =  coordinates[step]['y']
    x_two =  coordinates[step_rev]['x']
    y_two =  coordinates[step_rev]['y']
    turtle.penup()
    turtle.goto(x_one, y_one)
    turtle.pendown()
    turtle.goto(x_two, y_two)
```

The code block above draws horizontal lines in the circle by connecting chords within the circle that are parallel to the chord that connects steps 0 and steps 500. the next chord is connecting the steps 1 with steps 499, and so on.

This is simple enough to repeat the same logic with the bottom half of the circle using a range from 500 to 1000.

The issue now is I want to be able to draw lines of any angle within the circle. The code black above can't be easily generalized to accomplish this