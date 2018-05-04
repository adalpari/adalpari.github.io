---
title: "Open/Closed Principle - SOLID part 2"
header:
  teaser: /assets/images/shapes.jpg
  overlay_image: /assets/images/shapes.jpg
---

SOLID is an acronym that groups five basic principles every object oriented programmer should know. This principes set the base to write and maitain a robust code base.

1. [Single responsibility principle.](../2018-04-30-SOLID-S.md)
2. [Open/closed principle.](../2018-05-03-SOLID-O.md)
3. [Liskov substitution principle](../2018-05-04-SOLID-L.md).
4. Interface segregation principle.
5. Dependency inversion principle

In this post, we are going to talk about the second one.

## 2. Open/Closed Principle.

This principle is referring to all classes in out code. It says that every class should be open to expand but close to modify. Not so clear, right?. Let's explain it

Probably you know about inheritance, don't you?. Then, what the principle means is that we should avoid making changes inside any of our class when we are adding new functionallites. But any class should be prepared and ready be extended for that purpose.
This is, essentially, open to be extended but closed to be modified.

Why this principle is good for the code base?.
- We wont introduce new bugs to already working classes when add new features.
- If we don't need to modify our classes when adding new features, means that our classes are well defined.

I have explainded it with inheritance, but there are some authors that prefer interfaces instead of inheritance because the second one escalates in completity much faster (and some other reasson which are not the purpose of this article).

I'm going to do an example with both.

Imagine we have the class shape and a shape drawer:

```java
public class Shape {
	public enum ShapeType { SQUARE, TRIANGLE }

	private ShapeType shapeType;

	public Shape(ShapeType shapeType) {
		this.shapeType = shapeType;
	}

	public ShapeType getType(){
        return shapeType;
    }
}
```
```java
public class ShapeDrawer {
	public void drawShape(Shape shape) {
	    switch (shape.getType()) {
	        case SQUARE:
	            // draw a square
	            break;
	        case TRIANGLE:
	            // draw a triangle
	            break;
	    }
	}
}
```

That code should works fine. But can you see what's wrong?. What would happend if we want to add a new shape?.

Exactly, we should modify all our clases in order to add a new one. So let's refactor it a bit to fit the _Open/Closed_ principle.

```java
public abstract class Shape {
	public abtract void draw();
}
```

```java
public class Square extends Shape {
	@Override
	public void draw() {
		// draw a square
	}
}
```

```java
public class Triangle extends Shape {
	@Override
	public void draw() {
		// draw a triangle
	}
}
```

You can see now how _Shape_ class is abstract and have a method which shuld be implemented by its siblings to draw the shape. Now, _Square_ and _Triangle_ are drawing itselves. We can even add new shapes without modifying the classes we already have:

```java
public class Circle extends Shape {
	@Override
	public void draw() {
		// draw a circle
	}
}
```

Some lines above I mentioned the possibility of follow this principle using interfaces. Personally I prefer use interfaces whenever it's possible.

The idea is use an interface and just create classes that implement it, in order to be able to use them.

```java
public interface DrawableShape {
	public void draw();
}
```

```java
public class Square implements DrawableShape {
	@Override
	public void draw() {
		// draw a square
	}
}
```

```java
public class Triangle implements DrawableShape {
	@Override
	public void draw() {
		// draw a triangle
	}
}
```

```java
public class ShapeDrawer {
	public void drawShape(DrawableShape drawableShape) {
	    drawableShape.draw();
	}
}
```

Simpler, right?