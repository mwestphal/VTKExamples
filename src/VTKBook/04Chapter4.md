# Chapter 4 - The Visualization Pipeline

**I**n the previous chapter we created graphical images using simple mathematical models for lighting, viewing, and geometry. The lighting model included ambient, diffuse, and specular effects. Viewing included the effects of perspective and projection. Geometry was defined as a static collection of graphics primitives such as points and polygons. In order to describe the process of visualization we need to extend our understanding of geometry to include more complex forms. We will see that the visualization process transforms data into graphics primitives. This chapter examines the process of data transformation and devel-ops a model of data flow for visualization systems.

## 4.1 Overview

Visualization transforms data into images that efficiently and accurately convey information about the data. Thus, visualization addresses the issues of *transformation* and *representation*.

Transformation is the process of converting data from its original form into graphics primitives, and eventually into computer images. This is our working definition of the visualization process. An example of such a transformation is the process of extracting stock prices and creating an *x-y* plot depicting stock price as a function of time.

Representation includes both the internal data structures used to depict the data and the graphics primitives used to display the data. For example, an array of stock prices and an array of times are the computational representation of the data, while the *x-y* plot is the graphical representation. Visualization transforms a computational form into a graphical form.

From an object-oriented viewpoint, transformations are processes in the functional model, while representations are the objects in the object model. Therefore, we characterize the visualization model with both functional models and object models.

**A Data Visualization Example**

A simple mathematical function for a quadric will clarify these concepts. The function

$$
\begin{equation*}
F(x,y,z) = a_0 x^2 + a_1 y^2 + a_2 z^2 + a_3 x y + a_4 y z + a_5 x z + a_6 x + a_7 y + a_8 z + a_9
\end{equation*}
\bf\tag{4-1}
$$

is the mathematical representation of a quadric. **Figure 4–1(a)** shows a visualization of **Equation 4-1** in the region $–1 \leq x, y, z \leq 1$. The visualization process is as follows. We sample the data on a regular grid at a resolution of $50 \times 50 \times 50$. Three different visualization techniques are then used. On the left, we generate 3D surfaces corresponding to the function $F(x, y, z) = c$ where c is an arbitrary constant (i.e., the isosurface value). In the center, we show three different planes that cut through the data and are colored by function value. On the right we show the same three planes that have been contoured with constant valued lines. Around each we place a wireframe outline.

** The Functional Model**

The functional model in Figure 4–1(b) illustrates the steps to create the visualization. The oval blocks indicate operations (processes) we performed on the data, and the rectangular blocks represent data stores (objects) that represent and provide access to data. Arrows indicate the direction of data movement. Arrows that point into a block are inputs; data flowing out of a block indicate outputs. The blocks also may have local parameters that serve as additional input. Processes that create data with no input are called data source objects, or simply sources. Processes that consume data with no output are called sinks (the are also called mappers because these processes map data to a final image or output). Processes with both an input and an output are called filters. The functional model shows how data flows through the system. It also describes the dependency of the various parts upon one another. For any given process to execute correctly, all the inputs must be up to date. This suggests that functional models require a synchronization mechanism to insure that the correct output will be generated.

The functional model shows how data flows through the system. It also describes the depen-dency of the various parts upon one another. For any given process to execute correctly, all the inputs must be up to date. This suggests that functional models require a synchronization mecha-nism to insure that the correct output will be generated.

**The Visualization Model**

In the examples that follow we will frequently use a simplified representation of the functional model to describe visualization processes ( **Figure 4-1** (c)). We will not explicitly distinguish between sources, sinks, data stores, and process objects. Sources and sinks are implied based on the number of inputs or outputs. Sources will be process objects with no input. Sinks will be process objects with no output. Filters will be process objects with at least one input and one output. Inter-mediate data stores will not be represented. Instead we will assume that they exist as necessary to support the data flow. Thus, as **Figure 4-1** (c) shows, the *Lines* data store that the *Outline* object generates ( **Figure 4-1** (b)) are combined into the single object *Outline*. We use oval shapes to repre-sent objects in the visualization model. (b) Functional model *Sample F(x,y,z)*

<figure id="Figure 4-1">
 <figure id="Figure 4-1"a>
  <img src="https://raw.githubusercontent.com/lorensen/VTKExamples/master/src/Testing/Baseline/Cxx/Visualization/TestQuadricVisualization.png?raw=true width="640" alt="Figure 4-1a">
  <figcaption style="color:blue"><b>Figure 4-1</b>. Visualizing a quadric function *F(x,y,z) = c*. <a href="../../Cxx/Visualization/QuadricVisualization" title="QuadricVisualization"> See QuadricVisualization.cxx</a> and <a href="../../Python/Visualization/QuadricVisualization" title="QuadricVisualization"> QuadricVisualization.py</a>.</figcaption>
  </figure>
  <figure id="Figure 4-1b">
  <img src="https://raw.githubusercontent.com/lorensen/VTKExamples/master/src/VTKBook/Figures/Figure4-1bc.png?raw=true width="640" alt="Figure4-1bc">
  </figure>
<figcaption style="color:blue"><b>Figure 4-1</b>. Visualizing a quadric function F(x,y,z) = c..</figcaption>
</figure>

<figure id="Figure 4-2">
  <img src="https://raw.githubusercontent.com/lorensen/VTKExamples/master/src/VTKBook/Figures/Figure4-2.png?raw=true width="640" alt="Figure4-2">
</figure>
<figcaption style="color:blue"><b>Figure 4-2</b>. Object model design choices. One basic choice is to combine processes and data stores into a single object. This is the usual object-oriented choice. Another choice creates separate data objects and process objects.</figcaption>
</figure>

**The Object Model**

The functional model describes the flow of data in our visualization,
the object model describes which modules operate on it. But what *are*
the objects in the system? At first glance, we have two choices (**Figure 4-2**).

The first choice combines data stores (object attributes) witha processes (object methods) into a single object. In the second choice we use separate objects for data stores and processes. There is actually a third alternative: a hybrid combination of these two choices.

The conventional object-oriented approach (our first choice above) combines data stores and processes into a single object. This view follows the standard definition that objects contain a data representation combined with procedures to operate on the data. One advantage of this approach is that the processes, which are the data visualization algorithms, have complete access to the data structures, resulting in good computational performance. But this choice suffers from several draw-backs.

-   From a user's perspective, processes are often viewed as independent of data representation. In other words, processes are naturally viewed as objects in the system. For example, we often say we want to "contour" data, meaning creating lines or surfaces corresponding to a constant data value. To the user it is convenient to have a single contour object to operate on different data representations.

-   We must duplicate algorithm implementation. As in the previous contouring example, if we bind data stores and processes into a single object, the contour operation must be recreated for each data type. This results in duplicating code even though the implementations of an algorithm may be functionally and structurally similar. Modifying such algorithms also means modifying a large amount of code, since they are implemented across many objects.

-   Binding data stores and algorithms together results in complex, data dependent code. Some algorithms may be much more complex than the data they operate on, with large numbers of instance variables and elaborate data structures. By combining many such algorithms with a data store, the complexity of the object greatly increases, and the simple meaning of the object becomes lost.

The second choice separates the data stores and processes. That is, one set of objects represents and provides access to the data, while another set of objects implements all operations on the data. Our experience shows that this is natural to users, although it may be considered unconventional to the object-oriented purist. We also have found that the resulting code is simple, modular, and easy for developers to understand, maintain, and extend.

One disadvantage to the second choice is that the interface between data representation and process is more formal. Thus the interface must be carefully designed to insure good performance and flexibility. Another disadvantage is that strong separation of data and process results in duplicate code. That is, we may implement operations that duplicate algorithms and that cannot be con-sidered strictly data access methods. One example of such a situation is computing data derivatives. This operation is more than simple data access, so strictly speaking it doesn't belong in the data object methods. So to compute derivatives we would have to duplicate the code each time we needed derivatives computed. (Or create a procedural library of functions or macros!)

As a result of these concerns we use the hybrid approach in the *Visualization Toolkit* . Our approach is closest to the second choice described above, but we have selected a small set of critical operations that we implement within the data objects. These operations have been identified based on our experience implementing visualization algorithms. This effectively combines the first two choices to receive the maximum benefit and fewest disadvantages of each.

## 4.2 The Visualization Pipeline

In the context of data visualization, the functional model of **Figure 4-1** (c) is referred to as the *visualization pipeline* or *visualization network* . The pipeline consists of objects to represent data (data objects), objects to operate on data (process objects), and an indicated direction of data flow (arrow connections between objects). In the text that follows, we will frequently use visualization networks to describe the implementation of a particular visualization technique.

**Data Objects**

*Data objects* represent information. Data objects also provide methods to create, access, and delete this information. Direct modification of the data represented by the data objects is not allowed except through formal object methods. This capability is reserved for process objects. Additional methods are also available to obtain characteristic features of the data. This includes determining the minimum and maximum data values, or determining the size or the number of data values in the object.

Data objects differ depending upon their internal representation. The internal representation has significant impact on the access methods to the data, as well as on the storage efficiency or computational performance of process objects that interact with the data object. Hence, different data objects may be used to represent the same data depending on demands for efficiency and pro-cess generality.

**Process Objects**

*Process objects* operate on input data to generate output data. A process object either derives new data from its inputs, or transforms the input data into a new form. For example, a process object might derive pressure gradient data from a pressure field or transform the pressure field into constant value pressure contours. The input to a process object includes both one or more data objects as well as local parameters to control its operation. Local parameters include both instance variables or associations and references to other objects. For example, the center and radius are local parameters to control the generation of sphere primitives.

Process objects are further characterized as *source objects* , *filter objects* , or *mapper objects* . This categorization is based on whether the objects initiate, maintain, or terminate visualization data flow.

Source objects interface to external data sources or generate data from local parameters. Source objects that generate data from local parameters are called *procedural* objects. The previous example of **Figure 4-1** uses a procedural object to generate function values for the quadric function of **Equation4-1** . Source objects that interface to external data are called *reader* objects since the external file must be read and converted to an internal form. Source objects may also interface to external data communication ports and devices. Possible examples include simulation or modelling programs, or data acquisition systems to measure temperature, pressure, or other similar physical attributes.

Filter objects require one or more input data objects and generate one or more output data objects. Local parameters control the operation of the process object. Computing weekly stock market averages, representing a data value as a scaled icon, or performing union set operations on two input data sources are typical example processes of filter objects.

Mapper objects correspond to the sinks in the functional model. Mapper objects require one or more input data objects and terminate the visualization pipeline data flow. Usually mapper objects are used to convert data into graphical primitives, but they may write out data to a file or interface with another software system or devices. Mapper objects that write data to a computer file are termed *writer* objects.

## 4.3 Pipeline Topology

In this section we describe how to connect data and process objects to form visualization networks.

**Pipeline Connections**

The elements of the pipeline (sources, filters, and mappers) can be
connected in a variety of ways to create visualization networks.
However, there are two important issues that arise when we try to
assemble these networks: *type* and *multiplicity* .

Type means the form or type of data that process objects take as input
or generate as output. For example, a sphere source object may
generate as output a polygonal or faceted representation, an implicit
representation (e.g., parameters of a conic equation), or a set of
occupancy values in a discretized representation of 3D space. Mapper
objects might take as input polygonal, triangle strip, line, or point
geometric representations. The input to a process object must be
specified correctly for successful operation.

<figure id="Figure 4-3">
  <img src="https://raw.githubusercontent.com/lorensen/VTKExamples/master/src/VTKBook/Figures/Figure4-3.png?raw=true width="640" alt="Figure4-3">
</figure>
<figcaption style="color:blue"><b>Figure 4-3</b>. Maintaining compatible data type. (a) Single-type systems require no type checking. </figcaption>
</figure>

In multiple-type systems only compatible types can be connected
together.

There are two general approaches to maintain proper input type. One
approach is to design with type-less or single-type systems. That is, create a single type
of data object and create filters that operate only on this one type ( **Figure 4-3** (a)). For example,
we could design a general *DataSet* that represents any form of data that we're interested in, and the
process objects would only input *DataSets* and generate *DataSets* . This approach is simple and
elegant, but inflexible. Often, particularly useful algorithms (i.e., process objects) will operate only on
specific types of data and generalizing them results in large inefficiencies in representation or data
access. A typical example is a data object that represents structured data such as pixmaps or 3D
volumes. Because the data is structured it can easily be accessed as planes or lines. However, a
general representation will not include this capability since typically data is not structured.

Another approach to maintain proper input type is to design typed
systems. In typed systems only objects of compatible type are allowed to be
connected together. That is, more than one type is designed, but type
checking is performed on the input to insure proper con-nection.
Depending on the particular computer language, type checking can be
performed at com-pile, link, or run time. Although type checking does
insure correct input type, this approach often suffers from an
explosion of types. If not careful, the designers of a visualization
system may create too many types, resulting in a fragmented, hard to
use and understand system. In addition, the system may require a large number of *type-converter* filters.
(Type-converter filters serve only to transform data from one form to
another.) Carried to extremes, excessive type conversion results in
computationally and memory wasteful systems.

The issue of multiplicity deals with the number of input data objects
allowed, and the number of output data objects created during the
operation of a process object (**Figure 4-4**). We know that all
filter and mapper objects require at minimum one input data object,
but in general these filters can operate sequentially across a list of
input. Some filters may naturally require a specific number of inputs.
A filter implementing boolean operations is one example. Boolean
operations such as union or intersection are implemented on data
values two at a time. However, even here more than two inputs may be
defined as a recursive application of the operation to each input.

<figure id="Figure 4-4">
  <img src="https://raw.githubusercontent.com/lorensen/VTKExamples/master/src/VTKBook/Figures/Figure4-4.png?raw=true width="640" alt="Figure4-4">
</figure>
<figcaption style="color:blue"><b>Figure 4-4</b>. Multiplicity of input and output. (a) Definition of source, filter, and mapper objects. (b) Various types of input and output..</figcaption>
</figure>

We need to distinguish what is meant by multiplicity of output. Most sources and filters generate a single output. Multiple fan-out occurs when an object generates an output that is used for input by more than one object. This would occur, for example, when a source object is used to read a data file, and the resulting data is used to generate a wireframe outline of the data, plus contours of the data (e.g., **Figure 4–1(a)**). Multiple output occurs when an object generates two or more output data objects. An example of multiple output is generating x, y, and z components of a gradient function as distinct data objects. Combinations of multiple fan-out and multiple output are possible.


**Loops**

In the examples described so far, the visualization networks have been free of cycles. In graph theory these are termed directed, acyclic graphs. However, in some cases it is desirable to introduce feedback loops into our visualization networks. Feedback loops in a visualization network allow us to direct the output of a process object upstream to affect its input.

**Figure 4–5** shows an example of a feedback loop in a visualization network. We seed a velocity field with an initial set of random points. A probe filter is used to determine the velocity (and possibly other data) at each point. Each point is then repositioned in the direction of its associated vector value, possibly using a scale factor to control the magnitude of motion. The process continues until the points exit the data set or until a maximum iteration count is exceeded.

<figure id="Figure 4-5">
  <img src="https://raw.githubusercontent.com/lorensen/VTKExamples/master/src/VTKBook/Figures/Figure4-5.png?raw=true width="640" alt="Figure4-5">
</figure>
<figcaption style="color:blue"><b>Figure 4-5</b>. Looping in a visualization network. This example implements linear integration. The sample points are created to initialize the looping process. The output of the integration filter is used in place of the sample points once the process begins.</figcaption>
</figure>

##4.4 Executing the Pipeline

So far we have seen the basic elements of the visualization network and ways to connect these elements together. In this section we discuss how to control the execution of the network.

To be useful, a visualization network must process data to generate a desired result. The complete process of causing each process object to operate is called the execution of the network.

Most often the visualization network is executed more than once. For example, we may change the parameters of, or the input to, a process object. This is typically due to user interaction: The user may be exploring or methodically varying input to observe results. After one or more changes to the process object or its input, we must execute the network to generate up-to-date results.

For highest performance, the process objects in the visualization network must execute only if a change occurs to their input. In some networks, as shown in Figure 4–6, we may have parallel branches that need not execute if objects are modified local to a particular branch. In this figure, we see that object D and the downstream objects E and F must execute because D’s input parameter is changed, and objects E and F depend on D for their input. The other objects need not execute because there is no change to their input.

We can control the execution of the network using either a demand-driven or event-driven approach. In the demand-driven approach, we execute the network only when output is requested, and only that portion of the network affecting the result. In the event-driven approach, every change to a process object or its input causes the network to reexecute. The advantage of the eventdriven approach is that the output is always up to date (except during short periods of computation). The advantage of the demand-driven approach is that large numbers of changes can be processed without intermediate computation (i.e., data is processed only after the request for data is received). The demand-driven approach minimizes computation and results in more interactive visualization networks.

<figure id="Figure 4-6">
  <img src="https://raw.githubusercontent.com/lorensen/VTKExamples/master/src/VTKBook/Figures/Figure4-6.png?raw=true width="640" alt="Figure4-6">
</figure>
<figcaption style="color:blue"><b>Figure 4-6</b>. Network execution. Parallel branches need not execute.</figcaption>
</figure>

The execution of the network requires synchronization between process objects. We want to execute a process object only when all of its input objects are up to date. There are generally two ways to synchronize network execution: explicit or implicit control (**Figure 4-7**).

**Explicit Execution**

Explicit control means directly tracking the changes to the network,
and then directly controlling the execution of the process objects
based on an explicit dependency analysis. The major character-istic of
this approach is that a centralized *executive* is used to coordinate
network execution. This executive must track changes to the parameters
and inputs of each object, including subsequent changes to the network
topology ( **Figure 4-7**(a)).

The advantage of this approach is that synchronization analysis and
update methods are local to the single executive object. In addition,
we can create dependency graphs and perform analysis of data flow each
time output is requested. This capability is particularly important if
we wish to decompose the network for parallel computing or to
distribute execution across a network of com-puters.

The disadvantage of the explicit approach is that each process object
becomes dependent upon the executive, since the executive must be
notified of any change. Also, the executive cannot easily control
execution if the network execution is conditional, since whether to
execute or not depends on the local results of one or more process
objects. Finally, a centralized executive can cre-ate non-scalable
bottlenecks in parallel computing environments.

The explicit approach may be either demand-driven or event-driven. In
the event-driven approach, the executive is notified whenever a change
to an object occurs (typically in response to a user-interface event),
and the network is immediately executed. In the demand-driven
approach, the executive accumulates changes to object inputs and
executes the network based on explicit user demand.

<figure id="Figure 4-7">
  <img src="https://raw.githubusercontent.com/lorensen/VTKExamples/master/src/VTKBook/Figures/Figure4-7.png?raw=true width="640" alt="Figure4-7">
</figure>
<figcaption style="color:blue"><b>Figure 4-7</b>. Explicit and implicit network execution.</figcaption>
</figure>

The explicit approach with a central executive is typical of many
commercial visualization systems such as AVS, Irix Explorer, and IBM
Data Explorer. Typically these systems use a visual-programming
interface to construct the visualization network. Often these systems
are imple-mented on parallel computers, and the ability to distribute
computation is essential.

**Implicit Execution**

Implicit control means that a process object executes only if its local input or parameters change (**Figure 4-7** (b)). Implicit control is implemented using a two-pass process. First, when output is requested from a particular object, that object requests input from its input objects. This process is recursively repeated until source objects are encountered. The source objects then execute if they have changed or their external inputs have changed. Then the recursion unwinds as each process object examines its inputs and determines whether to execute. This procedure repeats until the initial requesting object executes and terminates the process. These two steps are called the *update* and *execution* passes.

Implicit network execution is naturally implemented using *demand-driven* control. Here net-work execution occurs only when output data is requested. Implicit network execution may also be event-driven if we simply request output each time an appropriate event is encountered (such as change to object parameter).

<figure id="Figure 4-8">
  <img src="https://raw.githubusercontent.com/lorensen/VTKExamples/master/src/VTKBook/Figures/Figure4-8.png?raw=true width="640" alt="Figure4-8">
</figure>
<figcaption style="color:blue"><b>Figure 4-8</b>. Examples of conditional execution. Depending upon range, data is mapped through different color lookup tables.</figcaption>
</figure>

The primary advantage of the implicit control scheme is its simplicity. Each object only need keep track of its internal modification time. When output is requested, the object compares its mod-ification time with that of its inputs, and executes if out of date. Furthermore, process objects need only know about their direct input, so no global knowledge of other objects (such as a network executive) is required.

The disadvantage of implicit control is that it is harder to distribute network execution across computers or to implement sophisticated execution strategies. One simple approach is to create a queue that executes process objects in order of network execution (possibly in a distributed fash-ion). Of course, once a central object is introduced back into the system, the lines between implicit and explicit control are blurred.

**Conditional Execution**

Another important capability of visualization networks is conditional
execution. For example, we may wish to map data through different
color lookup tables depending upon the variation of range in the data.
Small variations can be amplified by assigning more colors within the
data range, while we may compress our color display by assigning a
small number of colors to the data range (**Figure 4-8** ).

The conditional execution of visualization models (such as that shown
**Figure 4-1** (c)) can be realized in principle. However, in practice
we must supplement the visualization network with a conditional
language to express the rules for network execution. Hence,
conditional execution of visualization networks is a function of
implementation language. Many visualization systems are programmed
using the visual programming style. This approach is basically a
visual editor to con-struct data flow diagrams directly. It is
difficult to express conditional execution of networks using this
approach. Alternatively, in a procedural programming language,
conditional execution of net-

+-----------------------------------+-----------------------------------+
| works is straightforward. We      | "Putting It All Together " on   |
| defer discussion of this topic    |                                   |
| until                             |                                   |
+-----------------------------------+-----------------------------------+
| page101 .                         |                                   |
+-----------------------------------+-----------------------------------+

## 4.5 Memory and Computation Trade-off

Visualization is a demanding application, both in terms of computer
memory and computational

**Figure 4-9** Comparison of static versus dynamic memory models for
typical network. Execution begins when output is requested from
objects *C* and *D*. In more complex dynamic models, we can pre-vent
*B* from executing twice by performing a more thorough dependency
analysis.

requirements. Data streams on the order of one megabyte to one
gigabyte are not uncommon. Many visualization algorithms are
computationally expensive, in part due to input size, but also due to
the inherent algorithm complexity. In order to create applications
that have reasonable performance, most visualization systems have
various mechanisms to trade off memory and computation costs.

**Static and Dynamic Memory Models**

Memory and computation trade-offs are important performance issues
when executing visualiza-tion networks. In the networks presented thus
far, the output of a process object is assumed to be available to
downstream process objects at all times. Thus, network computation is
minimized.

However, the computer memory requirement to preserve filter output can
be huge. Networks of only a few objects can tie up extensive computer
memory resources.

An alternative approach is to save intermediate results only as long
as they are needed by other objects. Once these objects finish
processing, the intermediate result can be discarded. This approach
results in extra computation each time output is requested. The memory
resources required are greatly reduced at the expense of increased
computation. Like all trade-offs, the proper solution depends upon the
particular application and the nature of the computer system executing
the visualization network.

We term these two approaches as *static* and *dynamic* memory models.
In the static model intermediate data is saved to reduce overall
computation. In the dynamic model intermediate data is discarded when
it is no longer needed. The static model serves best when small,
variable portions of the network reexecute, and when the data sizes
are manageable by the computer system. The dynamic model serves best
when the data flows are large, or the same part of the network
executes each time. Often, it is desirable to combine both the static
and dynamic models into the same net-work. If an entire leg of the
network must execute each time, it makes no sense to store
intermedi-ate results, since they are never reused. On the other hand,
we may wish to save an intermediate result at a branch point in the
network, since the data will more likely be reused. A comparison of
the static and dynamic memory model for a specific network is shown in
**Figure 4-9** .

As this figure shows, the static model executes each process object
only once, storing inter-mediate results. In the dynamic model, each
process object releases memory after downstream objects complete
execution. Depending upon the implementation of the dynamic model,
process object B may execute once or twice. If a thorough dependency
analysis is performed, process B will release memory only after both
objects C and D execute. In a simpler implementation, object B will
release memory after C and subsequently, D executes.

**Reference Counting & Garbage Collection**

Another valuable tool to minimize memory cost is to share storage
using reference counting. To use reference counting, we allow more
than one process object to refer to the same data object and keep
track of the number of references. For example, assume that we have
three objects *A*, *B*, and *C* that form a portion of a visualization
network as shown in **Figure 4-10** . Also assume that these objects
modify only part of their input data, leaving the data object that
specifies *x-y-z* coordinate position unchanged. Then to conserve
memory resources we can allow the output of each process object to
refer to the single data object representing these points. Data that
is changed remains local to each filter and is not shared. It is only
when the reference count goes to zero that the object is deleted.

Garbage collection is an alternative memory management strategy that
is not well suited to visualization applications. The garbage
collection process is automatic; it attempts to reclaim mem-ory used
by objects that will never again be accessed by the running
application. While convenient due to its automated nature, in general
garbage collection introduces overhead that may inadvert-ently
introduce pauses into software execution at inopportune times (during
an interactive process). Of more concern, however, is that released,
unused memory may not be reclaimed by the system until some time after
the last reference to the memory is dropped, and in visualization
pipelines this memory may be too large to leave around for any length
of time. That is, in some applications if memory usage in a filter is
not released immediately, downstream filters may not have enough
memory resource available to them to successfully execute.

## 4.6 Advanced Visualization Pipeline Models

The preceding sections have provided a general framework for the
implementation of a useful visu-alization pipeline model. However,
there are several advanced capabilities that complex applica-tions
often require. These capabilities are driven by deficiencies in the
simpler design described previously. The principle drivers for
developing advanced models include: processing unknown dataset types,
managing complex execution strategies including processing pieces of
data, and extending the visualization pipeline to propagate new
information. These concerns are discussed in the following three
sections.

**Processing Unknown Dataset Types**

There exist data files and data sources where the type of dataset
represented by the file or source is unknown until run-time. For
example, consider a general purpose VTK reader that can read any type
of VTK data file. Such a class is convenient because the user need not
concern himself with the type of dataset, instead the user may want to
set up a single pipeline that processes whatever type is found. As
indicated by **Figure 4-3** , such an approach works well if the
system is of a single dataset type, however in practice, and due to
performance/efficiency concerns, there typically exist many different
types of data in a visualization system. The other alternative shown
in **Figure 4-3** is enforced type checking. However, in situations
like the reader example described above, it is not possible to enforce
type checking at compile-time because the type is determined by the
data. As a result, type checking must be performed at run-time.

Run-time type checking of a multiple dataset type visualization system
requires that the data passed between filters is a generic dataset
container (i.e., it appears like a single type but contains the actual
data and methods to determine what type of data it is). Run-time type
checking has the advantage of flexibility, but the trade-off is that a
pipeline may not execute properly until the pro-gram executes. For
example, a generic pipeline may be designed that can process
structured data (see "Types of Datasets " on page134 ), but the data
file may contain unstructured data. In this case, the pipeline will be
unable to execute at run-time, producing empty output. Thus pipelines
designed to process any type of data must be carefully assembled to
create robust applications.

**Extending the Data Object Representation**

As described earlier in this chapter, a pipeline consists of data
objects that are operated on by pro-cess objects. Further, because the
process objects are separate from the data objects on which they
operate, there is necessarily an expected interface through which
these objects exchange informa-tion. Defining this interface has the
side effect of cementing the data representation, implying that it is
difficult to extend it without modifying the corresponding interface,
and hence all the classes that depend on the interface (of which there
are many). Fortunately what tends to change is not the basic data
representations (these are generally well established), rather the
metadata associated with the dataset itself changes. (In the context
of visualization, metadata are data that describe datasets.) While it
is feasible to represent new datasets by creating new classes (since
the addition of new dataset types occurs infrequently); the diversity
of metadata precludes creating new classes because the resulting
explosion of data types, and the potential change to programming
interfaces, would adversely affect the stability of the visualization
system. Hence a general mechanism to support metadata is required.
Packaging metadata into a generic container that contains both the
dataset andcmetadata is a obvious design, and is compatible with the design
described in the previous section. Examples of metadata include time
step information, data ranges or other data characteristics, acquisition protocols, patient names, and annotation. In an extensible visualization pipeline, a specific data reader (or other data source)
may read such information and associate it with the output data that
it produces. While many filters may ignore the metadata, they can be
configured to pass the information along the pipeline. Alternatively,
a pipeline sink (or mapper) may request that specific metadata be
passed through the pipeline so it can be processed appropriately. For
example, a mapper may request annotations, and if available, place
them on the final image.

**Managing Complex Execution Strategies**

In real-world applications the pipeline design described thus far may
not adequately support com-plex execution strategies, or may fail to
execute successfully when data sizes become large. In the next
sections we address these issues by considering alternative design
possibilities.

**Large Data.** Previous discussions relative to the visualiza-tion
pipeline have assumed that the size of a particular dataset does not
exceed the total memory resource of a computer sys-tem. However, with
modern dataset sizes pushing into the ter-abyte and even petabyte
range, a typical desktop computer system is incapable of processing
such datasets. Thus alterna-tive strategies must be adopted when
processing large data. One such approach is based on breaking data
into pieces, and then *streaming* the pieces through the visualization
pipeline <em style="color:blue;background-color: white">\[Martin2001\]</em> . **Figure 4-11** illustrates how a dataset
can be divided into pieces.

Streaming data through a visualization pipeline offers

+-----------------------+-----------------------+-----------------------+
| two major benefits.   | **Figure 4-11**    |                       |
| The first is that     | Dividing a sphere   |                       |
| visualization data    | into                |                       |
| that                  |                       |                       |
+-----------------------+-----------------------+-----------------------+
| would not normally    |                       |                       |
| fit into memory can   |                       |                       |
| be processed. The     |                       |                       |
+-----------------------+-----------------------+-----------------------+
|                       | a piece (red) with  |                       |
|                       | ghost level cells   |                       |
+-----------------------+-----------------------+-----------------------+
| second is that        | and points (blue    |                       |
| visualizations can be | and green).         |                       |
| run with a smaller    |                       |                       |
| mem-                  |                       |                       |
+-----------------------+-----------------------+-----------------------+
| ory footprint         |                       |                       |
| resulting in higher   |                       |                       |
| cache hits, and       |                       |                       |
| little or no          |                       |                       |
+-----------------------+-----------------------+-----------------------+

swapping to disk. To realize these benefits the visualization software
must support breaking the dataset into pieces and correctly processing
those pieces. This requires that the dataset and the algo-rithms that
operate on it are *separable* , *mappable* , and *result invariant* as
described in the following <em style="color:blue;background-color: white">\[Law99\]</em> .

1.  **Separable.** The data must be separable. That is, the data can be
    broken into pieces. Ideally, each piece should be coherent in
    geometry, topology, and/or data structure. The separation of the
    data should be simple and efficient. In addition, the algorithms
    in this architecture must be able to correctly process pieces of
    data.

2.  **Mappable.** In order to control the streaming of the data through
    a pipeline, we must be able to determine what portion of the input
    data is required to generate a given portion of the out-put. This
    allows us to control the size of the data through the pipeline,
    and configure the algo-rithms.

3.  **Result Invariant.** The results should be independent of the
    number of pieces, and indepen-

  -------------------------------------------- --------
  4.6 Advanced Visualization Pipeline Models   **97**
                                               
  -------------------------------------------- --------

dent of the execution mode (i.e., single- or multi-threaded). This
means proper handling of boundaries and developing algorithms that are
multi-thread safe across pieces that may over-lap on their boundaries.

Separating data into pieces is relatively straightforward if the data
is structured, i.e., topologically regular (see "Types of Datasets "
on page134 ). Such datasets can be topological described by a
rect-angular extent in a regularly *x-y-z* subdivided cubical domain
(see **Figure5--7** (a)-(c)). However, if the data is unstructured
(e.g. a mesh of triangles or polygons), then specifying pieces is
difficult. Generally an unstructured extent is defined by grouping
adjacent data (e.g., cells) into pieces, and then addressing each
piece using a *N* of *M* notation, where *N* is the n *^th^* piece out
of a total of *M* pieces. The exact organizational structure of a
piece is left unspecified and depends on the particu-lar application
and algorithm used to group the data.

To satisfy the third requirement of results invariancy, processing
pieces also requires the abil-ity to generate boundary data, or *ghost
levels* . Boundary information is necessary when information from the
neighbors of a piece is needed to perform a computation. For example,
gradient calcula-tions or boundary analysis (e.g., do I have a cell
face neighbor?) require one level of boundary

information. In rare cases, two or more levels are required.
**Figure 4-11** illustrates boundary cells and points corresponding to
the central red piece of the sphere.

Finally, it should be noted that the ability to divide data into
pieces for streaming is exactly the same capability required for data
parallel processing. In such methods, data is subdivided and sent to
different processors to be operated on in parallel. Boundary
information may also be required to perform certain computations.
Parallel processing has the added complexity that the data must be
communicated to processors (in the case of distributed computing) or
mutual exclu-sion (i.e., mutexing) must be employed to avoid
simultaneous write operations. Thus streaming and parallel processing
are complementary technologies used in large data computing.

**Complex Execution Strategies.** In many cases the simple execution
model of **Figure 4-7** (b) is not suitable for complex data
processing tasks. For example, as discussed in the previous section,
streaming data is a complex execution strategy required when a dataset
becomes too large to fit into

+-----------------------------------+-----------------------------------+
| memory, or when parallel          | "Executing the Pipe-            |
| computing is used. In some cases  |                                   |
| event-driven (see                 |                                   |
+-----------------------------------+-----------------------------------+
| line" on page89 ) or "push"       |
| pipelines (i.e., those that       |
| receive data and push the data    |
| through the                       |
+-----------------------------------+-----------------------------------+
| pipeline for processing) may be   |
| preferred. Finally, there exist   |
| hierarchical data structures such |
| as                                |
+-----------------------------------+-----------------------------------+
| multi-block or adaptive mesh      | <em style="color:blue;background-color: white">\[Berger84\]</em> grids. Processing  |
| refinement (AMR)                  | such datasets in a              |
+-----------------------------------+-----------------------------------+

pipeline requires hierarchical traversal as filters process each block
in the grid (an advanced research topic in the visualization field and
not covered in this edition of the book).

Addressing these requirements implies that the execution model must be
extended. Thus we revisit the object-oriented design in the next
section.

**Object-Oriented Design Revisited. Figure 4-2** illustrates two choices relative to the design of the visualization object model. The first choice, which was discarded, was to combine data and operations on the data into a single object, a typical object-oriented design
pattern. The second choice, which was advocated, was to create a
design consisting of two classes---data objects and process
objects---which were then combined into visualization pipelines. While
this second strategy works well for simple pipelines, when complex
execution strategies are introduced, this design begins to break down.
This is because the execution strategy is necessarily, and implicitly,
distributed across the data objects and process objects; there is no
explicit mechanism to implement a particular strategy. Thus the design is problematic because new strategies cannot be
introduced without modifying the interface to both the data and
process objects. Good design demands that the execution strategy is
separated from the data objects and process objects. The benefits of
such a design include reducing the complexity of the data and process
objects, encapsulating execution strategies, performing run-time type
checking (see "Processing Unknown Dataset Types " on page95 ) and even
managing metadata (see "Extending the Data Object Representation " on
page95 ).

As the execution model becomes more complex, execution strategies are
separated from the data and pro-cess objects as separate classes.

**Figure 4-12**

**Process Object**

**Process Object**

**Data Object**

**Executive**

**Data Object**


The advanced design re-introduces the notion of an executive (see
"Executing the Pipeline " on page89 ).

However, the design differs from that of **Figure 4-7** . As that
figure illustrated, a single, centralized executive introduces
dependencies into the pipeline that will not scale as pipeline
complexity increases, or in parallel processing applications. In the
advanced design, we assume *multiple* executives, typically one per filter. In 
some cases the executive may control multiple filters.

This is particularly useful if the filters are interdepen-dent or
complex execution strategies are required. Different classes of
executive can implement different execution strategies, for example a
demand-driven, streaming pipeline is one such strategy. Other
important classes include executives that coordinate the execution of
filters on composite datasets.

**Figure 4-12** is a high-level view of the executive and its
relationship to data and process

objects. In "Pipeline Design and Implementation " on page103 the design
is explored in more detail.

## 4.7 Programming Models

Visualization systems are by their very nature designed for human
interaction. As a result they must be easy to use. On the other hand,
visualization systems must readily adapt to new data, and must be
flexible enough to allow rapid data exploration. To meet these
demands, a variety of program-ming models have been developed.

**Visualization Models**

At the highest level are applications. Visualization applications have
finely tailored user-interfaces that are specific to an application
area, e.g., fluid flow visualization. Applications are the easiest to
use, but are the least flexible. It is very difficult or impossible
for the user to extend applications into a new domain because of
inherent logistical issues. Commercial turn-key visualization
soft-ware is generally considered to be application software.

At the opposite end of the spectrum are programming libraries. A
conventional programming library is a collection of procedures that
operate on a library-specific data structure. Often these libraries
are written in conventional programming languages such as C or
FORTRAN. These offer great flexibility and can be easily combined with
other programming tools and techniques. Pro-gramming libraries can be
extended or modified by the addition of user-written code.
Unfortu-nately, the effective use of programming libraries requires
skilled programmers. Furthermore, non

  ------------------------ --------
  4.7 Programming Models   **99**
                           
  ------------------------ --------

graphics/visualization experts cannot easily use programming libraries
because there is no notion of how to fit (or order) the procedures
together correctly. These libraries also require extensive
syn-chronization schemes to control execution as input parameters are
varied.

Many visualization systems lie between these two extremes. These
typically use a *visual pro-gramming* approach to construct
visualization networks. The basic idea is to provide graphical tools
and libraries of modules or process objects. Modules may be connected
subject to input/output type constraints, using simple graphical
layout tools. In addition, user interface tools allow association

of interface widgets with object input parameters. System execution is
generally transparent to the user by way of an internal execution
executive.

**Alternative Visual Programming Models**

There are two other graphics and visualization programming models that
bear mentioning. These are *scene graphs* and the *spreadsheet* model.

Scene graphs are typically found in 3D graphics systems such as
OpenInventor <em style="color:blue;background-color: white">\[Wernecke94\]</em> . Scene graphs are acyclic tree-structures
that represent objects, or nodes, in an order defined by the tree
layout. The nodes may be geometry (called shape nodes), graphics
properties, transformations, manipulators, lights, cameras, and so
forth, that define a complete scene. The par-ent/child relationship
controls how properties and transformations are applied to the nodes
as they are rendered, or how the objects relate to other objects in
the scene (e.g., which objects the lights shine on). Scene graphs are
not used to control the execution of a visualization pipeline, rather
they are used to control the rendering process. Scene graphs and
visualization pipelines may be used together in the same application.
In such a case the visualization pipeline is the generator of the
shape nodes, and the scene graph controls the rendering of the scene
including the shapes.

Scene graphs have found wide use in the graphics community because of
their ability to com-pactly and graphically represent a scene. In
addition, scene graphs have been popularized by their recent use in
Web tools such as VRML and Java3D. See Chapter 11 for more
information.

Another recently introduced technique for visual programming is the
spreadsheet technique of Levoy <em style="color:blue;background-color: white">\[Levoy94\]</em> . In the spreadsheet model,
we arrange operations on a regular grid similar to the common
electronic accounting spreadsheets. The grid consists of rows and
columns of cells, where each cell is expressed as a computational
combination of other cells. The combination is expressed for each cell
by using a simple programming language to add, subtract, or perform
other more com-plex operations. The result of the computation (i.e., a
visual output) is displayed in the cell. A

recent extension to the spreadsheet approach is exemplified by
VisTrails <em style="color:blue;background-color: white">\[Bavoil2005\]</em> , a system that enables interactive
multiple-view visualizations by simplifying the creation and
maintenance of visualization pipelines, and by optimizing the
execution of the pipelines. VisTrials has the further benefit that it
tracks changes to the visualization pipeline so that it is
straightforward to create extensive design studies.

Although visual programming systems are widely successful, they suffer
two drawbacks. First, they are not as tailored as an application and
require extensive programming, albeit visual, to be so. Second, visual
programming is too limited for detailed control, so constructing
complex low-level algorithms and user-interfaces is not feasible. What
is required is a visualization system that provides the "modularity"
and automatic execution control of a visual system, and the low-level
programming capability of a programming library. Object-oriented
systems have the potential to provide these capabilities. Carefully
crafted object libraries provide the ease of use of visual sys-

**100** The Visualization Pipeline

tems with the control of programming libraries. That is a major goal
of the described in this text.

*Visualization Toolkit*

## 4.8 Data Interface Issues

At this point in the text you may be wondering how to apply a
visualization pipeline towards your own data. The answer depends on
the type of data you have, preferences in programming style, and
required complexity. Although we have not yet described particular
types of data (we will in the next chapter), there are two general
approaches you may wish to consider when interfacing your data to a
visualization system: a programming interface and an application
interface.

**Programming Interface**

The most powerful and flexible approach is to directly program your
application to read, write, and process data. There is almost no limit
to what you can achieve using this approach. Unfortunately, in a
complex system like VTK this requires a level of expertise that may be
beyond your time bud-get to obtain. (If you are interested in this
approach using VTK, you'll have to become familiar with the objects in
the system. You will also want to refer to the Doxygen-generated
manual pages ---on-line at http://www.vtk.org or CD-ROM. The companion
text *The VTK User's Guide* is also helpful.)

Typical applications requiring a programming interface are interfacing
to data files that are not currently supported by the system or
generating synthetic data (e.g., from a mathematical rela-tionship)
where no data file is available. Also, sometimes it is useful to
directly code your data in the form of a program, and then execute the
program to visualize the results. (This is exactly what many of the
VTK examples do.)

In general, programming a complex system such as VTK is a difficult
undertaking because of the initial learning curve. There are, however,
simpler ways to interface to data. While skilled developers may be
required to create sophisticated applications, the points of an
object-oriented toolkit like VTK is that it provides many of the
pieces required to interface to common data forms. Thus focusing on
those objects that import and export data is a good start towards
interfacing with data. In VTK, these objects are known as readers,
writers, importers and exporters.

**File Interface (Readers / Writers).** In this chapter we saw that
readers are source objects, and writers are mappers. What this means
from a practical point of view is that readers will ingest data from a
file, create a data object, and then pass the object down the pipeline
for processing. Simi-larly, writers ingest a data object and then
write the data object to a file. Thus, readers and writers will
interface to your data well if VTK supports your format, *and* you
only need to read or write a *single* data object. If your data file
format is not supported by the system, you will need to interface to
your data via a general programming interface described above. Or, if
you wish to interface to a collection of objects, you will probably
want to see whether an exporter or importer object (described in the
next section) exists to support your application.

Examples of readers include vtkSTLReader (read stereo-lithography
files) and vtkBYUReader (read MOVIE.BYU format data files). Similarly
the objects vtkSTLWriter

and

vtkBYUWriter can be used to write data files. To see which readers and
writers are supported by VTK, see the *VTK User's Guide* or refer to the Web pages at
http://www.vtk.org for the cur-rent Doxygen manual pages.

**File Interface (Importers / Exporters).** *Importers* and
*exporters* are objects in the system that read or write data files
consisting of more than one object. Typically importers and exporters
are used to save or restore an entire scene (i.e., lights, cameras,
actors, data, transformations, etc.). When an importer is executed, it
reads one or more files and may create several objects. For exam-ple,
in VTK the vtk3DSImporter imports a *3D Studio* file and creates a
rendering window, renderer, lights, cameras, and actors. Similarly,
the vtkVRMLExporter creates a VRML file given a VTK render window. The
VRML file contains cameras, lights, actors, geometry, transformations,
and the like, indirectly referred to by the rendering window provided.

In the *Visualization Toolkit* , there are several importers and
exporters. To see which importers and exporters are supported by VTK,
see the *VTK User's Guide* . You may also want to check the Web pages
at http://www.vtk.org for the current Doxygen manual pages. If the
exporter you are looking for does not exist, you will have to develop
your own using the programming interface.

**Figure 4-13** shows an image created from a *3D Studio* model and
saved as a *Renderman* RIB file.

**Application Interface**

The majority of users interface to their data by using an existing
application. Rather than program-ming pipelines or writing their own
readers and writers, users acquire an application that suits their
particular visualization needs. Then to interface to their data, users
simply identify the reader, writer, importer, and/or exporter that can
successfully process it. In some cases, users may have to modify the
program used to generate the data so that it exports it in a standard
data format. The advantage of using existing applications is that the
user interface and pipeline are pre-programmed, insuring that the user
can focus on their data, rather than expending the significant
resources required to write visualization programs. The disadvantage
of using existing applications is that necessary features are often
missing, and applications typically lack the flexibility that a
general purpose tool can provide.

Selecting the right application is not always simple. The application
must support the correct dataset types, and support suitable rendering
devices, for example generating images on large dis-plays
<em style="color:blue;background-color: white">\[Humphreys99\]</em> or in a Cave <em style="color:blue;background-color: white">\[CruzNeira93\]</em> environment. In some
cases user interaction is required, and demands on parallel processing
or data handling capacities further complicates the selection. For
example, while a general purpose tool like ParaView ( **Figure 4-14**
(a)) can be used to visualize most types of data, including providing
support for large data and parallel computing, a specialized tool such
as VolView ( **Figure 4-14** (b)) may be better suited for the a
particular type task such as viewing medical data shown in the figure.
It is imperative that users have a familiarity with the visualization
process if they are to successfully choose the right application for
their data.

## 4.9 Putting It All Together

In the previous sections we have treated a variety of topics relating
to the visualization model. In

this section we describe the particular implementation details that we
have adopted in the *Visualiza-tion Toolkit* .

**102** The Visualization Pipeline

***iflamigm.3ds***

***importExport.rib***

``` tcl
-   import from 3d Studio vtk3DSImporter importer

importer ComputeNormalsOn

importer SetFileName "$VTK_DATA_ROOT/Data/iflamigm.3ds" importer
Read

-   export to rib format

vtkRIBExporter exporter

exporter SetFilePrefix importExport

exporter SetRenderWindow <em style="color:blue;background-color: white">\[importer GetRenderWindow\]</em>

exporter BackgroundOn

exporter Write
```

**Figure 4-13** Importing and exporting files in VTK. An importer
creates a vtkRenderWindow that describes the scene. Exporters use an
instance of vtkRenderWindow to obtain a description of the scene (
3dsToRIB.tcl and flamingo.tcl ).

**Procedural Language Implementation**

The *Visualization Toolkit* is implemented in the procedural language
C++. Automated wrapping technology creates language bindings to the
Python, Tcl and Java interpretive programming lan-guages <em style="color:blue;background-color: white">\[King03\]</em> .
The class library contains data objects, filters (i.e., process
objects) and executives to facilitate the construction of
visualization applications. A variety of supporting abstract
super-classes are available to derive new objects including data
objects and filters. The visualization pipe-line is designed to
connect directly to the graphics subsystem described in the previous
chapter. This connection is via VTK's mappers, which are the sinks of
the pipeline and interface to the VTK's actors.

A visual programming interface could be (and has been) implemented
using the class library provided. However, for real-world applications
the procedural language implementation provides several advantages.
This includes straightforward implementation of conditional network
execution and looping, ease of interface to other systems, and the
ability to create custom applications with sophisticated graphical
user interfaces. The VTK community has created several visual program-

(b) ParaView parallel visualization application. (b) VolView volume
rendering application.

**Figure 4-14** The choice of an appropriate visualization
application depends on the type of dataset(s) it must support,
required interaction techniques, rendering capabilities, and support
for large data, including parallel processing. While both applications
above are built using the VTK visualization toolkit, they provide very
different user experiences. ParaView (paraview.org) is a general
purpose visualization system that can process large data in a
distributed, parallel envi-ronment (as well as on single processor
systems), with the ability to display on a Cave or tiled display.
VolView (volview.com) focuses on volumetric and image data and uses
multi-threading and sophisticated level-of-detail methods to achieve
interactive performance.

ming and visualization applications from the toolkit. Many of these
are available as open-source software (e.g., ParaView at paraview.org)
or as commercial applications (e.g., VolView at www.volview.com).

**Pipeline Design and Implementation**

The *Visualization Toolkit* implements a general execution mechanism.
Filters are divided into two basic parts: algorithm and executive
objects. An algorithm object, whose class is derived from vtkAlgorithm
, is responsible for processing information and data. An executive
object, whose class is derived from vtkExecutive , is responsible for
telling an algorithm when to execute and what information and data to
process. The executive component of a filter may be created
independently of the algorithm component allowing custom pipeline
execution mechanisms without modifying core VTK classes.

Information and data produced by a filter are stored in one or more
output *ports*. An output port corresponds to one logical output of
the filter. For example, a filter producing a color image and a
corresponding binary mask image would define two output ports, each
holding one of the images. Pipeline-related information is stored in
an instance of vtkInformation on each output port. The data for an
output port is stored in an instance of a class derived from
vtkDataObject .

Information and data consumed by a filter are retrieved through one or
more input ports. An input port corresponds to one logical input of
the filter. For example, a glyph filter would define one input port
for the glyph itself and another input port defining the glyph
positions. Input ports store input connections that reference the
output ports of other filters; these output ports eventually provide
information and data to the filter. Each input connection provides one
data object and its corresponding information obtained from the output
port to which the connection is made. Since connections are stored
through logical ports and not in the data flowing through those ports,
the data type need not be known when the connection is made. This is
particularly useful when creating

*Direction of data flow via RequestData()*

**Figure 4-15** Description of implicit execution process implemented
in VTK. The Update() method is initiated via the Render() method from
the actor. Data flows back to the mapper via the RequestData() method.
Arrows connecting filter and data objects indicate direction of the
Update() process.

pipelines whose source is a reader that does not know its output data
type until the file is read (see "Pipeline Connections " on page86 and
"Processing Unknown Dataset Types " on page95 ).

To understand the execution of the VTK pipeline, it is useful to view
the process from several different vantage points. Note that each of
the following figures is not completely accurate, rather they are
presented as depictions whose purpose is to describe the important
features of the process.

**Figure 4-15** shows a simplified description of VTK's execution
process. Generally the exe-cution of the pipeline is triggered by a
mapper's Render() method invocation, typically in response to a
Render() method invocation on an associated vtkActor (which in turn
receives it from the

render window). Next, the Update() method is called on the input to
the mapper (resulting in a

cascade of method invocations requesting information and data).
Eventually, data must be

computed and returned to the object initiating the request, in this
case the mapper. The

RequestData() method actually executes the filter(s) in the pipeline
and produces output data. Note

the direction of flow---here we define the direction of data flow as
the *downstream* direction, and the direction of the Update()
invocation the *upstream* direction.

The next figure, **Figure 4-16** , shows the relationship between the
executive and the algo-rithm, which are paired to form a filter. This
view of the filter is independent of the pipeline and contains all the
information about the interface of the algorithm, namely the number
and availabil-ity of inputs and outputs. Finally **Figure 4-17** shows
the connections between filters. Notice that the output data object is
not directly wired to the input connection. Rather the downstream
filters's input connection is associated with the upstream filter's
output port. This separation of data object from the input port means
that data type checking can be deferred until run-time, when the
consum-ing filter requests data from the producer of the data. Thus
the producer can generate different types of data (e.g., it is a
reader that produces different data types), and as long as the
consumer supports these different data types, the pipeline will
execute without error.

4.11 Bibliographic Notes

The practical way to learn about the visualization process is to study
commercially available sys-tems. These systems can be categorized as
either direct visual programming environments or as

applications. Common visual programming systems include AVS <em style="color:blue;background-color: white">\[AVS89\]</em>
, Iris Explorer <em style="color:blue;background-color: white">\[IrisEx-plorer\]</em> , IBM Data Explorer <em style="color:blue;background-color: white">\[DataExplorer\]</em>
, aPE <em style="color:blue;background-color: white">\[aPE90\]</em> , and Khoros <em style="color:blue;background-color: white">\[Rasure91\]</em> . Application sys-tems
generally provide less flexibility than visual programming systems,
but are better tailored to a particular problem domain. PLOT3D
<em style="color:blue;background-color: white">\[PLOT3D\]</em> is an early example of a tool for CFD visualization. This
has since been superseded by FAST <em style="color:blue;background-color: white">\[FAST90\]</em> . FieldView is another
popular CFD visualizer <em style="color:blue;background-color: white">\[FieldView91\]</em> . VISUAL3 <em style="color:blue;background-color: white">\[VISUAL3\]</em> is a
general tool for unstructured or structured grid visualiza-tion.
PV-WAVE <em style="color:blue;background-color: white">\[Charal90\]</em> can be considered a hybrid system, since it has
both simple visual pro-gramming techniques to interface to data files
as well as a more structured user-interface than the visual
programming environments. Wavefront's DataVisualizer
<em style="color:blue;background-color: white">\[DataVisualizer\]</em> is a general-pur-pose visualization tool. It is
unique in that it is part of a powerful rendering and animation
package. A nice system for visualizing 3D gridded data (such as that
produced by numerical weather models) is VIS5D. Find out more at the VIS5D Web site
http://www.ssec.wisc.edu/\~billh/vis5d.html .

Although many visualization systems claim to be object-oriented, this
is often more in appearance than implementation. Little has been
written on object-oriented design issues for visual-ization. VISAGE <em style="color:blue;background-color: white">\[VISAGE92\]</em> presents an architecture similar to that described in this chapter. Favre <em style="color:blue;background-color: white">\[Favre94\]</em> describes a more conventional object-oriented approach. His dataset classes are based on topological dimension and both data and methods are combined into classes.

## 4.12 References

<em style="color:blue;background-color: white">\[aPE90\]</em>
D. S. Dyer. "A Dataflow Toolkit For Visualization." *IEEE Computer
Graphics and Applications.*
10(4):60--69, July 1990.

<em style="color:blue;background-color: white">\[AVS89\]</em>
C. Upson, T. Faulhaber Jr., D. Kamins and others. "The Application
Visualization System: A Computational Environment for Scientific
Visualization." *IEEE Computer Graphics and Appli-cations.*
9(4):30--42, July 1989.

<em style="color:blue;background-color: white">\[Bavoil2005\]</em>
L. Bavoil, S.P. Callahan, P.J. Crossno, J. Freire, C.E. Scheidegger,
C.T. Silva and H.T. Vo. "Vis-Trails: Enabling Interactive
Multiple-View Visualizations." In *Proceedings of IEEE Visualization*
*2005* . IEEE Computer Society Press, 2005.

<em style="color:blue;background-color: white">\[Berger84\]</em>
M. Berger and J. Oliger. Adaptive Mesh Refinement for Hyperbolic
Partial Differential Equations.
*Journal of Computational Physics* , 53:484-512, March 1984.

<em style="color:blue;background-color: white">\[Charal90\]</em>
S. Charalamides. "New Wave Technical Graphics Is Welcome." *DEC USER*
, August 1990.

<em style="color:blue;background-color: white">\[CruzNeira93\]</em>
C. Cruz-Neira, D.J. Sandin, and T. DeFanti. "Surround-screen
projection-based virtual reality:
The design and implementation of the CAVE." In *Proceedings of
SIGGRAPH 93* , pp. 135---142, August 1993.

<em style="color:blue;background-color: white">\[DataExplorer\]</em>
*Data Explorer Reference Manual* . IBM Corp, Armonk, NY, 1991.

<em style="color:blue;background-color: white">\[DataVisualizer\]</em>
*Data Visualizer User Manual .* Wavefront Technologies, Santa Barbara,
CA, 1990.

<em style="color:blue;background-color: white">\[FAST90\]</em>
G. V. Bancroft, F. J. Merritt, T. C. Plessell, P. G. Kelaita, R. K.
McCabe, and A.Globus. "FAST: A Multi-Processed Environment for
Visualization." In *Proceedings of Visualization '90* . pp. 14-27,
IEEE Computer Society Press, Los Alamitos, CA, 1990.

<em style="color:blue;background-color: white">\[Favre94\]</em>
J. M. Favre and J. Hahn. "An Object-Oriented Design for the
Visualization of Multi-Variate Data Objects." In *Proceedings of
Visualization '94* . pp. 319--325, IEEE Computer Society Press, Los
Alamitos, CA, 1994.

<em style="color:blue;background-color: white">\[FieldView91\]</em>
S. M. Legensky. "Advanced Visualization on Desktop Workstations." In
*Proceedings of Visual-ization '91* . pp. 372--378, IEEE Computer
Society Press, Los Alamitos, CA, 1991.

<em style="color:blue;background-color: white">\[Haeberli88\]</em>
P. E. Haeberli. "ConMan: A Visual Programming Language for Interactive
Graphics." *Computer* *Graphics (SIGGRAPH '88)* . 22(4):103--11, 1988.

<em style="color:blue;background-color: white">\[Humphreys99\]</em>
G. Humphreys and P. Hanrahan. "A Distributed Graphics System for Large
Tiled Displays." In *Proc. IEEE Visualization '99* , pp. 215--224,
IEEE Computer Society Press, October 1999.

<em style="color:blue;background-color: white">\[IrisExplorer\]</em>
*Iris Explorer User's Guide* . Silicon Graphics Inc., Mountain View,
CA, 1991.

<em style="color:blue;background-color: white">\[King03\]</em>
B. King and W. Schroeder. "Automated Wrapping of Complex C++ Code."
*C/C++ Users Jour-nal,* January 2003.

<em style="color:blue;background-color: white">\[Law99\]</em>
C. Charles Law, K. M. Martin, W. J. Schroeder, J. E. Temkin. "A
Multi-Threaded Streaming Pipe-line Architecture for Large Structured
Data Sets." In *Proc. of Visualization \'99* . IEEE Computer Society
Press, 1999.

<em style="color:blue;background-color: white">\[Levoy94\]</em>
M. Levoy. "Spreadsheets for Images." In *Proceedings of SIGGRAPH '94*
. pp. 139--146, 1994.

<em style="color:blue;background-color: white">\[Martin2001\]</em>
K.M. Martin, B. Geveci, J. Ahrens, C. Law. "Large Scale Data
Visualization Using Parallel Data Streaming." *IEEE Computer Graphics
& Applications* , 21(4):34-41, July 2001.

<em style="color:blue;background-color: white">\[PLOT3D\]</em>
P. P. Walatka and P. G. Buning. *PLOT3D User's Manual* . NASA Fluid
Dynamics Division, 1988.

<em style="color:blue;background-color: white">\[Rasure91\]</em>
J. Rasure, D. Argiro, T. Sauer, and C. Williams. "A Visual Language
and Software Development Environment for Image Processing."
*International Journal of Imaging Systems and Technology* . 1991.

<em style="color:blue;background-color: white">\[VISAGE92\]</em>
W. J. Schroeder, W. E. Lorensen, G. D. Montanaro, and C. R. Volpe.
"VISAGE: An Object-Ori-ented Visualization System." In *Proceedings of
Visualization '92* . pp. 219--226, IEEE Computer Society Press, Los
Alamitos, CA, 1992.

<em style="color:blue;background-color: white">\[VISUAL3\]</em>
R. Haimes and M. Giles. "VISUAL3: Interactive Unsteady Unstructured 3D
Visualization." AIAA Report No. AIAA-91-0794. January 1991.

<em style="color:blue;background-color: white">\[ Wernecke94\]</em>
J. Wernecke. *The Inventor Mentor* . Addison-Wesley Publishing
Company, ISBN 0-201-62495-8, 1994.

## 4.13 Exercises

4.1 Consider the following 2D visualization techniques: *x-y*
plotting, bar charts, and pie charts.

For each technique:

a)  Construct functional models.

b)  Construct object models.

4.2 A *height field* is a regular array of 2D points wherehfxy= () ,
*h* is an altitude above the point *(x,y*). Height fields are often
used to represent terrain data. Design an object-oriented system to
visualize height fields.

a)  How would you represent the height field?

b)  What methods would you use to access this data?

c)  Develop one process object (i.e., visualization technique) to
    visualize a height field. Describe the methods used by the object
    to access and manipulate the height field.

4.3 Describe how you would implement an explicit control mechanism for
network execution.

a)  How do process objects register their input data with the executive?

b)  How is the executive notified of object modification?

c)  By what method is the executive notified that network execution is
    necessary?

d)  Describe an approach for network dependency analysis. How does the
    executive invoke execution of the process objects?

4.4 Visual programming environments enable the user to construct
visualization applications by graphically connecting process objects.

a)  Design a graphical notation to represent process objects, their
    input and output, and data flow direction.

b)  How would you modify instance variables of process objects (using a
    graphical tech-

  ---------------- ---------
  4.13 Exercises   **119**
                   
  ---------------- ---------

nique)?

c)  By what mechanism would network execution be initiated?

d)  How would you control conditional execution and looping in your
    network?

e)  How would you take advantage of parallel computing?

f)  How would you distribute network execution across two or more
    computers sharing a net-work connection?

4.5 Place oriented cylinders (instead of cones) on the mace in

**Figure 4-20**

. (

*Hint:* use

vtkCylinderSource .)

4.6 The implicit update method for the visualization network used by
VTK is simple to imple-ment and understand. However, it is prone to a
common programming error. What is this error?

4.7 Experiment with the transformation object in **Figure 4-20** .

a\) Translate the actor with vtkTransform 's Translate() method.

b)  Rotate the actor with the RotateX() , RotateY() , and RotateZ()
    methods.

c)  Scale the actor with the Scale() method.

d)  Try combinations of these methods. Does the actor transform in ways
    that you expect?

+-------------+-------------+-------------+-------------+-------------+
| 4.8         |
| Visualize   |
| the         |
| following   |
| functions.  |
| ( *Hint:*   |
| use         |
| vtkSampleFu |
| nction      |
| and refer   |
| to          |
| **Figure 4- |
| 1**         |
| .)          |
+-------------+-------------+-------------+-------------+-------------+
| a)        | Fxyz(),,  | =           | ~x~2      |             |
+-------------+-------------+-------------+-------------+-------------+
| b)        | Fxyz(),,  | =           | x2y3z1+   | ++          |
+-------------+-------------+-------------+-------------+-------------+
| c)        | Fxyz(),,  | =           | ~x~2      | \+ y^2^ --  |
|             |             |             |             | ()cos1z +   |
+-------------+-------------+-------------+-------------+-------------+

