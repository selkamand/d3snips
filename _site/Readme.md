# D3 snippets

Quick Reference D3 snippets

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

### Data bounds: d3.extent (recommended)

Getting the min and max of your data using **d3.extents** to set 'domain' of D3 scales.

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
  .domain(xExtent)
  .range([margin.left, width - margin.right])

const yScale = d3
  .scaleLinear()
  .domain(yExtent)
  .range([height - margin.bottom, margin.top])
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
  x: d.age,
  y: d.height,
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

### Version 2 (use an accessor function + scale function)

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