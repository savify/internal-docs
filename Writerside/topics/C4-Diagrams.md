# C4 Diagrams

> The **C4 model** is a lean graphical notation technique for modeling the architecture of software systems. It is based on 
> a structural decomposition of a system into containers and components and relies on existing modelling techniques such 
> as the **Unified Modelling Language (UML)** or **Entity Relation Diagrams (ERD)** for the more detailed decomposition of the 
> architectural building blocks. 
> 
> C4 Model defines 4 levels (views) of the system architecture: System Context, Container, Component and Code.
> 
> Read **[C4 Model Documentation](https://c4model.com/)** to get more information about this notation. 

## Workflow

We use **[PlantUML](https://plantuml.com/)** and its **[C4 Model extension](https://github.com/plantuml-stdlib/C4-PlantUML)**
to create and maintain C4 diagrams. You can install it as JetBrains plugin to Writerside IDE.

### Prepare tools

To start maintain C4 diagrams, you should download icon sprites that are used in `puml` files. Run:
```bash
make prepare-c4
```

### Maintenance

`puml` files can be found in `Diagrams/C4` directory. After editing it you should add/replace image in `Writerside/images` 
directory. That's it, so simple :)

## C1 - Context level

This diagram shows the Savify external dependencies and actors.

![C1 Context](c1-context.png)

## C2 - Container level

This diagram shows zoomed in Savify context.

![C2 Context](c2-container.png)

## C3 - Component level

This diagram shows modules inside Savify API and dependencies between them.

![C3 Component](c3-component.png)

## C3 - Component level: Module View

This is more zoomed in Component diagram, that shows what's going on inside single module and how it depends on another 
modules.

![C3 Component Module](c3-component-module.png)