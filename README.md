# Nest4J

Nest4J is a nesting algorithm library. It's a Java adaptation of [SVGNest](https://github.com/Jack000/SVGnest).

# What is "nesting"?

Imagine you've got a rectangular sheet and a bunch of letter-shaped pieces — how do you fit all those letters onto the sheet without overlapping? By finding just the right order and position for each letter, along with the perfect angle to rotate them, we get the job done. This kind of puzzle — figuring out how the letters and the sheet fit together and at what angles — is what we call a nesting problem.

In the CNC world this is called "nesting", and software that does this is typically targeted at industrial customers and very expensive.

![example](./png/nest.png)

## Performance
I used SVGNest Demo to test Nest4J and here is my result.

![sample](./png/sample.png)

## Usage

This algorithm, based on SVGNest, has been tweaked to run in Java so it can handle backend calculations on a server.

### Representing polygons

Nest4J uses a standard way of representing a polygon by a set of points. Here's how you make a rectangular polygon shape.

**Keep in mind, Nest4J polygons are based on a 2D Cartesian coordinate system, which means you need to ensure that the polygons you input won't overlap, or it'll throw an error.**

```java
NestPath bin = new NestPath();
double binWidth = 511.822;
double binHeight = 339.235;
bin.add(0, 0);
bin.add(binWidth, 0);
bin.add(binWidth, binHeight);
bin.add(0, binHeight);
```


### Making sheet materials

Once we know how to make a polygon, making a collection of bins is basically making a collection of polygons.

```java
List<NestPath> list = new ArrayList<NestPath>();
list.add(polygon1);
list.add(polygon2);
list.add(polygon3);
```

### Extended options for polygons

By default, a polygon has a rotation property of 0, which means it's locked in place and can't rotate during nesting. To let it rotate, set its rotation property to 4, which allows it to rotate by 90°, 180°, and 270° during nesting. In general, if you set a polygon's rotation to N, you're giving it N options for rotation: (360/N)*k , k= 0,1,2,3,...N.

You can also set a bid (bin id) property for each sheet, which helps match them up before and after nesting.

``` java
polygon.bid = id;
polygon.setRotation(4);
``` 

### Polygons with holes

Representing a polygon with holes in Nest4J is also very simple; as long as the shape of the holes is described using a two-dimensional Cartesian coordinate system, they can be placed within the interior of the polygon. Nest4J will automatically detect the presence of holes in the polygon.

```java
NestPath outer = new NestPath();
outer.add(600, 0);
outer.add(600, 200);
outer.add(800, 200);
outer.add(800, 0);
outer.setRotation(0);
outer.bid = 1;
NestPath inner = new NestPath();
inner.add(650, 50);
inner.add(650, 150);
inner.add(750, 150);
inner.add(750, 50);
inner.bid = 2;
```


### Configuration 

You can customize a bunch of settings in Nest4J prior to starting the nesting calculation.

```java
Config config = new Config();
config.SPACING = 0;
config.POPULATION_SIZE = 5;
```


<table>
    <tr>
        <td>Attr</td>
        <td>Description</td>
        <td>Default</td>
    </tr>
    <tr>
        <td>SPACING</td>
        <td>the distance of each plygons on bin</td>
        <td>0</td>
    </tr>
    <tr>
        <td>POPULATION_SIZE</td>
        <td>the number of population in GA algorithm</td>
        <td>10</td>
    </tr>
    <tr>
        <td>MUTATION_RATE</td>
        <td>the rate of mutate in GA algorithm</td>
        <td>10%</td>
    </tr> 
    <tr>
        <td>USE_HOLE</td>
        <td>allow to put polygons into hollow polygons</td>
        <td>false</td>
    </tr>     
</table>


## Starting the calculation

Once the base sheet, polygon collection, and necessary parameters have been set, along with the number of iterations we want, we're ready to calculate.

```java
Nest nest = new Nest(bin, polygons, config, 2);
List<List<Placement>> appliedPlacement = nest.startNest();
```


### Placement

Placement represents a unit of our final result, indicating the offset and rotation angle of a polygon (identified by bid) relative to the top-left corner of its belonging bin.

```java
public class Placement {
    public int bid;
    public Segment translate;
    public double rotate;


    public Placement(int bid, Segment translate, double rotate) {
        this.bid = bid;
        this.translate = translate;
        this.rotate = rotate;
    }

    public Placement() {
    }
}
```

## Visualization 

To output results, I've provided a method based on SVG for visualization. You can take a look in the NestTest.

```java
List<String> strings = SvgUtil.svgGenerator(polygons, appliedPlacement, binWidth, binHeight);
saveSvgFile(strings);
```


## Reference Papers

- [López-Camacho *et al.* 2013](http://www.cs.stir.ac.uk/~goc/papers/EffectiveHueristic2DAOR2013.pdf)
- [Kendall 2000](http://www.graham-kendall.com/papers/k2001.pdf)
- [E.K. Burke *et al.* 2006](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.440.379&rep=rep1&type=pdf)


## Todo

1. make Nest4J process more parallel.


