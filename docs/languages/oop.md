# Object-Oriented Programing

## Design patterns

=== "Singleton Pattern"

    __Principles__:

    - __Single Instance__: Ensures only one instance of the class exists throughout the application.
    - __Global Access__: Provide a global point of access to that instance.
    - __Lazy or Egar Initialization__: Support intantiating either when needed(lazy) or when the class is loaded(eager).
    - __Thread Safety__: Restrict multiple threads from creating seperate instances simultaneously.
    - __Private Constructor__: Restrict direct initialization by making constructor private, forcing the use of access point

    __Use Cases__:

    - logging
    - Managing connections to hardware or databases
    - Caching data
    - Handling thread pools
    - Global configuration management

    ```{.java .copy}
    class Singleton {
        private static volatile Singleton obj = null; // (1)!

        private Singleton() {}

        public static Singleton getInstance() {
            if (obj == null) {
                // To make thread safe
                synchronized (Singleton.class)
                {
                    // check again as multiple threads
                    // can reach above step
                    if (obj == null)
                        obj = new Singleton();
                }
            }
            return obj;
        }

        public static void doSomething() {
            System.out.println("Somethong is Done.");
        }
    }

    class GFG {
        public static void main(String[] args) {
            Singleton.getInstance().doSomething();
        }
    }
    ```

    1. - Eager Instantiation `#!java private static Singleton obj = new Singleton();`

        - We have declared the `#!java obj` [volatile](https://www.geeksforgeeks.org/volatile-keyword-in-java/) which ensures that multiple threads offer the `#!java obj` variable correctly when it is being initialized to the Singleton instance. This method drastically reduces the overhead of calling the synchronized method every time.


=== "Factory Method Pattern"

    __Principles__:

    - __Creator__: This is an abstract class or an interface that declares the factory method. The creator typically contains a method that serves as a factory for creating objects. It may also contain other methods that work with the created objects.
    - __Concrete Creator__: Concrete Creator classes are subclasses of the Creator that implement the factory method to create specific types of objects. Each Concrete Creator is responsible for creating a particular product.
    - __Product__: This is the interface or abstract class for the objects that the factory method creates. The Product defines the common interface for all objects that the factory method can create.
    - __Concrete Product__: Concrete Product classes are the actual objects that the factory method creates. Each Concrete Product class implements the Product interface or extends the Product abstract class.

    __Use Cases__:

    - If object creation process is complex or varies under different conditions, using a factory method can make your client code simpler and promote reusability.
    - Since objects are created through interface or abstract class, hiding the details of concrete implementations. This reduces dependencies and makes it easier to modify or expand the system without affecting existing code.
    - Provides flexible way to handle variations of product or introduction of new types, by defining specific factory methods for each product type.
    - Factories can also encapsulate configuration logic, allowing clients to customize the object creation process by providing parameters or options to the factory method.
    - Log4j: configurable loggers

    ```{.java .copy}
    // Product Interface
    abstract class Vehicle {
        public abstract void printVehicle();
    }

    class TwoWheeler extends Vehicle {
        public void printVehicle() {
            System.out.println("I am two wheeler");
        }
    }

    class FourWheeler extends Vehicle {
        public void printVehicle() {
            System.out.println("I am four wheeler");
        }
    }

    // Factory Interface
    interface VehicleFactory {
        Vehicle createVehicle();
    }

    // Concrete Factory for TwoWheeler
    class TwoWheelerFactory implements VehicleFactory {
        public Vehicle createVehicle() {
            return new TwoWheeler();
        }
    }

    // Concrete Factory for FourWheeler
    class FourWheelerFactory implements VehicleFactory {
        public Vehicle createVehicle() {
            return new FourWheeler();
        }
    }

    // Client class
    class Client {
        private Vehicle pVehicle;

        public Client(VehicleFactory factory) {
            pVehicle = factory.createVehicle();
        }

        public Vehicle getVehicle() {
            return pVehicle;
        }
    }

    // Driver program
    public class GFG {
        public static void main(String[] args) {
            VehicleFactory twoWheelerFactory = new TwoWheelerFactory();
            Client twoWheelerClient = new Client(twoWheelerFactory);
            Vehicle twoWheeler = twoWheelerClient.getVehicle();
            twoWheeler.printVehicle();

            VehicleFactory fourWheelerFactory = new FourWheelerFactory();
            Client fourWheelerClient = new Client(fourWheelerFactory);
            Vehicle fourWheeler = fourWheelerClient.getVehicle();
            fourWheeler.printVehicle();
        }
    }
    ```


=== "Builder Pattern"

    __Principles__:

    - __Product__: The Product is the complex object that the Builder pattern is responsible for constructing.
        - It may consist of multiple components or parts, and its structure can vary based on the implementation.
        - The Product is typically a class with attributes representing the different parts that the Builder constructs.
    - __Builder__: An interface or an abstract class that declares the construction steps for building a complex object.
        - Typically includes methods for constructing individual parts of the product.
    - __Concrete Builder__: implements the Builder interface, provids specific implementations for building each part of the product.
        - Each ConcreteBuilder is custmized to create a specific variation of the product.
        - It keeps track of the product being constructed and provides methods for setting or constructing each part.
    - __Director(Optional)__: The Director is responsible for managing the construction process of the complex object.
        - Collaborates with a Builder, but it doesn’t know the specific details about how each part of the object is constructed.
        - Provides a high-level interface for constructing the product and managing the steps needed to create the complex object.
    - __Client__: The Client is the code that initiates the construction of the complex object.
        - It creates a Builder object and passes it to the Director to initiate the construction process.
        - The Client may retrieve the final product from the Builder after construction is complete.

    ```{.java .copy}
    // Product
    class Computer {
        private string cpu;
        private string ram;
        private string storage;

        public void setCPU(String cpu) { cpu = cpu; }

        public void setRAM(String ram) { ram = ram; }

        public void setStorage(String storage) { storage = storage; }

        public void displayInfo() {
            System.out.println(STR."Computer Configuration: CPU: \{cpu}, RAM: \{ram}, Storage: \{storage}");
        }
    }

    // Builder interface
    interface Builder {
        public void buildCPU();
        public void buildRAM();
        public void buildStorage();
        public Computer getResult();
    }

    // ConcreteBuilder
    class GamingComputerBuilder implements Builder {
        private Computer computer;

        public void buildCPU() { computer.setCPU("Gaming CPU"); }

        public void buildRAM() { computer.setRAM("16GB DDR4"); }

        public void buildStorage() { computer.setStorage("1TB SSD"); }

        public Computer getResult() { return computer; }
    }

    // Director
    class ComputerDirector {
        public void construct(Builder builder) {
            builder.buildCPU();
            builder.buildRAM();
            builder.buildStorage();
        }
    }

    // Client
    class Client {
        ComputerDirector director;

        public Client() {
            this.director = new ComputerDirector();
        }

        public Computer getGammingComputer() {
            Builder gamingBuilder = new GamingComputerBuilder();
            director.construct(gamingBuilder);
            Computer gamingComputer = gamingBuilder.getResult();
            return gamingComputer;
        }
    }
    ```