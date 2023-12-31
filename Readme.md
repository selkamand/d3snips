# D3 snippets <!-- omit from toc -->
 
Quick Reference D3 snippets

## Table of Contents

- [Table of Contents](#table-of-contents)
- [Get window dimensions](#get-window-dimensions)
  - [Dynamic](#dynamic)
  - [Static](#static)
- [Create SVG element](#create-svg-element)
- [Margin convention](#margin-convention)
- [Accessing 'columns'](#accessing-columns)
- [Getting Data bounds](#getting-data-bounds)
  - [Data bounds: d3.Min and d3.Max](#data-bounds-d3min-and-d3max)
  - [Data bounds: d3.extent \& nice (recommended)](#data-bounds-d3extent--nice-recommended)
  - [For Categorical Variables](#for-categorical-variables)
- [Drawing Axis](#drawing-axis)
  - [Reusable plot version](#reusable-plot-version)
- [Creating 'marks' data](#creating-marks-data)
  - [Version 1 (simple but inflexible)](#version-1-simple-but-inflexible)
  - [Version 2 (use an accessor function)](#version-2-use-an-accessor-function)
  - [Version 3 (use an accessor function + scale function)](#version-3-use-an-accessor-function--scale-function)
- [Mapping properties to point radius](#mapping-properties-to-point-radius)
- [Draw marks](#draw-marks)
- [Hover and Move events](#hover-and-move-events)
  - [Basics of listeners \& event functions](#basics-of-listeners--event-functions)
  - [Hover: change colour, outline \& renderOrder](#hover-change-colour-outline--renderorder)
- [Reusable Charts](#reusable-charts)
  - [Basics of Method Chaining](#basics-of-method-chaining)
- [Using Groups](#using-groups)
  - [Making new scales](#making-new-scales)
- [Vscode Setup](#vscode-setup)
  - [Webpack and bundling](#webpack-and-bundling)
  - [vs-code snippets](#vs-code-snippets)
    - [Getters and Setters with Method Chaining](#getters-and-setters-with-method-chaining)
- [Datasets](#datasets)
  - [Test Dataset](#test-dataset)
- [HTML](#html)
  - [Removing overflow and margins](#removing-overflow-and-margins)
- [Miscellaneous Javascript Tips](#miscellaneous-javascript-tips)
  - [Setting up unit testing](#setting-up-unit-testing)


## Get window dimensions

### Dynamic
```{javascript}
// Get Window Dimensions
const width = window.innerWidth;
const height = window.innerHeight;
```

### Static

```{javascript}
// Alternative for fixed size
const width = 960
const height = 500
```


## Create SVG element
```{javascript}
// Create SVG 
const svg = d3
  .select("body")
  .append("svg")
  .attr("width", width)
  .attr("height", height);

```

## Margin convention

In the SVG coordinate system the origin (0,0) is the top-left corner

```{javascript}
// Create Margin Object
const margin = {
  top: 20,
  right: 20,
  bottom: 30,
  left: 40,
};

// Example of how to use in scales to define range
const xScale = d3
  .scaleLinear()
  .range([margin.left, width - margin.right])

const yScale = d3
  .scaleLinear()
  .range([height - margin.bottom, margin.top])
```



## Accessing 'columns'
Accessing data 'columns' can be surprisingly complicated for newcomers, since most data is defined as an array of objects (e.g. 'Test dataset' below). 

For D3 its usually best to define an 'accessorFunction' that takes an object (like each row in data)

```{javascript}
// Define dataset
const data = [
  { x: 1, y: 10 },
  { x: 2, y: 20 },
  { x: 3, y: 15 },
  { x: 4, y: 25 },
  { x: 5, y: 18 },
];

// Get all values of x as an array
const x_array = data.map(d => d.x)

// Compute Minumum using D3
const xMin = d3.min(x_array)

// Alternatively, define an accessorFunction for your X and or Y axis that works with d3 functions like d3.min

// Create accessor
xAccessor = d => d.x

// Compute Minimum
const xMin = d3.min(data, xAccessor)
```

## Getting Data bounds

Getting the min and max of your data to set the 'domain' of D3 scales.

### Data bounds: d3.Min and d3.Max

Getting the min and max of your data using **d3.min** and **d3.max** to set 'domain' of D3 scales.

```{javascript}
// Define dataset
const data = [
  { x: 1, y: 10 },
  { x: 2, y: 20 },
  { x: 3, y: 15 },
  { x: 4, y: 25 },
  { x: 5, y: 18 },
];

// Create accessor
xAccessor = d => d.x
yAccessor = d => d.y

// Calculate Min
const xMin = d3.min(data, xAccessor)
const xMax = d3.max(data, xAccessor)
const yMin = d3.min(data, yAccessor)
const yMax = d3.max(data, yAccessor)

// Define Scales
const xScale = d3
  .scaleLinear()
  .domain([xMin, xMax])
  .range([margin.left, width - margin.right])

const yScale = d3
  .scaleLinear()
  .domain([yMin, yMax])
  .range([height - margin.bottom, margin.top])
```

### Data bounds: d3.extent & nice (recommended)

Getting the min and max of your data using **d3.extents** to set 'domain' of D3 scales.
This function returns the min and max values of the array. You can also add `.nice()` to the domain calls to round the extreme values (floor the min and ceil the max) and make the axes look nicer

```{javascript}
// Define dataset
const data = [
  { x: 1, y: 10 },
  { x: 2, y: 20 },
  { x: 3, y: 15 },
  { x: 4, y: 25 },
  { x: 5, y: 18 },
];

// Create accessor
xAccessor = d => d.x
yAccessor = d => d.y

// Calculate Min
const xExtent = d3.extent(data, xAccessor)
const yExtent = d3.extent(data, yAccessor)

// Define Scales
const xScale = d3
  .scaleLinear()
  .domain(xExtent).nice()
  .range([margin.left, width - margin.right])

const yScale = d3
  .scaleLinear()
  .domain(yExtent).nice()
  .range([height - margin.bottom, margin.top])
```

### For Categorical Variables

The domain of categorical axes defined using scaleBand, the 'extent' of the data (i.e. domain) is a unique array of all possible values. These can be defined from the data as follows:


```{javascript}
const data = [
  { x: 'A', y: 10 },
  { x: 'A', y: 20 },
  { x: 'B', y: 15 },
  { x: 'C', y: 25 },
  { x: 'C', y: 18 },
];

// Get Unique values for X and Y axes
const uniqueX = [...new Set(data.map((item) => item.x))];

const xScale = d3
  .scaleBand()
  .domain(uniqueX);
```

However its worth noting, the d3 .domain function automatically makes inputs unique, so you can just input the whole array and let d3 handle making the values unique

```{javascript}
// D3 .domain method automatically coercess inputs to be unique
const xScale = d3
  .scaleBand()
  .domain(data.map(d => d.x));
```



## Drawing Axis

Once you've created scales using d3.scaleLinear you actually have to draw them

```{javascript}
// Create Axes From scales (e.g. using d3.scaleLinear())
const xAxis = d3.axisBottom(xScale);
const yAxis = d3.axisLeft(yScale);


// Append axes to svg
svg
  .append("g")
  .attr("class", "x axis")
  .attr("transform", `translate(0, ${height - margin.bottom})`)
  .call(xAxis);


// Append y axis to svg
svg
  .append("g")
  .attr("class", "y axis")
  .attr("transform", `translate(${margin.left}, 0)`)
  .call(yAxis);

```

### Reusable plot version

If drawing plots in a function that might be called multiple times, you'll have to drop all 'append' calls (otherwise the axes will be drawn over the top of one another each time function is called).

Instead, use d3 selectAll/join pattern with a single length data array to update existing axes each time the plotting wrapper function is called.


```{javascript}
const xAxis = d3.axisBottom(xScale);
const yAxis = d3.axisLeft(yScale);

svg
  .selectAll(".x-axis")
  .data([null])
  .join("g")
  .attr("class", "x-axis")
  .attr("transform", `translate(0, ${height - margin.bottom})`)
  .call(xAxis);

selection
  .selectAll('.y-axis')
  .data([null])
  .join('g')
  .attr('class', 'y-axis')
  .attr('transform', `translate(${margin.left},0)`)
  .call(yAxis)

```

## Creating 'marks' data

A great d3 convention is to create a marks array of objects that maps your data values to aesthetic properties (xpos & ypos, color, etc).

### Version 1 (simple but inflexible)

```{javascript}
// Define dataset
const data = [
  { age: 1, height: 50 },
  { age: 2, height: 60 },
  { age: 3, height: 70 },
  { age: 4, height: 80 },
  { age: 5, height: 90 },
];

// Create marks array
const marks = data.map((d) => ({
  xpos: d.age,
  ypos: d.height,
}));
```

### Version 2 (use an accessor function)

Improved flexibility  have already defined an accessor function

```{javascript}
// Define dataset
const data = [
  { age: 1, height: 50 },
  { age: 2, height: 60 },
  { age: 3, height: 70 },
  { age: 4, height: 80 },
  { age: 5, height: 90 },
];

// Accessor functions
const xAccessor = (d) => d.age;
const yAccessor = (d) => d.height;


// Create marks array
const marks = data.map((d) => ({
  x: xAccessor(d),
  y: yAccessor(d),
}));
```

### Version 3 (use an accessor function + scale function)

We can add a previously defined scale function to automatically rescale data to pixel values using scales such as d3.linearScale()

```{javascript}
// Define dataset
const data = [
  { age: 1, height: 50 },
  { age: 2, height: 60 },
  { age: 3, height: 70 },
  { age: 4, height: 80 },
  { age: 5, height: 90 },
];

// Accessor functions
const xAccessor = (d) => d.age;
const yAccessor = (d) => d.height;

// create margins
const margin = {
  top: 20,
  right: 20,
  bottom: 30,
  left: 40,
};

// Create scales
const xScale = d3
  .scaleLinear()
  .range([margin.left, width - margin.right])
  .domain([xDomainMin, xDomainMax]);

const yScale = d3
  .scaleLinear()
  .range([height - margin.bottom, margin.top])
  .domain([yDomainMin, yDomainMax]);

// Create marks array
const marks = data.map((d) => ({
  x: xScale(xAccessor(d)),
  y: yScale(yAccessor(d)),
}));
```

## Mapping properties to point radius

When mapping a numerical value to size of a point in d3, its best practice to ensure area of the point is linearly related to the size of the data. Because area of a circle is pi*r^2 we should actually use d3.scaleSqrt()

```{javascript}
d3
.scaleSqrt()
.domain([0, maxDataValue]) 
.range([0, maxRadius]) // Where maxRadius = the maximum radius possible for the circle
```

Some important notes. The above system of mapping follows a very conservative philosophy that we want to map area size to values as directly as possible. If a value is 0 or negative, then we wont even see the point on the plot. This can be considered honest representations of the data.

An alternative approach is to always scale the values between variables. This will make sure the sqrt of data gets mapped to a radius of between 1 and 5. Even negative values will be shown on the plot and the size will always vary. Its like showing a heatmap with a relative scale. Depending on the situation, and what you're trying to show, it can be bad practice.

```{javascript}
minRadius = 1
maxRadius = 5
const radiusScale = d3
  .scaleSqrt()
  .domain(d3.extent(data, radiusAccessor))
  .range([minRadius, maxRadius]);
```

## Draw marks

Draw marks

```{javascript}
// draw marks
svg
  .selectAll("circle")
  .data(marks)
  .join("circle")
  .attr("cx", (d) => d.xpos)
  .attr("cy", (d) => d.ypos)
  .attr("r", (d) => d.radius);
```

## Hover and Move events

### Basics of listeners & event functions

To trigger events (e.g. tooltip rendering) on mouse on / mousemove / mouseleave etc, you can use the **.on** method on a d3 selection to add a listener that triggers some function on whichever events you are interested in.

Within event functions, you can figure out what value syou have access to using the MDN documentation (e.g. for [mouseover]([https://developer.mozilla.org/en-US/docs/Web/API/Element/mouseover_event])). The most important properties include **target**

```{javascript}
// Define functions that run when events are triggered
const mouseover = (event) => {
  console.log("mouseover");
  // Run whatever you want
};
const mousemove = (event) => {
  console.log("mousemove");
};
const mouseleave = (event) => {
  console.log("mouseleave");
};


// Add use .on('') to add triggers to points when drawing your marks

svg
  .selectAll("circle")
  .data(marks)
  .join("circle")
  .attr("cx", (d) => d.xpos)
  .attr("cy", (d) => d.ypos)
  .attr("r", (d) => d.radius)
  // Mouse over methods
  .on("mouseover", mouseover)
  .on("mousemove", mousemove)
  .on("mouseleave", mouseleave);
```

So how do we use this to create cool effects?

### Hover: change colour, outline & renderOrder

We may want to change fill colour of a point when hovered over, and revert when not.

```{javascript}
// Define functions that run when events are triggered
const mouseover = (event) => {
  const target = event.target

  target.style.fill = "purple" 
  target.style.stroke = "black";
  target.style.strokeWidth = "8pt";

  // Move element to end of the parent group so it renders on`` top of the others
  const parent = event.target.parentElement;
  parent.appendChild(event.target);
};

const mouseleave = (event) => {
  const target = event.target
  target.style.fill = target.getAttribute("fill");
  target.style.strokeWidth = target.getAttribute("strokeWidth");
  target.style.stroke = target.getAttribute("stroke");
};

// Add use .on('') to add triggers to points when drawing your marks
svg
  .selectAll("circle")
  .data(marks)
  .join("circle")
  .attr("cx", (d) => d.xpos)
  .attr("cy", (d) => d.ypos)
  .attr("r", (d) => d.radius)
  .attr("fill", "black")
  // Mouse methods
  .on("mouseover", mouseover)
  .on("mouseleave", mouseleave);
```

Note that mouseleave works since the attribute 'fill' stays the same once set by `.attr("fill", "black")` even when we change the style. It is the style that controls how it looks in the end. 
``
We can use this as a general pattern to change/revert any aesthetic property on hover.

## Reusable Charts

### Basics of Method Chaining

To understand the following example, you must understand a couple of javascript nuances

1. Functions are objects. For example the `scatterPlot` function returns a function `my`. This function its own methods (e.g. getter setters), each of which return the `my` function

```
export const scatterPlot = () => {
  // Properties
  let width;
  let height;
  let data;
  let xValue;
  let yValue;
  let margin;
  let radius;


  const my = (selection) => {
    // Create Chart
  };


  // Getter and setter for 'width' property
  // If an argument is provided, set 'width' to the provided value and return 'my' for method chaining. 
  // If no width is supplied return the current value of width

  my.width = function (_) {
    return arguments.length ? ((width = +_), my) : width;
  };

  my.height = function (_) {
    return arguments.length ? ((height = +_), my) : height;
  };

  my.data = function (_) {
    return arguments.length ? ((data = _), my) : data;
  };

  my.xValue = function (_) {
    return arguments.length ? ((xValue = _), my) : xValue;
  };

  my.yValue = function (_) {
    return arguments.length ? ((yValue = _), my) : yValue;
  };

  my.margin = function (_) {
    return arguments.length ? ((margin = _), my) : margin;
  };

  my.radius = function (_) {
    return arguments.length ? ((radius = +_), my) : radius;
  };

  //return function for method chaining
  return my;
};


```

## Using Groups

In D3 you can create 'group' elements that you can append loads of elements to. This allows you to move all the elements around as a group using the 'transform' attribute of the parent group. Child x and y positions are generated relative to the parent group.

See the example below.


```
    const circlesData = [
      { cx: 0, cy: 0, r: 20 },
      { cx: 50, cy: 50, r: 20 },
      { cx: 100, cy: 100, r: 30 },
      { cx: 150, cy: 150, r: 25 }
    ];

    // Create an SVG element
    const svg = d3.select("svg");

    // Create a group element and append it to the SVG
    const group = svg.append("g")
                     .attr("transform", "translate(50, 50)"); // Move the group 50px to the right and 50px down

    // Create circles and append them to the group
    group.selectAll("circle")
         .data(circlesData)
         .join("circle")
         .attr("cx", d => d.cx)
         .attr("cy", d => d.cy)
         .attr("r", d => d.r)
         .attr("fill", "steelblue");
```

You can define an expression to evaluate in the translate call:

```
const x = 50, y = 50;
const group = svg.append("g")
                     .attr("transform", `translate( ${x},${y})`); // Move the group 50px to the right and 50px down

```


### Making new scales

Use the example above but the `my` function needs to take as input values and output pixel values

## Vscode Setup

### Webpack and bundling

Get javascript imports and es6 modules working using webpack and babel.

1. Run `npm init -y` to initialise your project
2. Run `npm install webpack webpack-cli --save-dev`
3. Install any other dependencies you want (e.g. `npm install d3`)
4. Install webpack extension for vscode if you dont already have it
5. Run: Create webpack. Should create a webpack.config.js file
6. Edit webpadk.config.js and set the entrypoint. If your using an index.js entrypoint, set it to: 
   `entry: path.join(__dirname, "index"),`
7. Edit your packages.json and ensure `"main": "index.js"` and **build** is set to "webpack" (see below)

``` 
"scripts": {
    "build": "webpack",
    "test": "echo \"Error: no test specified\" && exit 1"
  }
```

5. Run: `npm run build` to create your bundle.js
6. Ensure your index.html is pointed towards your `bundle.js file`

```
 <script src="./dist/bundle.js"></script>
```

Once you run `npm run build` webpack will keep running and regenerate your bundle.js on-save so you can work as normal


### vs-code snippets
For convenience: heres some snippets to add to your vscode javascript.json file

#### Getters and Setters with Method Chaining

```
	"getter & setter for method chaining": {
		"prefix": "getset",
		"body": [
			"my.${1:property} = function (_) {",
			"\treturn arguments.length ? ((${1:property} = _), my) : ${1:property};",
			"};"
		],
		"description": "Getter and setter for a method chaining function"
	},

	"Longform getter & setter for method chaining": {
		"prefix": "getsetlong",
		"body": [
			"my.${1:property} = function (_) {",
			"\tif (!arguments.length) return ${1:property};",
			"\t${1:property} = _;",
			"\treturn my",
			"};"
		],
		"description": "Longform getter and setter for a method chaining function"
	},
	
	"Getter": {
		"prefix": "get",
		"body": [
			"my.${1:property} = function () {",
			"\treturn ${1:property};",
			"};"
		],
		"description": "Getter"
	}
```


## Datasets

### Test Dataset

This test dataset includes both categorical and quantitative features.
correlatedVar and antiCorrelatedVar are relative to age.

```{javascript}
// Dataset
// prettier-ignore
const data = [
    { id: 1, name: 'Person1', age: 30, correlatedVar: 42, antiCorrelatedVar: 72, gender: 'Male', city: 'New York' },
    { id: 2, name: 'Person2', age: 25, correlatedVar: 37, antiCorrelatedVar: 77, gender: 'Female', city: 'Los Angeles' },
    { id: 3, name: 'Person3', age: 40, correlatedVar: 54, antiCorrelatedVar: 60, gender: 'Male', city: 'Chicago' },
    { id: 4, name: 'Person4', age: 33, correlatedVar: 42, antiCorrelatedVar: 67, gender: 'Female', city: 'Houston' },
    { id: 5, name: 'Person5', age: 29, correlatedVar: 37, antiCorrelatedVar: 75, gender: 'Male', city: 'Miami' },
    { id: 6, name: 'Person6', age: 35, correlatedVar: 42, antiCorrelatedVar: 65, gender: 'Female', city: 'New York' },
    { id: 7, name: 'Person7', age: 22, correlatedVar: 35, antiCorrelatedVar: 78, gender: 'Male', city: 'Los Angeles' },
    { id: 8, name: 'Person8', age: 47, correlatedVar: 59, antiCorrelatedVar: 53, gender: 'Female', city: 'Chicago' },
    { id: 9, name: 'Person9', age: 39, correlatedVar: 48, antiCorrelatedVar: 61, gender: 'Male', city: 'Houston' },
    { id: 10, name: 'Person10', age: 41, correlatedVar: 54, antiCorrelatedVar: 59, gender: 'Female', city: 'Miami' },
    { id: 11, name: 'Person41', age: 23, correlatedVar: 35, antiCorrelatedVar: 87, gender: 'Male', city: 'Los Angeles' },
    { id: 12, name: 'Person42', age: 51, correlatedVar: 61, antiCorrelatedVar: 55, gender: 'Female', city: 'Chicago' },
    { id: 13, name: 'Person43', age: 36, correlatedVar: 46, antiCorrelatedVar: 66, gender: 'Male', city: 'Houston' },
    { id: 14, name: 'Person44', age: 42, correlatedVar: 49, antiCorrelatedVar: 61, gender: 'Female', city: 'Miami' },
    { id: 15, name: 'Person45', age: 38, correlatedVar: 46, antiCorrelatedVar: 62, gender: 'Male', city: 'New York' },
    { id: 16, name: 'Person46', age: 32, correlatedVar: 42, antiCorrelatedVar: 68, gender: 'Female', city: 'Los Angeles' },
    { id: 17, name: 'Person47', age: 26, correlatedVar: 34, antiCorrelatedVar: 79, gender: 'Male', city: 'Chicago' },
    { id: 18, name: 'Person48', age: 28, correlatedVar: 40, antiCorrelatedVar: 78, gender: 'Female', city: 'Houston' },
    { id: 19, name: 'Person49', age: 33, correlatedVar: 42, antiCorrelatedVar: 72, gender: 'Male', city: 'Miami' },
    { id: 20, name: 'Person50', age: 45, correlatedVar: 57, antiCorrelatedVar: 55, gender: 'Female', city: 'New York' }
  ];
```

## HTML

### Removing overflow and margins

To remove scroll bars and html default margins around svg elements add the following to your html head

```{html}
  <style>
      body {
        margin: 0;
        overflow: hidden;
      }
    </style>
```


## Miscellaneous Javascript Tips

### Setting up unit testing

1. npm install jest
2. Set 'test' script in package.json to 'jest': `"test": "jest --coverage"`
3. If using es6 modules & webpack you will need to make additional changes: https://jestjs.io/docs/getting-started#using-webpack
