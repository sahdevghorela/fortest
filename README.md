==> Item-1 - Consider static factory methods instead of constructors
Advantage 1 - One advantage of static factory methods is that, unlike constructors, they have names.
Advantage 2 - static factory methods is that, unlike constructors,
they are not required to create a new object each time they’re invoked. You can return cached instance (flyweight pattern)
or you can return same instance. Classes that do this are called instance-controlled classes. This can allow an immutable class to 
gaurantee no two equal instances exist. a.equals(b) only if a==b. This can improve performance. Enums provide this gaurantee.
Advantage 3 - unlike constructors, they can return an object of any subtype of their return type.
API can return objects without making their classes public. Hiding implementation classes in this fashion leads to
a very compact API. This technique lends itself to interface-based frameworks.
For eg, the Java Collections Framework has thirty-two convenience implementations of its collection interfaces, providing unmodifiable collections,
synchronized collections, and the like. Nearly all of these implementations are exported via static factory methods in one noninstantiable class (java.util.Collections). The classes of the returned objects are all nonpublic.
Not only can the class of an object returned by a public static factory method be nonpublic, but the class can vary from invocation to invocation depending on
the values of the parameters to the static factory.
The class of the object returned by a static factory method need not even exist at the time the class containing the method is written. Such flexible static factory methods form the basis of service provider frameworks, such as the Java Database Connectivity API (JDBC).
Advantage 4 - static factory methods is that they reduce the verbosity of creating parameterized type instances.
Map<String, List<String>> m = new HashMap<String, List<String>>();
//if hashmap has this static method
public static <K, V> HashMap<K, V> newInstance() {
return new HashMap<K, V>();
}
Map<String, List<String>> m = HashMap.newInstance();
Disadvantage 1 - The main disadvantage of providing only static factory methods is that classes without public or protected constructors cannot be subclassed.

==> Item-2 - Consider a builder when faced with many constructor parameters.

==> Item-3 - Enforce the singleton property with a private constructor or an enum type.
it is not sufficient merely to add implements Serializable to its declaration. To maintain the singleton guarantee, you
have to declare all instance fields transient and provide a readResolve method (Item 77). Otherwise, each time a serialized instance is deserialized, a new
instance will be created.
private Object readResolve() { return INSTANCE; }
As of release 1.5, there is a third approach to implementing singletons. Simply
make an enum type with one element:
public enum Elvis {
INSTANCE;
public void leaveTheBuilding() { ... }
}
This approach is functionally equivalent to the public field approach, except that it is more concise, provides the serialization machinery for free, and provides an ironclad guarantee against multiple instantiation, even in the face of sophisticated serialization or reflection attacks

==> Item-4 - Enforce noninstantiability with a private constructor
Utility classes should not be designed to be instantiated:
public class UtilityClass {
private UtilityClass() {
throw new AssertionError();
}
}

==> Item-5 - Avoid creating unnecessary objects
String s = new String("stringette"); // DON'T DO THIS!
You can often avoid creating unnecessary objects by using static factory methods (Item 1) in preference to constructors on immutable classes that provide both.
For example, the static factory method Boolean.valueOf(String) is almost always preferable to the constructor Boolean(String). The constructor creates a
new object each time it’s called, while the static factory method is never required to do so and won’t in practice.

public static void main(String[] args) {
Long sum = 0L; 
for (long i = 0; i < Integer.MAX_VALUE; i++) {
sum += i; // unnecessary creation of objects, slower program
}
System.out.println(sum);
}

==> Item-6 - Eliminate obsolete object references
The best way to eliminate an obsolete reference is to let the variable that contained the reference fall out of scope. This occurs naturally if you define each variable in the narrowest possible scope.
Generally speaking, whenever a class manages its own memory, the programmer should be alert for memory leaks. Whenever an element is freed, any
object references contained in the element should be nulled out.
Another common source of memory leaks is caches.If you’re lucky enough to implement a cache for which an entry is relevant exactly
so long as there are references to its key outside of the cache, represent the cache as a WeakHashMap; entries will be removed automatically after they become obsolete. Remember that WeakHashMap is useful only if the desired lifetime of cache entries is determined by external references to the key, not the value.
A third common source of memory leaks is listeners and other callbacks. The best way to ensure that callbacks are garbage collected promptly is to store only weak references to them, for instance, by storing them only as keys in a WeakHashMap.

==> Item-7 - Avoid finalizers. never depend on a finalizer to update critical persistent state.
there is a severe performance penalty for using finalizers. On my machine, the time to create and destroy a simple object is about
5.6 ns. Adding a finalizer increases the time to 2,400 ns. In other words, it is about 430 times slower to create and destroy objects with finalizers.
It is important to note that “finalizer chaining” is not performed automatically. If a class (other than Object) has a finalizer and a subclass overrides it, the subclass finalizer must invoke the superclass finalizer manually. You should finalize
the subclass in a try block and invoke the superclass finalizer in the corresponding finally block.
So what, if anything, are finalizers good for? There are perhaps two legitimate uses. One is to act as a “safety net” in case the owner of an object forgets to call its explicit termination method. While there’s no guarantee that the finalizer will be invoked promptly, it may be better to free the resource late than never, in those (hopefully rare) cases when the client fails to call the explicit termination method.

=============================================================================================================================================================

==> Item-8 - Obey the general contract when overriding equals
The easiest way to avoid problems is not to override the equals method, This is the right thing to do if any of the following conditions apply
Each instance of the class is inherently unique. This is true for classes such as Thread that represent active entities rather than values
You don’t care whether the class provides a “logical equality” test
A superclass has already overridden equals, and the superclass behavior is appropriate for this class.
The class is private or package-private, and you are certain that its equals method will never be invoked.
when is it appropriate to override Object.equals? When a class has a notion of logical equality that differs from mere object identity, and a superclass
has not already overridden equals to implement the desired behavior. This is generally the case for value classes. A value class is simply a class that represents a value, such as Integer or Date.
One kind of value class that does not require the equals method to be overridden is a class that uses instance control (Item 1) to ensure that at most one object
exists with each value. Enum types (Item 30) fall into this category.
Equals method contract:
Reflexive: x.equals(x) must return true.
Symmetry: x.equals(y) then y.equals(x)
Transitivity: x.equals(y), y.equals(z) then x.equals(z)
Consider the case of a subclass that adds a new value component to its superclass. In other words, the subclass adds a piece of information that affects equals comparisons
public class Point {
	private final int x;
	private final int y;
	public Point(int x, int y) {
		this.x = x;
		this.y = y;
	}
	@Override 
	public boolean equals(Object o) {
		if (!(o instanceof Point))
			return false;
			Point p = (Point)o;
			return p.x == x && p.y == y;
	}
	... // Remainder omitted
}
public class ColorPoint extends Point {
	private final Color color;
	public ColorPoint(int x, int y, Color color) {
		super(x, y);
		this.color = color;
	}
	... // Remainder omitted
}
1. // Broken - violates symmetry!
@Override public boolean equals(Object o) {
	if (!(o instanceof ColorPoint))
	return false;
	return super.equals(o) && ((ColorPoint) o).color == color;
}
2. // Broken - violates transitivity!
@Override public boolean equals(Object o) {
	if (!(o instanceof Point))
	return false;
	// If o is a normal Point, do a color-blind comparison
	if (!(o instanceof ColorPoint))
	return o.equals(this);
	// o is a ColorPoint; do a full comparison
	return super.equals(o) && ((ColorPoint)o).color == color;
}

There is no way to extend an instantiable class and add a value component while preserving the equals contract.
// Broken - violates Liskov substitution principle (page 40)
@Override 
public boolean equals(Object o) {
	if (o == null || o.getClass() != getClass())
		return false;
		Point p = (Point) o;
		return p.x == x && p.y == y;
}
The Liskov substitution principle says that any important property of a type should also hold for its subtypes, so that any method written for the type should
work equally well on its subtypes
While there is no satisfactory way to extend an instantiable class and add a value component, there is a fine workaround. Follow the advice of Item 16, “Favor
composition over inheritance.” Instead of having ColorPoint extend Point, give ColorPoint a private Point field and a public view method.
// Adds a value component without violating the equals contract
public class ColorPoint {
	private final Point point;
	private final Color color;
	public ColorPoint(int x, int y, Color color) {
		if (color == null)
		throw new NullPointerException();
		point = new Point(x, y);
		this.color = color;
	}
/**
* Returns the point-view of this color point.
*/
public Point asPoint() {
return point;
}
@Override 
public boolean equals(Object o) {
	if (!(o instanceof ColorPoint))
	return false;
	ColorPoint cp = (ColorPoint) o;
	return cp.point.equals(point) && cp.color.equals(color);
}
}
Note that you can add a value component to a subclass of an abstract class without violating the equals contract.
Consistency: if two objects are equal, they must remain equal for all time unless one (or both) of them is modified.
Non-nullity: This test is unnecessary
if (o == null)
return false;
Following instanceof check if sufficient for null check as well
if (!(o instanceof MyType))
return false;

==> Item-9 - Always override hashCode when you override equals. 
You must override hashCode in every class that overrides equals.
equal objects must have equal hash codes but unequal objects may have equal hash code. (A good hash function tends to produce unequal hash codes for unequal
objects.)
Here is a simple recipe:
1. Store some constant nonzero value, say, 17, in an int variable called result.
2. For each significant field f in your object (each field taken into account by the equals method, that is), do the following:
	a. Compute an int hash code c for the field:
		i. If the field is a boolean, compute (f ? 1 : 0).
		ii. If the field is a byte, char, short, or int, compute (int) f.
		iii. If the field is a long, compute (int) (f ^ (f >>> 32)).
		iv. If the field is a float, compute Float.floatToIntBits(f).
		v. If the field is a double, compute Double.doubleToLongBits(f), and then hash the resulting long as in step 2.a.iii.
		vi. If the field is an object reference and this class’s equals method compares the field by recursively invoking equals, recursively
			invoke hashCode on the field. If a more complex comparison is required, compute a “canonical representation” for this field and
			invoke hashCode on the canonical representation
	b. Combine the hash code c computed in step 2.a into result as follows:
		result = 31 * result + c;
3. Return result.
Description: The value 17(should be non-zero) is arbitrary. The multiplication in step 2.b makes the result depend on the order of the
fields, yielding a much better hash function if the class has multiple similar fields. For example, if the multiplication were omitted from a String hash function, all anagrams would have identical hash codes. The value 31 was chosen because it is an odd prime. If it were even and the multiplication overflowed, information would be lost, as multiplication by 2 is equivalent to shifting. The advantage of using a prime is less clear, but it is traditional. A nice property of 31 is that the multiplication can be replaced by a shift and a subtraction for better performance: 31 * i == (i << 5) - i. Modern VMs do this sort of optimization automatically.
e.g.
@Override
public int hashCode() {
	int result = 17;
	result = 31 * result + areaCode;
	result = 31 * result + prefix;
	result = 31 * result + lineNumber;
	return result;
}

If a class is immutable and the cost of computing the hash code is significant, you might consider caching the hash code in the object rather than recalculating it each time it is requested. If you believe that most objects of this type will be used as hash keys, then you should calculate the hash code  when the instance is created. Otherwise, you might choose to lazily initialize it the first time hashCode is invoked.
// Lazily initialized, cached hashCode
private volatile int hashCode; // (See Item 71)
@Override 
public int hashCode() {
	int result = hashCode;
	if (result == 0) {
		result = 17;
		result = 31 * result + areaCode;
		result = 31 * result + prefix;
		result = 31 * result + lineNumber;
	hashCode = result;
	}
	return result;
}

==> Item-10 - Always override toString
providing a good toString implementation makes your class much more pleasant to use.
The toString method is automatically invoked when an object is passed to println, printf, the string concatenation operator, or 
assert, or printed by a debugger.
One important decision you’ll have to make when implementing a toString method is whether to specify the format of the return value in the documentation.
It is recommended that you do this for value classes, such as phone numbers or matrices. The advantage of specifying the format is that it serves as a standard,
unambiguous, human-readable representation of the object.
The disadvantage of specifying the format of the toString return value is that once you’ve specified it, you’re stuck with it for life, assuming your class is
widely used. Programmers will write code to parse the representation, to generate it, and to embed it into persistent data. If you change the representation in a future release, you’ll break their code and data, and they will yowl. By failing to specify a format, you preserve the flexibility to add information or improve the format in a subsequent release.

==> Item-11 - Override clone judiciously
The Cloneable interface was intended as a mixin interface (Item 18) for objects to advertise that they permit cloning. Unfortunately, it fails to serve this purpose. Its primary flaw is that it lacks a clone method, and Object’s clone method is protected. You cannot, without resorting to reflection (Item 53), invoke the clone method on an object merely because it implements Cloneable.

Normally, implementing an interface says something about what a class can do for its clients. In the case of Cloneable, it modifies the behavior of a protected method on a superclass.
So what does Cloneable do, given that it contains no methods? It determines the behavior of Object’s protected clone implementation: if a class implements
Cloneable, Object’s clone method returns a field-by-field copy of the object; otherwise it throws CloneNotSupportedException.This is a highly atypical use
of interfaces and not one to be emulated.
The resulting mechanism is extralinguistic: it creates an object without calling a constructor.
The general contract for the clone method is weak.
x.clone() != x will be true, and the expression
x.clone().getClass() == x.getClass() will be true, but these are not absolute requirements.
While it is typically the case that x.clone().equals(x) will be true, this is not an absolute requirement.

if you override the clone method in a nonfinal class, you should return an object obtained by invoking super.clone. If all of a class’s superclasses
obey this rule, then invoking super.clone will eventually invoke Object’s clone method, creating an instance of the right class. This mechanism is
vaguely similar to automatic constructor chaining, except that it isn’t enforced

The Cloneable interface does not, as of release 1.6, spell out in detail the responsibilities that a class takes on when it implements this interface. In practice, a class that implements Cloneable is expected to provide a properly functioning public clone method. It is not, in general, possible to do so unless
all of the class’s superclasses provide a well-behaved clone implementation, whether public or protected.

If every field contains a  primitive value or a reference to an immutable object, the returned object may be exactly what you need, in which case no further processing is necessary.
@Override 
public PhoneNumber clone() {
	try {
		return (PhoneNumber) super.clone();
	} catch(CloneNotSupportedException e) {
		throw new AssertionError(); // Can't happen
	}
}
If an object contains fields that refer to mutable objects, using the simple clone implementation shown above can be disastrous.
In order for the clone method on Stack to work properly, it must copy the internals of the stack. The easiest way to do this is to call clone recursively on the elements array:
@Override 
public Stack clone() {
	try {
		Stack result = (Stack) super.clone();
		result.elements = elements.clone();
		return result;
	} catch (CloneNotSupportedException e) {
		throw new AssertionError();
	}
}

Note also that the above solution would not work if the elements field were final, because clone would be prohibited from assigning a new value to the field.
This is a fundamental problem: the clone architecture is incompatible with normal use of final fields referring to mutable objects, except in cases where
the mutable objects may be safely shared between an object and its clone. In order to make a class cloneable, it may be necessary to remove final modifiers from
some fields.

Entry deepCopy() {
	return new Entry(key, value,
	next == null ? null : next.deepCopy());
}
@Override 
public HashTable clone() {
	try {
		HashTable result = (HashTable) super.clone();
		result.buckets = new Entry[buckets.length];
		for (int i = 0; i < buckets.length; i++)
		if (buckets[i] != null)
		result.buckets[i] = buckets[i].deepCopy();
		return result;
	} catch (CloneNotSupportedException e) {
		throw new AssertionError();
	}
}

// Iteratively copy the linked list headed by this Entry
Entry deepCopy() {
	Entry result = new Entry(key, value, next);
	for (Entry p = result; p.next != null; p = p.next)
		p.next = new Entry(p.next.key, p.next.value, p.next.next);
	return result;
}

Like a constructor, a clone method should not invoke any nonfinal methods on the clone under construction (Item 17). If clone invokes an overridden method,
this method will execute before the subclass in which it is defined has had a chance to fix its state in the clone, quite possibly leading to corruption in the clone and the original.

Object’s clone method is declared to throw CloneNotSupportedException, but overriding clone methods can omit this declaration. Public clone methods
should omit it because methods that don’t throw checked exceptions are easier to use (Item 59). If a class that is designed for inheritance (Item 17) overrides clone, the overriding method should mimic the behavior of Object.clone: it should be declared protected, it should be declared to throw CloneNotSupportedException, and the class should not implement Cloneable. This gives subclasses the freedom to implement Cloneable or not, just as if they extended Object directly.

If the class contains only primitive fields or references to immutable objects, then it is probably the case that no fields need to be fixed. There are exceptions to this rule. For example, a field representing a serial number or other unique ID or a field representing the object’s creation time will need to be fixed, even if it is primitive or immutable.

Is all this complexity really necessary? Rarely. If you extend a class that implements Cloneable, you have little choice but to implement a well-behaved
clone method. Otherwise, you are better off providing an alternative means of object copying, or simply not providing the capability. For example, it doesn’t
make sense for immutable classes to support object copying, because copies would be virtually indistinguishable from the original.

A fine approach to object copying is to provide a copy constructor or copy factory.
public Yum(Yum yum); or public static Yum newInstance(Yum yum);
The copy constructor approach and its static factory variant have many advantages over Cloneable/clone
they don’t rely on a risk-prone extralinguistic object creation mechanism; they don’t demand unenforceable adherence to thinly
documented conventions; they don’t conflict with the proper use of final fields; they don’t throw unnecessary checked exceptions; and they don’t require casts.
While it is impossible to put a copy constructor or factory in an interface, Cloneable fails to function as an interface because it lacks a public clone
method. 
Furthermore, a copy constructor or factory can take an argument whose type is an interface implemented by the class. For example, by convention all generalpurpose collection implementations provide a constructor whose argument is of type Collection or Map.
Suppose you have a HashSet s, and you want to copy it as a TreeSet. The clone method can’t offer this functionality, but it’s easy with a conversion constructor: new TreeSet(s).

==> Item-12 - Consider implementing Comparable
It is similar in character to Object’s equals method, except that it permits order comparisons in addition to simple equality comparisons, and it is generic. By implementing Comparable, a class indicates that its instances have a natural ordering.
public class WordList {
	public static void main(String[] args) {
		Set<String> s = new TreeSet<String>();
		Collections.addAll(s, args);
		System.out.println(s);
	}
}
Virtually all of the value classes in the Java platform libraries implement Comparable. If you are writing a value class with an obvious natural ordering, such as alphabetical order, numerical order, or chronological order, you should strongly consider implementing the interface:

One consequence of these three provisions is that the equality test imposed by a compareTo method must obey the same restrictions imposed by the equals contract:
reflexivity, symmetry, and transitivity. Therefore the same caveat applies: there is no way to extend an instantiable class with a new value component while
preserving the compareTo contract, unless you are willing to forgo the benefits of object-oriented abstraction (Item 8). The same workaround applies, too. If you
want to add a value component to a class that implements Comparable, don’t extend it; write an unrelated class containing an instance of the first class. Then
provide a “view” method that returns this instance.

The final paragraph of the compareTo contract, which is a strong suggestion rather than a true provision, simply states that the equality test imposed by the
compareTo method should generally return the same results as the equals method. If this provision is obeyed, the ordering imposed by the compareTo
method is said to be consistent with equals. If it’s violated, the ordering is said to be inconsistent with equals. A class whose compareTo method imposes an order that is inconsistent with equals will still work, but sorted collections containing elements of the class may not obey the general contract of the appropriate collection interfaces (Collection, Set, or Map). This is because the general contracts for these interfaces are defined in terms of the equals method, but sorted collections use the equality test imposed by compareTo in place of equals. It is not a catastrophe if this happens, but it’s something to be aware of.

For example, consider the BigDecimal class, whose compareTo method is inconsistent with equals. If you create a HashSet instance and add new
BigDecimal("1.0") and new BigDecimal("1.00"), the set will contain two elements because the two BigDecimal instances added to the set are unequal
when compared using the equals method. If, however, you perform the same procedure using a TreeSet instead of a HashSet, the set will contain only one
element because the two BigDecimal instances are equal when compared using the compareTo method.

public final class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {
	
	public int compareTo(CaseInsensitiveString cis) {
		return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
	}
}
Compare integral primitive fields using the relational operators < and >. For floating-point fields, use Double.compare or Float.compare in place of the
relational operators, which do not obey the general contract for compareTo when applied to floating point values. For array fields, apply these guidelines to each element.
If a class has multiple significant fields, the order in which you compare them is critical. You must start with the most significant field and work your way down.

int areaCodeDiff = areaCode - pn.areaCode;
if (areaCodeDiff != 0)
return areaCodeDiff;

This trick works fine here but should be used with extreme caution. Don’t use it unless you’re certain the fields in question are non-negative or, more generally, that the difference between the lowest and highest possible field values is less than or equal to Integer.MAX_VALUE (231-1). The reason this trick doesn’t always work is that a signed 32-bit integer isn’t big enough to hold the difference between two arbitrary signed 32-bit integers. If i is a large positive int and j is a large negative int, (i - j) will overflow and return a negative value

=============================================================================================================================================================
#### Classes and interfaces ######
==> Item-13 - Minimize the accessibility of classes and members
The single most important factor that distinguishes a well-designed module from a poorly designed one is the degree to which the module hides its internal data and other implementation details from other modules. A well-designed module hides all of its implementation details, cleanly separating its API from its implementation. This concept, known as information hiding or encapsulation, is one of the fundamental tenets of software design.
Information hiding is important for many reasons, most of which stem from the fact that it decouples the modules that comprise a system, allowing them to be
developed, tested, optimized, used, understood, and modified in isolation.
once a system is complete and profiling has determined which modules are causing performance problems (Item 55), those modules can be optimized without affecting the correctness of other modules
The rule of thumb is simple: make each class or member as inaccessible as possible.
If you declare a top-level class or interface with the public modifier, it will be public; otherwise, it will be package-private. If a top-level class or interface can be made package-private, it should be. By making it package-private, you make it part of the implementation rather than the exported API, and you can modify it, replace it, or eliminate it in a subsequent release without fear of harming existing clients.
If a package-private top-level class (or interface) is used by only one class,  consider making the top-level class a private nested class of the sole class that uses it (Item 22).
both private and package-private members are part of a class’s implementation and do not normally impact its exported API. These fields can, however, “leak” into the exported API if the class implements Serializable (Item 74, Item 75).
A protected member is part of the class’s exported API and must be supported forever.The need for protected members should be relatively rare.
If a method overrides a superclass method, it is not permitted to have a lower access level in the subclass than it does in the superclass  This is
necessary to ensure that an instance of the subclass is usable anywhere that an instance of the superclass is usable.
Instance fields should never be public (Item 14). If an instance field is nonfinal, or is a final reference to a mutable object, then by making the field public,
you give up the ability to limit the values that can be stored in the field.
Also, you give up the ability to take any action when the field is modified, so classes with public mutable fields are not thread-safe. Even if a field is final and refers to an immutable object, by making the field public you give up the flexibility to switch to a new internal data representation in which the field does not exist.
It is critical that public static final fields contain either primitive values or references to immutable objects (Item 15). A final field containing a reference to a mutable object has all the disadvantages of a nonfinal field. While the reference cannot be modified, the referenced object can be modified—with disastrous results.
Note that a nonzero-length array is always mutable, so it is wrong for a class to have a public static final array field, or an accessor that returns such a
field. There are two ways to fix the problem. You can make the public array private and add a public immutable list:
private static final Thing[] PRIVATE_VALUES = { ... };
public static final List<Thing> VALUES =
Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES)); 
OR
private static final Thing[] PRIVATE_VALUES = { ... };
public static final Thing[] values() {
return PRIVATE_VALUES.clone();
}

==> Item-14 - In public classes, use accessor methods, not public fields
Because the data fields of such classes are accessed directly, these classes do not offer the benefits of encapsulation (Item 13). You can’t change the representation without changing the API, you can’t enforce invariants, and you can’t take auxiliary action when a field is accessed.
It is less harmful, though still questionable, for public classes to expose immutable fields.

==> Item-15 - Minimize mutability
All of the information contained in each instance is provided when it is created and is fixed for the lifetime of the object. The Java platform libraries contain many immutable classes, including String, the boxed primitive classes, and BigInteger and BigDecimal.
follow these five rules:
1. Don’t provide any methods that modify the object’s state (known as mutators).
2. Ensure that the class can’t be extended. This prevents careless or malicious subclasses from compromising the immutable behavior of the class by behaving
as if the object’s state has changed. Preventing subclassing is generally accomplished by making the class final, but there is an alternative that we’ll
discuss later.
3. Make all fields final: to enforce non-change. Also, it is necessary to ensure correct behavior if a reference to a newly created instance is passed from one thread to another without synchronization.
4. Make all fields private.
5. Ensure exclusive access to any mutable components. Never initialize such a field to a client-provided object reference.Make defensive copies (Item 39) in constructors, accessors, and readObject methods.

Immutable objects are inherently thread-safe; they require no synchronization.
Therefore, immutable objects can be shared freely. Immutable classes should take advantage of this by encouraging clients to reuse existing instances wherever
possible. One easy way to do this is to provide public static final constants for frequently used values.
public static final Complex ZERO = new Complex(0, 0); public static final Complex ONE = new Complex(1, 0);
This approach can be taken one step further. An immutable class can provide static factories (Item 1) that cache frequently requested instances to avoid creating
new instances when existing ones would do. All the boxed primitive classes and BigInteger do this. Using such static factories causes clients to share instances
instead of creating new ones, reducing memory footprint and garbage collection costs. Opting for static factories in place of public constructors when designing a new class gives you the flexibility to add caching later, without modifying clients.

A consequence of the fact that immutable objects can be shared freely is that you never have to make defensive copies (Item 39). Therefore, you need not and should not provide a clone method or copy constructor (Item 11) on an immutable class. This was not well understood in the early days of the Java platform, so the String class does have a copy constructor, but it should rarely, if ever, be used (Item 5).

Not only can you share immutable objects, but you can share their internals. For example, the BigInteger class uses a sign-magnitude representation
internally. The sign is represented by an int, and the magnitude is represented by an int array. The negate method produces a new BigInteger of like magnitude
and opposite sign. It does not need to copy the array; the newly created BigInteger points to the same internal array as the original.
Immutable objects make great building blocks for other objects. A special case of this principle is that immutable objects make great map keys and
set elements: you don’t have to worry about their values changing once they’re in the map or set, which would destroy the map or set’s invariants.

The only real disadvantage of immutable classes is that they require a separate object for each distinct value.
The performance problem is magnified if you perform a multistep operation that generates a new object at every step, eventually discarding all objects except
the final result. There are two approaches to coping with this problem.
The first is to guess which multistep operations will be commonly required and provide them as primitives. If a multistep operation is provided as a primitive, the immutable class does not have to create a separate object at each step.
Secondaly, if you dont know which multistep operation will be ofteen used by the client, you can provide a public mutable companion class. The main example of this approach in the Java platform libraries is the String class, whose mutable companion is StringBuilder. BitSet plays the role of mutable companion to BigInteger under certain circumstances.

The alternative to making an immutable class final is to make all of its constructors private or package-private, and to add public static factories in
place of the public constructors. While this approach is not commonly used, it is often the best alternative. It is the most flexible because it allows the use of multiple package-private implementation classes. this approach makes it possible to tune the performance of the class in subsequent releases by improving the objectcaching capabilities of the static factories.

It was not widely understood that immutable classes had to be effectively final when BigInteger and BigDecimal were written, so all of their methods may be 
overridden. If you write a class whose security depends on the  immutability of a BigInteger or BigDecimal argument from an untrusted client, you must check to see that the argument is a “real” BigInteger or BigDecimal, rather than an instance of an untrusted subclass. If it is the latter, you must defensively
copy it under the assumption that it might be mutable.
public static BigInteger safeInstance(BigInteger val) {
	if (val.getClass() != BigInteger.class)
		return new BigInteger(val.toByteArray());
		return val;
}

some immutable classes have one or more nonfinal fields in which they cache the results of expensive computations the first time they are needed. If the
same value is requested again, the cached value is returned, saving the cost of recalculation.
One caveat should be added concerning serializability. If you choose to have your immutable class implement Serializable and it contains one or more fields
that refer to mutable objects, you must provide an explicit readObject or readResolve method, or use the ObjectOutputStream.writeUnshared and ObjectInputStream.readUnshared methods, even if the default serialized form is acceptable. Otherwise an attacker could create a mutable instance of your notquite- immutable class.
java.util.Date and java.awt.Point, that should have been immutable but aren’t. If a class cannot be made immutable, limit its mutability as much as possible. Reducing the number of states in which an object can exist makes it easier to reason about the object and reduces the likelihood of errors.

==> Item-16 - Favor composition over inheritance
Inheritance is a powerful way to achieve code reuse, but it is not always the best tool for the job. Used inappropriately, it leads to fragile software. It is safe to use inheritance within a package, where the subclass and the superclass implementations are under the control of the same programmers. It is also safe to use inheritance when extending classes specifically designed and documented for extension.
Unlike method invocation, inheritance violates encapsulation [Snyder86]. In other words, a subclass depends on the implementation details of its superclass
for its proper function. The superclass’s implementation may change from release to release, and if it does, the subclass may break, even though its code has not
been touched.
Instead
of extending an existing class, give your new class a private field that references an instance of the existing class.
Inheritance is appropriate only in circumstances where the subclass really is a subtype of the superclass. In other words, a class B should extend a class A only if an “is-a” relationship exists between the two classes. There are a number of obvious violations of this principle in the Java platform
libraries. For example, a stack is not a vector, so Stack should not extend Vector. Similarly, a property list is not a hash table, so Properties should not extend Hashtable.
Does the class that you contemplate extending have any flaws in its API? If so, are you comfortable propagating those flaws into your class’s API? Inheritance propagates any flaws in the superclass’s API, while composition lets you design a new API that hides these flaws.

==> Item-17 - Design and document for inheritance or else prohibit it
First, the class must document precisely the effects of overriding any method. In other words, the class must document its self-use of overridable methods.
For each public or protected method or constructor, the documentation must indicate which overridable methods the method or constructor invokes, in what
sequence, and how the results of each invocation affect subsequent processing.
But doesn’t this violate the dictum that good API documentation should describe what a given method does and not how it does it? Yes, it does! This is an 
unfortunate consequence of the fact that inheritance violates encapsulation. To document a class so that it can be safely subclassed, you must describe implementation details that should otherwise be left unspecified.
The only way to test a class designed for inheritance is to write subclasses.Experience shows that three subclasses are usually sufficient to test an extendable class. One or more of these subclasses should be written by someone other than the superclass author.
There are a few more restrictions that a class must obey to allow inheritance. Constructors must not invoke overridable methods, directly or indirectly.
public class Super {
	// Broken - constructor invokes an overridable method
	public Super() {
		overrideMe();
	}
	public void overrideMe() {
	}
}
public final class Sub extends Super {
	private final Date date; // Blank final, set by constructor
	Sub() {
		date = new Date();
	}
	// Overriding method invoked by superclass constructor
	@Override 
	public void overrideMe() {
		System.out.println(date);
	}
	public static void main(String[] args) {
		Sub sub = new Sub();
		sub.overrideMe();
	}
}
The Cloneable and Serializable interfaces present special difficulties when designing for inheritance. It is generally not a good idea for a class designed
for inheritance to implement either of these interfaces, as they place a substantial burden on programmers who extend the class.
If you do decide to implement Cloneable or Serializable in a class designed for inheritance, neither clone nor readObject may invoke an overridable method, directly or indirectly.
But what about ordinary concrete classes? Traditionally, they are neither final nor designed and documented for subclassing, but this state of affairs is dangerous. Each time a change is made in such a class, there is a chance that client classes that extend the class will break.There are two ways
to prohibit subclassing. The easier of the two is to declare the class final. The alternative is to make all the constructors private or package-private and to add public static factories in place of the constructors.

==> Item-18 - Prefer interfaces to abstract classes

1. Existing classes can be easily retrofitted to implement a new interface. All you have to do is add the required methods if they don’t yet exist and add an
implements clause to the class declaration. For example, many existing classes were retrofitted to implement the Comparable interface when it was introduced
into the platform. Existing classes cannot, in general, be retrofitted to extend a new abstract class.
2. Interfaces are ideal for defining mixins. Loosely speaking, a mixin is a type that a class can implement in addition to its “primary type” to declare that it provides some optional behavior. For example, Comparable is a mixin interface that allows a class to declare that its instances are ordered with respect to other mutually comparable objects. Such an interface is called a mixin because it allows the optional functionality to be “mixed in” to the type’s primary functionality.
3. Interfaces allow the construction of nonhierarchical type frameworks. For example, suppose we have an interface representing a singer and another representing a songwriter. In real life, some singers are also songwriters. Because we used interfaces rather than abstract classes to define these types, it is perfectly permissible for a single class to implement both Singer and Songwriter.
When you need this level of flexibility interfaces are a lifesaver.If there are n attributes in the type system, there are 2n possible combinations that you might have to support. This is what’s known as a combinatorial explosion. Bloated class hierarchies can lead to bloated classes containing many methods that differ only in the type of their arguments,  as there are no types in the class hierarchy to capture common behaviors.
4. Interfaces enable safe, powerful functionality enhancements via the wrapper class idiom.
While interfaces are not permitted to contain method implementations, using interfaces to define types does not prevent you from providing implementation
assistance to programmers. You can combine the virtues of interfaces and abstract classes by providing an abstract skeletal implementation class to go
with each nontrivial interface that you export.
By convention, skeletal implementations are called AbstractInterface. For example, the Collections Framework provides a skeletal implementation to go along with each main collection interface: AbstractCollection, AbstractSet, AbstractList.
If a preexisting class cannot be made to extend the skeletal implementation, the class can always implement the interface manually.
Furthermore, the skeletal implementation can still aid the implementor’s task. The class implementing the interface can forward invocations of interface methods to a contained instance of a private inner class that extends the skeletal implementation. This technique is known as simulated multiple inheritance.

5. Using abstract classes to define types that permit multiple implementations has one great advantage over using interfaces: It is far easier to evolve an
abstract class than an interface. If, in a subsequent release, you want to add a new method to an abstract class, you can always add a concrete method containing
a reasonable default implementation. All existing implementations of the abstract class will then provide the new method.Its impossible to add a method to a public interface without breaking all existing classes that implement the interface. Public interfaces, therefore, must be designed carefully. Once an interface is released and widely implemented, it is almost impossible to change. 
The best thing to do when releasing a new interface is to have as many programmers as possible implement the interface in as many ways as possible before the
interface is frozen

==> Item-19 - Use interfaces only to define types
constant interface contains no methods; it consists solely of static final fields, each exporting a constant.
// Constant interface antipattern - do not use!
public interface PhysicalConstants {
// Avogadro's number (1/mol)
static final double AVOGADROS_NUMBER = 6.02214199e23;
}
If you want to export constants, there are several reasonable choices. If the constants are strongly tied to an existing class or interface, you should add them to the class or interface. If the constants are best viewed as members of an enumerated type, you should export them with an enum type. 
Otherwise, you should export the constants with a noninstantiable utility class.

==> Item-20 - Prefer class hierarchies to tagged classes
Occasionally you may run across a class whose instances come in two or more flavors and contain a tag field indicating the flavor of the instance.
// Tagged class - vastly inferior to a class hierarchy!
class Figure {
enum Shape { RECTANGLE, CIRCLE };
// Tag field - the shape of this figure
final Shape shape;
// These fields are used only if shape is RECTANGLE
double length;
double width;
// This field is used only if shape is CIRCLE
double radius;
double area() {
	switch(shape) {
	case RECTANGLE:
		return length * width;
	case CIRCLE:
		return Math.PI * (radius * radius);
	default:
		throw new AssertionError();
	}
}
}
Such tagged classes have numerous shortcomings. They are cluttered with boilerplate, including enum declarations, tag fields, and switch statements. Readability
is further harmed because multiple implementations are jumbled together in a single class. Memory footprint is increased because instances are burdened
with irrelevant fields belonging to other flavors
To transform a tagged class into a class hierarchy, first define an abstract class containing an abstract method for each method in the tagged class whose behavior depends on the tag value. Next, define a concrete subclass of the root class for each flavor of the original tagged class.
abstract class Figure {
abstract double area();
} 
class Circle extends Figure
class Rectangle extends Figure

==> Item-21 - Use function objects to represent strategies
Some languages support function pointers, delegates, lambda expressions, or similar facilities that allow programs to store and transmit the ability to invoke a particular function. Java does not provide function pointers, but object references can be used to achieve a similar effect.
An instance of a class that exports exactly one such method is effectively a pointer to that method. Such instances are known as function objects.
class StringLengthComparator {
	public int compare(String s1, String s2) {
		return s1.length() - s2.length();
	}
}
In other words, a StringLengthComparator instance is a concrete strategy for string comparison.
Concrete strategy classes are often declared using anonymous classes
Arrays.sort(stringArray, new Comparator<String>() {
	public int compare(String s1, String s2) {
		return s1.length() - s2.length();
	}
});
If it is to be executed repeatedly, consider storing the function object in a private static final field and reusing it.
// Exporting a concrete strategy
class Host {
	private static class StrLenCmp implements Comparator<String>, Serializable {
		public int compare(String s1, String s2) {
			return s1.length() - s2.length();
		}
	}
// Returned comparator is serializable
public static final Comparator<String> STRING_LENGTH_COMPARATOR = new StrLenCmp();
}
The String class uses this pattern to export a case-independent string comparator via its CASE_INSENSITIVE_ORDER field.

==> Item-22 - Favor static member classes over nonstatic
There are four kinds of nested classes: static member classes, nonstatic member classes, anonymous classes, and local classes. All but the first kind are known as inner classes.
A static member class is the simplest kind of nested class. It is best thought of as an ordinary class that happens to be declared inside another class and has
access to all of the enclosing class’s members, even those declared private, obeys the same accessibility rules as other static members.
One common use of a static member class is as a public helper class, useful only in conjunction with its outer class. For example, consider an enum describing
the operations supported by a calculator. like Calculator.Operation.PLUS

Each instance of a nonstatic member class is implicitly associated with an enclosing instance of its containing class. Within instance methods of a nonstatic
member class, you can invoke methods on the enclosing instance or obtain a reference to the enclosing instance using the qualified this construct.it is impossible to create an instance of a nonstatic member class without an enclosing instance.
One common use of a nonstatic member class is to define an Adapter [Gamma95, p. 139] that allows an instance of the outer class to be viewed as an
instance of some unrelated class. For example, implementations of the Map interface typically use nonstatic member classes to implement their collection views,
which are returned by Map’s keySet, entrySet, and values methods. Similarly, implementations of the collection interfaces, such as Set and List, typically use
nonstatic member classes to implement their iterators:
public class MySet<E> extends AbstractSet<E> {
	public Iterator<E> iterator() {
		return new MyIterator();
	}
	private class MyIterator implements Iterator<E> {
	...
	}
}

If you declare a member class that does not require access to an enclosing instance, always put the static modifier in its declaration, making it a static
rather than a nonstatic member class. If you omit this modifier, each instance will have an extraneous reference to its enclosing instance. Storing this reference costs time and space, and can result in the enclosing instance being retained when it would otherwise be eligible for garbage collection.

An anonymous class has no name. It is not a member of its enclosing class.There are many limitations on the applicability of anonymous classes. You
can’t instantiate them except at the point they’re declared. You can’t perform instanceof tests or do anything else that requires you to name the class. You
can’t declare an anonymous class to implement multiple interfaces, or to extend a class and implement an interface at the same time.
Because anonymous classes occur in the midst of expressions, they must be kept short— about ten lines or fewer.One common use of anonymous classes is to create function objects (Item 21) on the fly.Another common use of anonymous classes is to create process objects, such as Runnable, Thread, or TimerTask instances.

Local classes are the least frequently used of the four kinds of nested classes. A local class can be declared anywhere a local variable can be declared and obeys the same scoping rules. Like anonymous classes, they have enclosing instances only if they are defined in a nonstatic context, and they cannot contain static members.
==============================================================================================================================================================

Generics

==> Item-23 - Don’t use raw types in new code
// Use of raw type for unknown element type - don't do this!
static int numElementsInCommon(Set s1, Set s2) {
	int result = 0;
	for (Object o1 : s1)
		if (s2.contains(o1))
			result++;
	return result;
}
This method works but it uses raw types, which are dangerous. Since release 1.5, Java has provided a safe alternative known as unbounded wildcard types.
// Unbounded wildcard type - typesafe and flexible
static int numElementsInCommon(Set<?> s1, Set<?> s2) {
	int result = 0;
	for (Object o1 : s1)
		if (s2.contains(o1))
			result++;
	return result;
}
What is the difference between the unbounded wildcard type Set<?> and the raw type Set?. You can put any element into a collection with a raw type, easily corrupting the collection’s type invariant you can’t put any element (other than null) into a Collection<?>.
There are two minor exceptions to the rule that you should not use raw types in new code, both of which stem from the fact that generic type information is
erased at runtime. You must use raw types in class literals. List.class, String[].class, and int.class are all legal, but List<String>.class and
List<?>.class are not. The second exception to the rule concerns the instanceof operator. Because generic type information is erased at runtime, it is illegal to use the instanceof operator on parameterized types other than unbounded wildcard types. 
// Legitimate use of raw type - instanceof operator
if (o instanceof Set) { // Raw type
Set<?> m = (Set<?>) o; // Wildcard type
}

Some terms: 
Generic type List<E>
Unbounded wildcard type List<?>
Bounded wildcard type List<? extends Number>
Bounded type parameter <E extends Number>
Recursive type bound <T extends Comparable<T>>
Generic method static <E> List<E> asList(E[] a)

==> Item-24 - Eliminate unchecked warnings
Eliminate every unchecked warning that you can. If you eliminate all warnings, you are assured that your code is typesafe, which is a very good thing. It means that you won’t get a ClassCastException at runtime, and it increases your confidence that your program is behaving as you intended.
If you can’t eliminate a warning, and you can prove that the code that provoked the warning is typesafe, then (and only then) suppress the warning
with an @SuppressWarnings("unchecked") annotation. Always use the Suppress-Warnings annotation on the smallest scope possible.
Every time you use an @SuppressWarnings("unchecked") annotation, add a comment saying why it’s safe to do so.

==> Item-25 - Prefer lists to arrays
Arrays differ from generic types in two important ways. First, arrays are covariant. This scary-sounding word means simply that if Sub is a subtype of Super, then the array type Sub[] is a subtype of Super[]. Generics, by contrast, are invariant: for any two distinct types Type1 and Type2, List<Type1> is neither a subtype nor a supertype of List<Type2>.
// Fails at runtime!
Object[] objectArray = new Long[1];
objectArray[0] = "I don't fit in"; // Throws ArrayStoreException
// Won't compile!
List<Object> ol = new ArrayList<Long>(); // Incompatible types
ol.add("I don't fit in");

The second major difference between arrays and generics is that arrays are reified [JLS, 4.7]. This means that arrays know and enforce their element types at
runtime. Generics, by contrast, are implemented by erasure [JLS, 4.6]. This means that they enforce their type constraints only at compile
time and discard (or erase) their element type information at runtime. Erasure is what allows generic types to interoperate freely with legacy code that does not use generics.
Because of these fundamental differences, arrays and generics do not mix well. For example, it is illegal to create an array of a generic type, a parameterized
type, or a type parameter. None of these array creation expressions are legal: new List<E>[], new List<String>[], new E[]. All will result in generic array creation errors at compile time.
The only parameterized types that are reifiable are unbounded wildcard types such as List<?> and Map<?,?>.
The prohibition on generic array creation can be annoying. It means, for example, that it’s not generally possible for a generic type to return an array of its element type (but see Item 29 for a partial solution). When you get a generic array creation error, the best solution is often to use
the collection type List<E> in preference to the array type E[].

// Naive generic version of reduction - won't compile!
static <E> E reduce(List<E> list, Function<E> f, E initVal) {
	E[] snapshot = list.toArray(); // Locks list
	E result = initVal;
	for (E e : snapshot)
		result = f.apply(result, e);
	return result;
}
// warning: [unchecked] unchecked cast
E[] snapshot = (E[]) list.toArray();
// List-based generic reduction
static <E> E reduce(List<E> list, Function<E> f, E initVal) {
	List<E> snapshot;
	synchronized(list) {
		snapshot = new ArrayList<E>(list);
	}
	E result = initVal;
	for (E e : snapshot)
		result = f.apply(result, e);
	return result;
}

==> Item-26 - Favor generic types
// Initial attempt to generify Stack = won’t compile!
public class Stack<E> {
	private E[] elements;
	private int size = 0;
	public Stack() {
		elements = new E[DEFAULT_INITIAL_CAPACITY]; //As explained in Item 25, you can’t create an array of a non-reifiable type, such as E.
	}
	public E pop() {
		if (size==0)
			throw new EmptyStackException();
		E result = elements[--size];
		elements[size] = null; // Eliminate obsolete reference
		return result;
	}
}
There are two ways to solve it. 1. create an array of Object and cast it to the generic array type. Now in place of an error, the compiler will emit a warning.
elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
Once you’ve proved that an unchecked cast is safe, suppress the warning in as narrow a scope as possible.
@SuppressWarnings("unchecked")
public Stack() {
	elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
}
2. The second way to eliminate the generic array creation error in Stack is to change the type of the field elements from E[] to Object[]. If you do this, you’ll
get a different error: E result = elements[--size]; found : Object, required: E
You can change this error into a warning by casting the element retrieved from the array from Object to E:
E result = (E) elements[--size]; Because E is a non-reifiable type, there’s no way the compiler can check the cast at runtime.
@SuppressWarnings("unchecked") 
E result = (E) elements[--size];

The foregoing example may appear to contradict Item 25, which encourages the use of lists in preference to arrays. It is not always possible or desirable to use
lists inside your generic types. Java doesn’t support lists natively, so some generic types, such as ArrayList, must be implemented atop arrays. Other generic types, such as HashMap, are implemented atop arrays for performance.

==> Item-27 - Favor generic methods
Just as classes can benefit from generification, so can methods. Static utility methods are particularly good candidates for generification. All of the “algorithm” methods in Collections (such as binarySearch and sort) have been generified.
// Generic method
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
	Set<E> result = new HashSet<E>(s1);
	result.addAll(s2);
	return result;
}
The compiler figures out the value of the type parameters by examining the types of the method arguments. This process is called "type inference".

A related pattern is the generic singleton factory. On occasion, you will need to create an object that is immutable but applicable to many different types.
Because generics are implemented by erasure (Item 25). This pattern is most frequently used for function objects (Item 21) such as
Collections.reverseOrder, but it is also used for collections such as Collections.emptySet.
public interface UnaryFunction<T> {
T apply(T arg);
}
// Generic singleton factory pattern
private static UnaryFunction<Object> IDENTITY_FUNCTION = new UnaryFunction<Object>() {
public Object apply(Object arg) { return arg; }
};

// IDENTITY_FUNCTION is stateless and its type parameter is
// unbounded so it's safe to share one instance across all types.
@SuppressWarnings("unchecked")
public static <T> UnaryFunction<T> identityFunction() {
	return (UnaryFunction<T>) IDENTITY_FUNCTION;
}
the identity function is special: it returns its argument unmodified, so we know that it is typesafe to use it as a UnaryFunction<T> whatever the value
of T. Therefore, we can confidently suppress the unchecked cast warning that is generated by this cast.

It is permissible, though relatively rare, for a type parameter to be bounded by some expression involving that type parameter itself. This is what’s known as a
recursive type bound. The most common use of recursive type bounds is in connection with the Comparable interface.
// Returns the maximum value in a list - uses recursive type bound
public static <T extends Comparable<T>> T max(List<T> list) {
	Iterator<T> i = list.iterator();
	T result = i.next();
	while (i.hasNext()) {
		T t = i.next();
		if (t.compareTo(result) > 0)
			result = t;
	}
	return result;
}
The type bound <T extends Comparable<T>> may be read as “for every type T that can be compared to itself,”.


==> Item-28 - Use bounded wildcards to increase API flexibility
As noted in Item 25, parameterized types are invariant. Sometimes you need more flexibility than invariant typing can provide. Consider
the stack from Item 26
// pushAll method without wildcard type - deficient!
public void pushAll(Iterable<E> src) {
for (E e : src)
push(e);
}
Stack<Number> numberStack = new Stack<Number>();
Iterable<Integer> integers = ... ;
numberStack.pushAll(integers); //you’ll get error message because, as noted above, parameterized types are invariant.
Luckily, there’s a way out. The language provides a special kind of parameterized type call a bounded wildcard type to deal with situations like this.
// Wildcard type for parameter that serves as an E producer
public void pushAll(Iterable<? extends E> src) {
for (E e : src)
push(e);
}
// popAll method without wildcard type - deficient!
public void popAll(Collection<E> dst) {
while (!isEmpty())
dst.add(pop());
}
Stack<Number> numberStack = new Stack<Number>();
Collection<Object> objects = ... ;
numberStack.popAll(objects);
// Wildcard type for parameter that serves as an E consumer
public void popAll(Collection<? super E> dst) {
while (!isEmpty())
dst.add(pop());
}
The lesson is clear. For maximum flexibility, use wildcard types on input parameters that represent producers or consumers.  If an input parameter is
both a producer and a consumer, then wildcard types will do you no good.
mnemonic to help you remember - PECS stands for producer-extends, consumer-super.

The reduce method in Item 25 has this declaration:
static <E> E reduce(List<E> list, Function<E> f, E initVal)
// Wildcard type for parameter that serves as an E producer
static <E> E reduce(List<? extends E> list, Function<E> f,E initVal) //Suppose you have a List<Integer>, and you want to reduce it with a Function<Number>.

Now let’s look at the union method from Item 27
public static <E> Set<E> union(Set<E> s1, Set<E> s2)
Both parameters, s1 and s2, are E producers, so the PECS
public static <E> Set<E> union(Set<? extends E> s1,Set<? extends E> s2)
Do not use wildcard types as return types. Rather than providing additional flexibility for your users, it would force them to use wildcard types in client code.

Unfortunately, the type inference rules are quite complex.and they don’t always do what you want them to. Looking at the revised declaration for union, you
might think that you could do this:
Set<Integer> integers = ... ;
Set<Double> doubles = ... ;
Set<Number> numbers = union(integers, doubles); // you’ll get this error message: incompatible types
Luckily there is a way to deal with this sort of error. If the compiler doesn’t infer the type that you wish it had, you can tell it what type to use with an explicit type parameter. 
Set<Number> numbers = Union.<Number>union(integers, doubles);

Next let’s turn our attention to the max method from Item 27.
public static <T extends Comparable<T>> T max(List<T> list)
Here is a revised declaration that uses wildcard types:
public static <T extends Comparable<? super T>> T max(List<? extends T> list)
The tricky application is to the type parameter T. This is the first time we’ve seen a wildcard applied to a type parameter. T was originally specified to extend Comparable<T>, but a comparable of T consumes T instances (and produces integers indicating order relations). Therefore the parameterized type Comparable<T> is replaced by the bounded wildcard type Comparable<? super T>. Comparables are always consumers, so you should always use Comparable<? super T> in preference to Comparable<T>. The same is true of comparators, so you should always use Comparator<? super T> in preference to Comparator<T>.
example:
List<ScheduledFuture<?>> scheduledFutures = ... ; The reason that you can’t apply the original method declaration to this list is
that java.util.concurrent.ScheduledFuture does not implement Comparable<ScheduledFuture>. Instead, it is a subinterface of Delayed, which extends Comparable<Delayed>. In other words, a ScheduledFuture instance isn’t merely comparable to other ScheduledFuture instances; it’s comparable to any Delayed
instance, and that’s enough to cause the original declaration to reject it. 

There is one slight problem with the revised declaration for max:
// Won’t compile - wildcards can require change in method body!
public static <T extends Comparable<? super T>> T max(
List<? extends T> list) {
Iterator<T> i = list.iterator();
T result = i.next();
while (i.hasNext()) {
T t = i.next();
if (t.compareTo(result) > 0)
result = t;
}
return result;
}
That is the only change that we have to make to the body of the method - Iterator<? extends T> i = list.iterator(); 

There is one more wildcard-related topic that bears discussing. There is a duality between type parameters and wildcards, and many methods can be
declared using one or the other. 
For example, here are two possible declarations for a static method to swap two indexed items in a list.
// Two possible declarations for the swap method
public static <E> void swap(List<E> list, int i, int j);
public static void swap(List<?> list, int i, int j);
Which of these two declarations is preferable, and why? In a public API, the second is better because it’s simpler. You pass in a list—any list—and the method
swaps the indexed elements. There is no type parameter to worry about 

As a rule, if a type parameter appears only once in a method declaration, replace it with a wildcard. If it’s an unbounded type parameter, replace it with an unbounded wildcard; if it’s a bounded type parameter, replace it with a bounded wildcard.

There’s one problem with the second declaration for swap, which uses a wildcard in preference to a type parameter: the straightforward implementation won’t
compile:
public static void swap(List<?> list, int i, int j) {
list.set(i, list.set(j, list.get(i)));
}

The problem is that the type of list is List<?>, and you can’t put any value except null into a List<?>.
there is a way to implement - write a private helper method to capture the wildcard type. The helper method must be a generic method in order to capture the type.
public static void swap(List<?> list, int i, int j) {
swapHelper(list, i, j);
}
// Private helper method for wildcard capture
private static <E> void swapHelper(List<E> list, int i, int j) {
list.set(i, list.set(j, list.get(i)));
}

==> Item-29 - Consider typesafe heterogeneous containers
The most common use of generics is for collections, such as Set and Map, and single-element containers, such as ThreadLocal and AtomicReference. In all of
these uses, it is the container that is parameterized. This limits you to a fixed number of type parameters per container. Normally that is exactly what you want. A Set has a single type parameter, representing its element type; But Sometimes, however, you need more flexibility. For example, a database row
can have arbitrarily many columns, and it would be nice to be able to access all of them in a typesafe manner. Luckily, there is an easy way to achieve this  effect. The idea is to parameterize the key instead of the container. Then present the parameterized key to the container to insert or retrieve a value. The generic type system is used to guarantee that the type of the value agrees with its key.
As a simple example of this approach, consider a Favorites class that allows its clients to store and retrieve a “favorite” instance of arbitrarily many other
classes. The Class object will play the part of the parameterized key.
The reason this works is that class Class was generified in release 1.5. When a class literal is passed among methods to communicate both compile-time and
runtime type information, it is called a type token.
// Typesafe heterogeneous container pattern - implementation
public class Favorites {
private Map<Class<?>, Object> favorites = new HashMap<Class<?>, Object>();
public <T> void putFavorite(Class<T> type, T instance) {
if (type == null)
 throw new NullPointerException("Type is null");
 favorites.put(type, instance);
}
public <T> T getFavorite(Class<T> type) {
 return type.cast(favorites.get(type));
}
}
it’s not the type of the Map that’s a wildcard type but the type of its key. This means that every key can have a different parameterized type: one can be
Class<String>, the next Class<Integer>, and so on.
The implementation of the getFavorite method is trickier than that of put-Favorite. Its type is simply Object (the value type of the favorites
map) and we need to return a T. So, the getFavorite implementation dynamically casts the object reference to the type represented by the Class object,
using Class’s cast method.
// Achieving runtime type safety with a dynamic cast
public <T> void putFavorite(Class<T> type, T instance) {
favorites.put(type, type.cast(instance));
}
The second limitation of the Favorites class is that it cannot be used on a non-reifiable type (Item 25). In other words, you can store your favorite String or
String[], but not your favorite List<String>. List<String>.class is a syntax error.
Suppose you have an object of type Class<?> and you want to pass it to a method that requires a bounded type token.You could
cast the object to Class<? extends Annotation>, but this cast is unchecked, so it would generate a compile-time warning (Item 24). Luckily, class Class provides
an instance method that performs this sort of cast safely (and dynamically). The method is called asSubclass, and it casts the Class object on which it’s called to represent a subclass of the class represented by its argument. If the cast succeeds, the method returns its argument; if it fails, it throws a ClassCastException.

// Use of asSubclass to safely cast to a bounded type token
static Annotation getAnnotation(AnnotatedElement element, String annotationTypeName) {
Class<?> annotationType = null; // Unbounded type token
try {
 annotationType = Class.forName(annotationTypeName);
} catch (Exception ex) {
throw new IllegalArgumentException(ex);
}
return element.getAnnotation(
annotationType.asSubclass(Annotation.class));
}

==================================================================================================================
Enums and Annotations
Item-30 - Use enums instead of int constants
An enumerated type is a type whose legal values consist of a fixed set of constants, such as the seasons of the year, the planets in the solar system

Before enum types were added to the language, a common pattern for representing enumerated types was to declare a group of named int constants, one for each member of the type: 
// The int enum pattern - severely deficient!
public static final int APPLE_FUJI = 0;
The compiler won’t complain if you pass an apple to a method that expects an orange, compare apples to oranges with the == operator, or worse
Programs that use the int enum pattern are brittle. Because int enums are compile-time constants, they are compiled into the clients that use them. If the int
associated with an enum constant is changed, its clients must be recompiled. If they aren’t, they will still run, but their behavior will be undefined.
There is no easy way to translate int enum constants into printable strings.

Luckily, as of release 1.5, the language provides an alternative
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
The basic idea behind Java’s enum types is simple: they are classes that export one instance for each enumeration constant via a public static final field. Enum
types are effectively final, by virtue of having no accessible constructors. Because clients can neither create instances of an enum type nor extend it, there can be no instances but the declared enum constants. In other words, enum types are instance-controlled (page 6). They are a generalization of singletons (Item 3), which are essentially single-element enums.
Enums provide compile-time type safety. If you declare a parameter to be of type Apple, you are guaranteed that any non-null object reference passed to the
parameter is one of the three valid Apple values. Attempts to pass values of the wrong type will result in compile-time errors, as will attempts to assign an expression of one enum type to a variable of another, or to use the == operator to compare values of different enum types.
In addition to rectifying the deficiencies of int enums, enum types let you add arbitrary methods and fields and implement arbitrary interfaces. They provide
high-quality implementations of all the Object methods (Chapter 3), they implement Comparable (Item 12) and Serializable (Chapter 11)
So why would you want to add methods or fields to an enum type? For starters, you might want to associate data with its constants. Our Apple and Orange
types, for example, might benefit from a method that returns the color of the fruit, or one that returns an image of it.
// Enum type with data and behavior
public enum Planet {
MERCURY(3.302e+23, 2.439e6),
VENUS (4.869e+24, 6.052e6)
private final double mass; // In kilograms
private final double radius; // In meters
// Constructor
Planet(double mass, double radius) {
this.mass = mass;
this.radius = radius;
}
public double mass() { return mass; }
public double radius() { return radius; }
}

The techniques demonstrated in the Planet example are sufficient for most enum types, but sometimes you need more. There is different data associated with
each Planet constant, but sometimes you need to associate fundamentally different behavior with each constant. For example, suppose you are writing an enum
type to represent the operations on a basic four-function calculator.
public enum Operation {
PLUS, MINUS, TIMES, DIVIDE;
// Do the arithmetic op represented by this constant
double apply(double x, double y) {
switch(this) {
case PLUS: return x + y;
case MINUS: return x - y;
case TIMES: return x * y;
case DIVIDE: return x / y;
}
throw new AssertionError("Unknown op: " + this);
}
}
This code works, but is isn’t very pretty. It won’t compile without the throw statement because the end of the method is technically reachable, even though it
will never be reached. Luckily, there is a better way to associate a different behavior with each enum constant: declare an abstract apply method in the enum type, and override it with a concrete method for each constant in a constant-specific class body. Such methods are knows as constant-specific method implementations.
// Enum type with constant-specific method implementations
public enum Operation {
PLUS { double apply(double x, double y){return x + y;} },
MINUS { double apply(double x, double y){return x - y;} },
TIMES { double apply(double x, double y){return x * y;} },
DIVIDE { double apply(double x, double y){return x / y;} };
abstract double apply(double x, double y);
}
Constant-specific method implementations can be combined with constant specific data.
public enum Operation {
PLUS("+") {
double apply(double x, double y) { return x + y; }
},
MINUS("-") {
double apply(double x, double y) { return x - y; }
}
private final String symbol;
Operation(String symbol) { this.symbol = symbol; }
@Override public String toString() { return symbol; }
abstract double apply(double x, double y);
}
Enum constructors aren’t permitted to access the enum’s static fields, except for compile-time constant fields. This restriction is necessary
because these static fields have not yet been initialized when the constructors run.

A disadvantage of constant-specific method implementations is that they make it harder to share code among enum constants. For example, consider an
enum representing the days of the week in a payroll package.
// Enum that switches on its value to share code - questionable
enum PayrollDay {
MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY,SATURDAY, SUNDAY;
private static final int HOURS_PER_SHIFT = 8;
double pay(double hoursWorked, double payRate) {
double basePay = hoursWorked * payRate;
double overtimePay; // Calculate overtime pay
switch(this) {
case SATURDAY: case SUNDAY:
overtimePay = hoursWorked * payRate / 2;
default: // Weekdays
overtimePay = hoursWorked <= HOURS_PER_SHIFT ? 0 : (hoursWorked - HOURS_PER_SHIFT) * payRate / 2;
break;
}
return basePay + overtimePay;
}
}
This code is undeniably concise, but it is dangerous from a maintenance perspective. Suppose you add an element to the enum, perhaps a special value to represent a vacation day, but forget to add a corresponding case to the switch statement. The program will still compile, but the pay method will silently pay the worker the same amount for a vacation day as for an ordinary weekday. 
To perform the pay calculation safely with constant-specific method implementations, you would have to duplicate the overtime pay computation for each
constant, or move the computation into two helper methods (one for weekdays and one for weekend days) and invoke the appropriate helper method from each
constant. Either approach would result in a fair amount of boilerplate code, substantially reducing readability and increasing the opportunity for error.

The boilerplate could be reduced by replacing the abstract overtimePay method on PayrollDay with a concrete method that performs the overtime calculation
for weekdays. Then only the weekend days would have to override the method. But this would have the same disadvantage as the switch statement: if
you added another day without overriding the overtimePay method, you would silently inherit the weekday calculation.

What you really want is to be forced to choose an overtime pay strategy each time you add an enum constant. Luckily, there is a nice way to achieve this. The
idea is to move the overtime pay computation into a private nested enum, and to pass an instance of this strategy enum to the constructor for the PayrollDay enum. The PayrollDay enum then delegates the overtime pay calculation to the strategy enum, eliminating the need for a switch statement or constant-specific method implementation in PayrollDay. While this pattern is less concise than the switch statement, it is safer and more flexible:

// The strategy enum pattern
enum PayrollDay {
MONDAY(PayType.WEEKDAY), TUESDAY(PayType.WEEKDAY),
WEDNESDAY(PayType.WEEKDAY), THURSDAY(PayType.WEEKDAY),
FRIDAY(PayType.WEEKDAY),
SATURDAY(PayType.WEEKEND), SUNDAY(PayType.WEEKEND);
private final PayType payType;
PayrollDay(PayType payType) { this.payType = payType; }
double pay(double hoursWorked, double payRate) {
return payType.pay(hoursWorked, payRate);
}
// The strategy enum type
private enum PayType {
WEEKDAY {
double overtimePay(double hours, double payRate) {
return hours <= HOURS_PER_SHIFT ? 0 :
(hours - HOURS_PER_SHIFT) * payRate / 2;
}
},
WEEKEND {
double overtimePay(double hours, double payRate) {
return hours * payRate / 2;
}
};
private static final int HOURS_PER_SHIFT = 8;
abstract double overtimePay(double hrs, double payRate);
double pay(double hoursWorked, double payRate) {
double basePay = hoursWorked * payRate;
return basePay + overtimePay(hoursWorked, payRate);
}
}
}

If switch statements on enums are not a good choice for implementing constant-specific behavior on enums, what are they good for? Switches on enums
are good for augmenting external enum types with constant-specific behavior. For example, suppose the Operation enum is not under your control, and you
wish it had an instance method to return the inverse of each operation. simulate the effect with the following static method:
// Switch on an enum to simulate a missing method
public static Operation inverse(Operation op) {
switch(op) {
case PLUS: return Operation.MINUS;
case MINUS: return Operation.PLUS;
case TIMES: return Operation.DIVIDE;
case DIVIDE: return Operation.TIMES;
default: throw new AssertionError("Unknown op: " + op);
}
}

So when should you use enums? Anytime you need a fixed set of constants. Of course, this includes “natural enumerated types,” such as the planets, the days
of the week, and the chess pieces. But it also includes other sets for which you know all the possible values at compile time, such as choices on a menu, operation codes, and command line flags.

Item-31 - Use instance fields instead of ordinals
Many enums are naturally associated with a single int value. All enums have an ordinal method, which returns the numerical position of each enum constant in
its type. 
// Abuse of ordinal to derive an associated value - DON'T DO THIS
public enum Ensemble {
SOLO, DUET, TRIO, QUARTET, QUINTET,
SEXTET, SEPTET, OCTET, NONET, DECTET;
public int numberOfMusicians() { return ordinal() + 1; }
}
While this enum works, it is a maintenance nightmare. If the constants are reordered, the numberOfMusicians method will break.
Luckily, there is a simple solution to these problems. Never derive a value associated with an enum from its ordinal; store it in an instance field instead:
public enum Ensemble {
SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5)
private final int numberOfMusicians;
Ensemble(int size) { this.numberOfMusicians = size; }
public int numberOfMusicians() { return numberOfMusicians; }
}

Item-32 - Use EnumSet instead of bit fields
If the elements of an enumerated type are used primarily in sets, it is traditional to use the int enum pattern (Item 30), assigning a different power of 2 to each constant: 
// Bit field enumeration constants - OBSOLETE!
public class Text {
public static final int STYLE_BOLD = 1 << 0; // 1
public static final int STYLE_ITALIC = 1 << 1; // 2
public static final int STYLE_UNDERLINE = 1 << 2; // 4
public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8
// Parameter is bitwise OR of zero or more STYLE_ constants
public void applyStyles(int styles) { ... }
}

This representation lets you use the bitwise OR operation to combine several constants into a set, known as a bit field:
text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
The bit field representation also lets you perform set operations such as union and intersection efficiently using bitwise arithmetic. But bit fields have all the disadvantages of int enum constants and more.
The java.util package provides the EnumSet class to efficiently represent sets of values drawn from a single enum
type. This class implements the Set interface, providing all of the richness, type safety, and interoperability you get with any other Set implementation. But internally, each EnumSet is represented as a bit vector.
// EnumSet - a modern replacement for bit fields
public class Text {
public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }
// Any Set could be passed in, but EnumSet is clearly best
public void applyStyles(Set<Style> styles) { ... }
}
text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
In summary, just because an enumerated type will be used in sets, there is no reason to represent it with bit fields.

TIJ-EnumSets are built on top of longs, a long is 64 bits, and each enum instance requires one bit to indicate presence or absence. This means you can have an EnumSet for an enum of up to 64 elements without going beyond the use of a single long.


Item-33 - Use EnumMap instead of ordinal indexing
Occasionally you may see code that uses the ordinal method (Item 31) to index into an array. For example, consider this simplistic class meant to represent a culinary herb:
public class Herb {
public enum Type { ANNUAL, PERENNIAL, BIENNIAL }
private final String name;
private final Type type;
Herb(String name, Type type) {
this.name = name;
this.type = type;
}
@Override public String toString() {
return name;
}
}
Now suppose you have an array of herbs representing the plants in a garden, and you want to list these plants organized by type (annual, perennial, or biennial). To do this, you construct three sets, one for each type, and iterate through the garden, placing each herb in the appropriate set. Some programmers would do this by putting the sets into an array indexed by the type’s ordinal:
// Using ordinal() to index an array - DON'T DO THIS!
Herb[] garden = ... ;
Set<Herb>[] herbsByType = // Indexed by Herb.Type.ordinal()
(Set<Herb>[]) new Set[Herb.Type.values().length];
for (int i = 0; i < herbsByType.length; i++)
herbsByType[i] = new HashSet<Herb>();
for (Herb h : garden)
herbsByType[h.type.ordinal()].add(h);

This technique works, but it is fraught with problems. Because arrays are not compatible with generics (Item 25), the program requires an unchecked cast.
the most serious problem with this technique is that when you access an array that is indexed by an enum’s ordinal, it is your responsibility to use the correct int value;
there is a very fast Map implementation designed for use with enum keys, known as java.util.EnumMap. Here is how the program looks when it is rewritten to use EnumMap.
// Using an EnumMap to associate data with an enum
Map<Herb.Type, Set<Herb>> herbsByType =
new EnumMap<Herb.Type, Set<Herb>>(Herb.Type.class);
for (Herb.Type t : Herb.Type.values())
herbsByType.put(t, new HashSet<Herb>());
for (Herb h : garden)
herbsByType.get(h.type).add(h);
System.out.println(herbsByType);
This program is shorter, clearer, safer, and comparable in speed to the original version. There is no unsafe cast; no need to label the output manually, as the map keys are enums that know how to translate themselves to printable strings; and no possibility for error in computing array indices. The reason that EnumMap is comparable in speed to an ordinal-indexed array is that EnumMap uses such an array internally. But it hides this implementation detail from the programmer, combining the richness and type safety of a Map with the speed of an array.

You may see an array of arrays indexed (twice!) by ordinals used to represent a mapping from two enum values. For example, this program uses such an array to
map two phases to a phase transition (liquid to solid is freezing, liquid to gas is boiling, and so forth):
// Using ordinal() to index array of arrays - DON'T DO THIS!
public enum Phase { SOLID, LIQUID, GAS;
public enum Transition {
MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;
// Rows indexed by src-ordinal, cols by dst-ordinal
private static final Transition[][] TRANSITIONS = {
{ null, MELT, SUBLIME },
{ FREEZE, null, BOIL },
{ DEPOSIT, CONDENSE, null }
};
// Returns the phase transition from one phase to another
public static Transition from(Phase src, Phase dst) {
return TRANSITIONS[src.ordinal()][dst.ordinal()];
}
}
}
This program works and may even appear elegant, but appearances can be deceiving. Like the simpler herb garden example above, the compiler has no way
of knowing the relationship between ordinals and array indices.
Again, you can do much better with EnumMap. Because each phase transition is indexed by a pair of phase enums, you are best off representing the relationship
as a map from one enum (the source phase) to a map from the second enum (the destination phase) to the result (the phase transition).
// Using a nested EnumMap to associate data with enum pairs
public enum Phase {
SOLID, LIQUID, GAS;
public enum Transition {
MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);
final Phase src;
final Phase dst;
Transition(Phase src, Phase dst) {
this.src = src;
this.dst = dst;
}
// Initialize the phase transition map
private static final Map<Phase, Map<Phase,Transition>> m =
new EnumMap<Phase, Map<Phase,Transition>>(Phase.class);
static {
for (Phase p : Phase.values())
m.put(p,new EnumMap<Phase,Transition>(Phase.class));
for (Transition trans : Transition.values())
m.get(trans.src).put(trans.dst, trans);
}
public static Transition from(Phase src, Phase dst) {
return m.get(src).get(dst);
}
}
}
The code to initialize the phase transition map may look a bit complicated but it isn’t too bad. The type of the map is Map<Phase, Map<Phase, Transition>>,
which means “map from (source) phase to map from (destination) phase to transition.” The first loop in the static initializer block initializes the outer map to contain the three empty inner maps. The second loop in the block initializes the inner maps using the source and destination information provided by each state transition constant.
In summary, it is rarely appropriate to use ordinals to index arrays: use EnumMap instead. If the relationship that you are representing is multidimensional,
use EnumMap<..., EnumMap<...>>. This is a special case of the general principle that application programmers should rarely, if ever, use Enum.ordinal.

Item-34 - Emulate extensible enums with interfaces
using the language feature  it is not possible to have one enumerated type extend another; For the most part, extensibility of enums turns out to
be a bad idea. It is confusing that elements of an extension type are instances of the base type and not vice versa. There is no good way to enumerate over all of the elements of a base type and its extension.
That said, there is at least one compelling use case for extensible enumerated types, which is operation codes, also known as opcodes. An opcode is an enumerated type whose elements represent operations on some machine, such as the Operation type in Item 30, which represents the functions on a simple calculator. Sometimes it is desirable to let the users of an API provide their own operations, effectively extending the set of operations provided by the API.
Luckily, there is a nice way to achieve this effect using enum types
// Emulated extensible enum using an interface
public interface Operation {
double apply(double x, double y);
}
While the enum type (BasicOperation) is not extensible, the interface type (Operation) is, and it is the interface type that is used to represent operations in
APIs. You can define another enum type that implements this interface and use instances of this new type in place of the base type.
// Emulated extension enum
public enum ExtendedOperation implements Operation {
EXP("^") {
public double apply(double x, double y) {
return Math.pow(x, y);
}
},
REMAINDER("%") {
public double apply(double x, double y) {
return x % y;
}
};
private final String symbol;
ExtendedOperation(String symbol) {
this.symbol = symbol;
}
@Override public String toString() {
return symbol;
}
}

//here is a version of the test program

public static void main(String[] args) {
double x = Double.parseDouble(args[0]);
double y = Double.parseDouble(args[1]);
test(ExtendedOperation.class, x, y);
}
private static <T extends Enum<T> & Operation> void test(Class<T> opSet, double x, double y) { //The class literal serves as a bounded type token
for (Operation op : opSet.getEnumConstants())
System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
}

A second alternative is to use Collection<? extends Operation>, which is a bounded wildcard type
public static void main(String[] args) {
double x = Double.parseDouble(args[0]);
double y = Double.parseDouble(args[1]);
test(Arrays.asList(ExtendedOperation.values()), x, y);
}
private static void test(Collection<? extends Operation> opSet,double x, double y) {
for (Operation op : opSet)
System.out.printf("%f %s %f = %f%n",x, op, y, op.apply(x, y));
}
you are probably better off with  the bounded type token unless you need the flexibility to combine operations of multiple implementation types.
In summary, while you cannot write an extensible enum type, you can emulate it by writing an interface to go with a basic enum type that implements
the interface.

Item-35 - Prefer annotations to naming patterns
Prior to release 1.5, it was common to use naming patterns to indicate that some program elements demanded special treatment by a tool or framework. For example, the JUnit testing framework originally required its users to designate test methods by beginning their names with the characters test.
it has several big disadvantages. First, typographical errors may result in silent failures. For example, suppose you accidentally name a test method
tsetSafetyOverride instead of testSafetyOverride.
Annotations [JLS, 9.7] solve all of these problems nicely.
// Marker annotation type declaration
import java.lang.annotation.*;
/**
* Indicates that the annotated method is a test method.
* Use only on parameterless static methods.
*/
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
}

// Program containing marker annotations
public class Sample {
@Test public static void m1() { } // Test should pass
public static void m2() { }
@Test public static void m3() { // Test Should fail
throw new RuntimeException("Boom");
}
public static void m4() { }
@Test public void m5() { } // INVALID USE: nonstatic method
public static void m6() { }
@Test public static void m7() { // Test should fail
throw new RuntimeException("Crash");
}
public static void m8() { }
}

public class RunTests {
public static void main(String[] args) throws Exception {
int tests = 0;
int passed = 0;
Class testClass = Class.forName(args[0]);
for (Method m : testClass.getDeclaredMethods()) {
if (m.isAnnotationPresent(Test.class)) {
tests++;
try {
m.invoke(null);
passed++;
} catch (InvocationTargetException wrappedExc) {
Throwable exc = wrappedExc.getCause();
System.out.println(m + " failed: " + exc);
} catch (Exception exc) {
System.out.println("INVALID @Test: " + m);
}
}
}
System.out.printf("Passed: %d, Failed: %d%n",
passed, tests - passed);
}
}

// Annotation type with a parameter
import java.lang.annotation.*;
/**
* Indicates that the annotated method is a test method that
* must throw the designated exception to succeed.
*/
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
Class<? extends Exception> value();
}

Item-36 - Consistently use the Override annotation
In summary, the compiler can protect you from a great many errors if you use
the Override annotation on every method declaration that you believe to override
a supertype declaration.

Item-37 - Use marker interfaces to define types
A marker interface is an interface that contains no method declarations, but merely designates (or “marks”) a class that implements the interface.
Inexplicably, the authors of the ObjectOutputStream API did not take advantage of the Serializable interface in declaring the write method. The
method’s argument type should have been Serializable rather than Object. 

You may hear it said that marker annotations (Item 35) make marker interfaces obsolete. This assertion is incorrect. Marker interfaces have two advantages
over marker annotations. First and foremost, marker interfaces define a type that is implemented by instances of the marked class; marker annotations do
not. The existence of this type allows you to catch errors at compile time that you couldn’t catch until runtime if you used a marker annotation.
													Another advantage of marker interfaces over marker annotations is that they
can be targeted more precisely. If an annotation type is declared with target ElementType. TYPE, it can be applied to any class or interface. Suppose you have a
marker that is applicable only to implementations of a particular interface. If you define it as a marker interface, you can have it extend the sole interface to which it is applicable, guaranteeing that all marked types are also subtypes of the sole interface to which it is applicable.
Arguably, the Set interface is just such a restricted marker interface. It is applicable only to Collection subtypes, but it adds no methods beyond those
defined by Collection. It is not generally considered to be a marker interface because it refines the contracts of several Collection methods, including add,
equals, and hashCode.

The chief advantage of marker annotations over marker interfaces is that it is possible to add more information to an annotation type after it is already in use, by adding one or more annotation type elements with defaults [JLS, 9.6]. What starts life as a mere marker annotation type can evolve into a richer annotation type over time. Such evolution is not possible with marker interfaces.
Another advantage of marker annotations is that they are part of the larger annotation facility. Therefore, marker annotations allow for consistency in frameworks that permit annotation of a variety of program elements.

So when should you use a marker annotation and when should you use a marker interface? Clearly you must use an annotation if the marker applies to any
program element other than a class or interface, as only classes and interfaces can be made to implement or extend an interface. If the marker applies only to classes and interfaces, ask yourself the question, Might I want to write one or more methods that accept only objects that have this marking? If so, you should use a marker interface in preference to an annotation. This will make it possible for you to use the interface as a parameter type for the methods in question, which will result in the very real benefit of compile-time type checking.
If you answered no to the first question, ask yourself one more: Do I want to limit the use of this marker to elements of a particular interface, forever? If so, it makes sense to define the marker as a subinterface of that interface. If you answered no to both questions, you should probably use a marker annotation.

If you find yourself writing a marker annotation type whose target is ElementType.TYPE, take the time to figure out whether it really should be an
annotation type, or whether a marker interface would be more appropriate. In a sense, this item is the inverse of Item 19, which says, “If you don’t want
to define a type, don’t use an interface.” To a first approximation, this item says, “If you do want to define a type, do use an interface.”

==================================================================================================================

================================Methods==================================================================================
Item-38  - Check parameters for validity
If an invalid parameter value is passed to a method and the method checks its parameters before execution, it will fail quickly and cleanly with an appropriate
exception. If the method fails to check its parameters, several things could happen. The method could fail with a confusing exception in the midst of processing. Worse, the method could return normally but silently compute the wrong result.

Item-39 - Make defensive copies when needed
Even in a safe language, you aren’t insulated from other classes without some effort on your part. You must program defensively, with the assumption that
clients of your class will do their best to destroy its invariants. This may actually be true if someone tries to break the security of your system, but more likely your class will have to cope with unexpected behavior resulting from honest mistakes on the part of programmers using your API.

To protect the internals of a Period instance from this sort of attack, it is essential to make a defensive copy of each mutable parameter to the constructor
and to use the copies as components of the Period instance in place of the originals:
// Repaired constructor - makes defensive copies of parameters
public Period(Date start, Date end) {
this.start = new Date(start.getTime());
this.end = new Date(end.getTime());
if (this.start.compareTo(this.end) > 0)
throw new IllegalArgumentException(start +" after "+ end);
}
Note that defensive copies are made before checking the validity of the parameters (Item 38), and the validity check is performed on
the copies rather than on the originals. While this may seem unnatural, it is necessary. It protects the class against changes to the parameters from another thread during the “window of vulnerability” between the time the parameters are checked and the time they are copied. (In the computer security community, this is known as a time-of-check/time-of-use or TOCTOU attack.

Note also that we did not use Date’s clone method to make the defensive copies.Because Date is nonfinal, the clone method is not guaranteed to return an
object whose class is java.util.Date: it could return an instance of an untrusted subclass specifically designed for malicious mischief. Such a subclass could, for example, record a reference to each instance in a private static list at the time of its creation and allow the attacker to access this list. This would give the attacker free reign over all instances. To prevent this sort of attack, do not use the clone method to make a defensive copy of a parameter whose type is subclassable by untrusted parties.

To defend against the second attack, merely modify the accessors to return
defensive copies of mutable internal fields:
// Repaired accessors - make defensive copies of internal fields
public Date start() {
return new Date(start.getTime());
}
public Date end() {
return new Date(end.getTime());
}
In the accessors, unlike the constructor, it would be permissible to use the clone method to make the defensive copies. This is so because we know that the
class of Period’s internal Date objects is java.util.Date, and not some potentially untrusted subclass.

Defensive copying of parameters is not just for immutable classes. Anytime you write a method or constructor that enters a client-provided object into an
internal data structure, think about whether the client-provided object is potentially mutable. If it is, think about whether your class could tolerate a change in the object after it was entered into the data structure. If the answer is no, you must defensively copy the object and enter the copy into the data structure in place of the original.

Item-40 - Design method signatures carefully
Choose method names carefully.When in doubt, look to the Java library APIs for guidance.
Don’t go overboard in providing convenience methods.  Every method should “pull its weight.” Too many methods make a class difficult to learn, use.
Avoid long parameter lists. Aim for four parameters or fewer. There are three techniques for shortening overly long parameter lists. 
One is to break the method up into multiple methods, each of which requires only a subset of the parameters. If done carelessly, this can lead to too many methods, but it can also help reduce the method count by increasing orthogonality.
A second technique for shortening long parameter lists is to create helper classes to hold groups of parameters. Typically these helper classes are static
member classes (Item 22). This technique is recommended if a frequently occurring sequence of parameters is seen to represent some distinct entity. For example,
suppose you are writing a class representing a card game, and you find yourself constantly passing a sequence of two parameters representing a card’s rank and its suit.
A third technique that combines aspects of the first two is to adapt the Builder pattern (Item 2) from object construction to method invocation. If you have a
method with many parameters, especially if some of them are optional.
For parameter types, favor interfaces over classes
Prefer two-element enum types to boolean parameters. Also, it makes it easy to add more options later. For example, you might
have a Thermometer type with a static factory that takes a value of this enum:
public enum TemperatureScale { FAHRENHEIT, CELSIUS }
Not only does Thermometer.newInstance(TemperatureScale.CELSIUS) make a lot more sense than Thermometer.newInstance(true), but you can add
KELVIN to TemperatureScale in a future release without having to add a new static factory to Thermometer.

Item-41 - Use overloading judiciously
the choice of which overloading to invoke is made at compile time. selection among overloaded methods is static, while selection among overridden methods is
dynamic. The correct version of an overridden method is chosen at runtime, based on the runtime type of the object on which the method is invoked.
It is bad practice to write code whose behavior is likely to confuse programmers. This is especially true for APIs. If the typical user of an API does
not know which of several method overloadings will get invoked for a given set of parameters, use of the API is likely to result in errors.

A safe, conservative policy is never to export two overloadings with the same number of parameters.
Exporting multiple overloadings with the same number of parameters is unlikely to confuse programmers if it is always clear which overloading will apply
to any given set of actual parameters. This is the case when at least one corresponding formal parameter in each pair of overloadings has a “radically different” type in the two overloadings.

Prior to release 1.5, all primitive types were radically different from all reference types, but this is no longer true in the presence of autoboxing, and it has caused real trouble. Consider the following program: 
public class SetList {
public static void main(String[] args) {
Set<Integer> set = new TreeSet<Integer>();
List<Integer> list = new ArrayList<Integer>();
for (int i = -3; i < 3; i++) {
set.add(i);
list.add(i);
}
for (int i = 0; i < 3; i++) {
set.remove(i);
list.remove(i);
}
System.out.println(set + " " + list);
}
} // prints [-3, -2, -1] [-2, 0, 2] instead of [-3, -2, -1] [-3, -2, -1].
Here’s what’s happening: The call to set.remove(i) selects the overloading remove(E), where E is the element type of the set (Integer), and autoboxes i
from int to Integer. This is the behavior you’d expect, so the program ends up removing the positive values from the set. The call to list.remove(i), on the
other hand, selects the overloading remove(int i), which removes the element at the specified position from a list.
To fix the problem, cast list.remove’s argument to Integer, forcing the correct overloading to be selected. Alternatively, you could invoke Integer.valueOf on i and pass the result to list.remove. list.remove((Integer) i); // or remove(Integer.valueOf(i))

The confusing behavior demonstrated by the previous example came about because the List<E> interface has two overloadings of the remove method:
remove(E) and remove(int). Prior to release 1.5 when it was “generified,” the List interface had a remove(Object) method in place of remove(E), and the corresponding parameter types, Object and int, were radically different. But in the presence of generics and autoboxing, the two parameter types are no longer radically different. In other words, adding generics and autoboxing to the language damaged the List interface.

For example, the String class exports two overloaded static factory methods, valueOf(char[]) and valueOf(Object), that do completely different things when passed the same object reference. There is no real justification for this, and it should be regarded as an anomaly with the potential for real confusion.

Item-42 - Use varargs judiciously
In release 1.5, varargs methods, formally known as variable arity methods. Varargs methods accept zero or more arguments of a specified type.
// Simple use of varargs
static int sum(int... args) {
int sum = 0;
for (int arg : args)
sum += arg;
return sum;
}

You can retrofit an existing method that takes an array as its final parameter to take a varargs parameter instead with no effect on existing clients. But just
because you can doesn’t mean that you should!.
Consider the case of Arrays.asList. This method was never designed to gather multiple arguments into a list, but it seemed like a good idea to retrofit it to do so when varargs were added to the platform.
List<String> homophones = Arrays.asList("to", "too", "two");
This usage works, but it was a big mistake to enable it. Prior to release 1.5, this was a common idiom to print the contents of an array.
// Obsolete idiom to print an array!
System.out.println(Arrays.asList(myArray));
The idiom worked only on arrays of  object reference types, but if you accidentally tried it on an array of primitives, the program wouldn’t compile.
public static void main(String[] args) {
int[] digits = { 3, 1, 4, 1, 5, 9, 2, 6, 5, 4 };
System.out.println(Arrays.asList(digits));
}
would generate this error message in release 1.4:
Because of the unfortunate decision to retrofit Arrays.asList as a varargs method in release 1.5, this program now compiles without error or warning.

Instead of retrofitting Arrays.asList, it would have been better to add a new method to Collections specifically for the purpose of gathering its arguments
into a list: 
public static <T> List<T> gather(T... args) {
return Arrays.asList(args);
}

// The right way to print an array
System.out.println(Arrays.toString(myArray));

The lesson is clear. Don’t retrofit every method that has a final array
parameter; use varargs only when a call really operates on a variable-length
sequence of values.

Exercise care when using the varargs facility in performance-critical situations. Every invocation of a varargs method causes an array allocation and initialization. If you have determined empirically that you can’t afford this cost but you need the flexibility of varargs, there is a pattern that lets you have your cake and eat it too. Suppose you’ve determined that 95 percent of the calls to a method have three or fewer parameters. Then declare five overloadings of the method, one each with zero through three ordinary parameters, and a single varargs method for use when the number of arguments exceeds three.


Item-43 - Return empty arrays or collections, not nulls
It is not uncommon to see methods that look something like this:
private final List<Cheese> cheesesInStock = ...;
public Cheese[] getCheeses() {
if (cheesesInStock.size() == 0)
return null;
}
if (cheeses != null && Arrays.asList(cheeses).contains(Cheese.STILTON)) //special case for the situation where no cheeses

It is sometimes argued that a null return value is preferable to an empty array because it avoids the expense of allocating the array. This argument fails on two counts. First, it is inadvisable to worry about performance at this level unless profiling has shown that the method in question is a real contributor to performance problems (Item 55).
Second, it is possible to return the same zero-length array from every invocation that returns no items because zero-length arrays are immutable
and immutable objects may be shared freely (Item 15). In fact, this is exactly what happens when you use the standard idiom for dumping items from a collection
into a typed array: 

// The right way to return an array from a collection
private final List<Cheese> cheesesInStock = ...;
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];
/**
* @return an array containing all of the cheeses in the shop.
*/
public Cheese[] getCheeses() {
return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
In this idiom, an empty-array constant is passed to the toArray method to indicate the desired return type. Normally the toArray method allocates the
returned array, but if the collection is empty, it fits in the zero-length input array, and the specification for Collection.toArray(T[]) guarantees that the input array will be returned if it is large enough to hold the collection. Therefore the idiom never allocates an empty array.

In similar fashion, a collection-valued method can be made to return the same immutable empty collection every time it needs to return an empty collection.
// The right way to return a copy of a collection
public List<Cheese> getCheeseList() {
if (cheesesInStock.isEmpty())
return Collections.emptyList(); // Always returns same list
else
return new ArrayList<Cheese>(cheesesInStock);
}

Item-44 Write doc comments for all exposed API elements
If an API is to be usable, it must be documented. Traditionally API documentation as generated manually, and keeping it in sync with code was a chore. The Java
programming environment eases this task with the Javadoc utility. Javadoc generates API documentation automatically from source code with specially
formatted documentation comments, more commonly known as doc comments. 
To document your API properly, you must precede every exported class, interface, constructor, method, and field declaration with a doc comment.
/**
* Returns the element at the specified position in this list.
*
* <p>This method is <i>not</i> guaranteed to run in constant
* time. In some implementations it may run in time proportional
* to the element position.
*
* @param index index of element to return; must be
* non-negative and less than the size of this list
* @return the element at the specified position in this list
* @throws IndexOutOfBoundsException if the index is out of range
* ({@code index < 0 || index >= this.size()})
*/
E get(int index);

==================================================================================================================

===================================General Programming===============================================================================
Item-45 - Minimize the scope of local variables
This item is similar in nature to Item 13, “Minimize the accessibility of classes and members.” By minimizing the scope of local variables, you increase the readability and maintainability of your code and reduce the likelihood of error.
The most powerful technique for minimizing the scope of a local variable is to declare it where it is first used.
Nearly every local variable declaration should contain an initializer. If you don’t yet have enough information to initialize a variable sensibly, you should
postpone the declaration until you do. One exception to this rule concerns trycatch statements. If a variable is initialized by a method that throws a checked
exception, it must be initialized inside a try block. If the value must be used outside of the try block, then it must be declared before the try block.

Loops present a special opportunity to minimize the scope of variables. The or loop, in both its traditional and for-each forms, allows you to declare loop
variables, limiting their scope to the exact region where they’re needed.
// Preferred idiom for iterating over a collection
for (Element e : c) {
doSomething(e);
}

To see why these for loops are preferable to a while loop, consider the following
code fragment, which contains two while loops and one bug:
Iterator<Element> i = c.iterator();
while (i.hasNext()) {
doSomething(i.next());
}
...
Iterator<Element> i2 = c2.iterator();
while (i.hasNext()) { // BUG!
doSomethingElse(i2.next());
}
The second loop contains a cut-and-paste error: it initializes a new loop variable, i2, but uses the old one, i, which is, unfortunately, still in scope.
If a similar cut-and-paste error were made in conjunction with either of the for loops (for-each or traditional), the resulting code wouldn’t even compile.

A final technique to minimize the scope of local variables is to keep methods small and focused. If you combine two activities in the same method, local variables relevant to one activity may be in the scope of the code performing the other activity.

Item-46 - Prefer for-each loops to traditional for loops
Prior to release 1.5, this was the preferred idiom for iterating over a collection:
for (Iterator i = c.iterator(); i.hasNext(); ) {
doSomething((Element) i.next()); // (No generics before 1.5)
}
These idioms are better than while loops (Item 45), but they aren’t perfect. The iterator and the index variables are both just clutter.
In fact, it may offer a slight performance advantage over an ordinary for loop in some circumstances, as it computes the limit of the array index only once.

Here is a common mistake that people make when they try to do nested iteration over two collections:
// Can you spot the bug?
enum Suit { CLUB, DIAMOND, HEART, SPADE }
enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT,
NINE, TEN, JACK, QUEEN, KING }
...
Collection<Suit> suits = Arrays.asList(Suit.values());
Collection<Rank> ranks = Arrays.asList(Rank.values());
List<Card> deck = new ArrayList<Card>();
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); )
for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
deck.add(new Card(i.next(), j.next()));

The problem is that the next method is called too many times on the iterator for the outer collection (suits). It should be called from the outer loop, so that it is called once per suit, but instead it is called from the inner loop, so it is called once per card.

// Fixed, but ugly - you can do better!
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); ) {
Suit suit = i.next();
for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
deck.add(new Card(suit, j.next()));
}

If instead you use a nested for-each loop, the problem simply disappears. The resulting code is as succinct as you could wish for:
// Preferred idiom for nested iteration on collections and arrays
for (Suit suit : suits)
for (Rank rank : ranks)
deck.add(new Card(suit, rank));

Unfortunately, there are three common situations where you can’t use a for-each loop:
Filtering, Transforming and Parallel iteration

Item-47 - Know and use the libraries
By using a standard library, you take advantage of the knowledge of the experts who wrote it and the experience of those who used it before you.
A second advantage of using the libraries is that you don’t have to waste your time writing ad hoc solutions to problems that are only marginally related to your work.
A third advantage of using standard libraries is that their performance tends to improve over time, with no effort on your part. Because many people use them
and because they’re used in industry-standard benchmarks, the organizations that supply these libraries have a strong incentive to make them run faster.
To summarize, don’t reinvent the wheel

Item-48 - Avoid float and double if exact answers are required
The float and double types are designed primarily for scientific and engineering calculations. They perform binary floating-point arithmetic, which was carefully designed to furnish accurate approximations quickly over a broad range of magnitudes.
The float and double types are particularly illsuited for monetary calculations because it is impossible to represent 0.1 (or any other negative power of ten) as a float or double exactly.
For example, suppose you have $1.03 in your pocket, and you spend 42¢. How much money do you have left
System.out.println(1.03 - .42); Unfortunately, it prints out 0.6100000000000001. This is not an isolated case

// Broken - uses floating point for monetary calculation!
public static void main(String[] args) {
double funds = 1.00;
int itemsBought = 0;
for (double price = .10; funds >= price; price += .10) {
funds -= price;
itemsBought++;
}
System.out.println(itemsBought + " items bought.");
System.out.println("Change: $" + funds);
}

If you run the program, you’ll find that you can afford three pieces of candy, and you have $0.3999999999999999 left. This is the wrong answer! The right way to
solve this problem is to use BigDecimal, int, or long for monetary calculations.

Here’s a straightforward transformation of the previous program to use the BigDecimal type in place of double:
public static void main(String[] args) {
final BigDecimal TEN_CENTS = new BigDecimal( ".10");
int itemsBought = 0;
BigDecimal funds = new BigDecimal("1.00");
for (BigDecimal price = TEN_CENTS;
funds.compareTo(price) >= 0;
price = price.add(TEN_CENTS)) {
itemsBought++;
funds = funds.subtract(price);
}
System.out.println(itemsBought + " items bought.");
System.out.println("Money left over: $" + funds);
}

There are, however, two disadvantages to using BigDecimal: it’s less convenient than using a primitive arithmetic type, and it’s slower.

An alternative to using BigDecimal is to use int or long, depending on the amounts involved, and to keep track of the decimal point yourself.
public static void main(String[] args) {
int itemsBought = 0;
int funds = 100;
for (int price = 10; funds >= price; price += 10) {
itemsBought++;
funds -= price;
}
System.out.println(itemsBought + " items bought.");
System.out.println("Money left over: "+ funds + " cents");
}

If the quantities don’t exceed nine decimal digits, you can use int; if they don’t exceed eighteen digits, you can use long. If the quantities might exceed eighteen digits, you must use BigDecimal.

Item-49 - Prefer primitive types to boxed primitives
public class Unbelievable {
static Integer i;
public static void main(String[] args) {
if (i == 42)
System.out.println("Unbelievable");
}
}
when you mix primitives and boxed primitives in a single operation, the boxed primitive is auto-unboxed, and this case is no exception. If a null object reference is auto-unboxed, you get a NullPointerException.

// Hideously slow program! Can you spot the object creation?
public static void main(String[] args) {
Long sum = 0L;
for (long i = 0; i < Integer.MAX_VALUE; i++) {
sum += i;
}
System.out.println(sum);
}
This program is much slower than it should be because it accidentally declares a local variable (sum) to be of the boxed primitive type Long instead of the primitive type long. The program compiles without error or warning, and the variable is repeatedly boxed and unboxed, causing the observed performance degradation.


Item-50 - Avoid strings where other types are more appropriate
Strings are poor substitutes for enum types.
Strings are poor substitutes for aggregate types. // Inappropriate use of string as aggregate type
String compoundKey = className + "#" + i.next();
If the character used to separate fields occurs in one of the fields, chaos may result.

Strings are poor substitutes for capabilities.Occasionally, strings are used to grant access to some functionality. For example, consider the design of a
thread-local variable facility. Such a facility provides variables for which each thread has its own value. The Java libraries have had a thread-local variable facility since release 1.2, but prior to that, programmers had to roll their own.
// Broken - inappropriate use of string as capability!
public class ThreadLocal {
private ThreadLocal() { } // Noninstantiable
// Sets the current thread's value for the named variable.
public static void set(String key, Object value);
// Returns the current thread's value for the named variable.
public static Object get(String key);
}
The problem with this approach is that the string keys represent a shared global namespace for thread-local variables. In order for the approach to work, the
client-provided string keys have to be unique: if two clients independently decide to use the same name for their thread-local variable, they unintentionally share a single variable, which will generally cause both clients to fail.

This API can be fixed by replacing the string with an unforgeable key (sometimes called a capability):
public class ThreadLocal {
private ThreadLocal() { } // Noninstantiable
public static class Key { // (Capability)
Key() { }
}
// Generates a unique, unforgeable key
public static Key getKey() {
return new Key();
}
public static void set(Key key, Object value);
public static Object get(Key key);
}

public final class ThreadLocal {
public ThreadLocal() { }
public void set(Object value);
public Object get();
}

This API isn’t typesafe, because you have to cast the value from Object to its actual type when you retrieve it from a thread-local variable.
public final class ThreadLocal<T> {
public ThreadLocal() { }
public void set(T value);
public T get();
}
This is, roughly speaking, the API that java.lang.ThreadLocal provides

Item-51 - Beware the performance of string concatenation
Using the string
concatenation operator repeatedly to concatenate n strings requires time quadratic in n. It is an unfortunate consequence of the fact that strings are immutable
(Item 15). When two strings are concatenated, the contents of both are copied.
// Inappropriate use of string concatenation - Performs horribly!
public String statement() {
String result = "";
for (int i = 0; i < numItems(); i++)
result += lineForItem(i); // String concatenation
return result;
}

public String statement() {
StringBuilder b = new StringBuilder(numItems() * LINE_WIDTH);
for (int i = 0; i < numItems(); i++)
b.append(lineForItem(i));
return b.toString();
}
The difference in performance is dramatic. If numItems returns 100 and lineForItem returns a constant 80-character string, the second method is eightyfive
times faster than the first on my machine. Because the first method is quadratic in the number of items and the second is linear, the performance
difference is even more dramatic for larger numbers of items. Note that the second method preallocates a StringBuilder large enough to hold the result. Even if it is detuned to use a default-sized StringBuilder, it is still fifty times faster.

Item-52 - Refer to objects by their interfaces
// Bad - uses class as type!
Vector<Subscriber> subscribers = new Vector<Subscriber>();
If you get into the habit of using interfaces as types, your program will be much more flexible.
List<Subscriber> subscribers = new ArrayList<Subscriber>();

It is entirely appropriate to refer to an object by a class rather than an interface if no appropriate interface exists. For example, consider value
classes, such as String and BigInteger. Value classes are rarely written with multiple implementations in mind. They are often final and rarely have corresponding interfaces.


Item-53 - Prefer interfaces to reflection
The core reflection facility, java.lang.reflect, offers programmatic access to information about loaded classes. Given a Class object, you can obtain Constructor, Method, and Field instances representing the constructors, methods, and fields of the class represented by the Class instance.

This power, however, comes at a price:
You lose all the benefits of compile-time type checking, including exception checking. If a program attempts to invoke a nonexistent or inaccessible method
reflectively, it will fail at runtime unless you’ve taken special precautions
The code required to perform reflective access is clumsy and verbose
Performance suffers. Reflective method invocation is much slower than normal method invocation.

The core reflection facility was originally designed for component-based -application builder tools. There are a few sophisticated applications that require reflection. Examples include class browsers, object inspectors, code analysis tools, and interpretive embedded systems. Reflection is also appropriate for use in remote procedure call (RPC) systems to eliminate the need for stub compilers.

You can obtain many of the benefits of reflection while incurring few of its costs by using it only in a very limited form. For many programs that must
use a class that is unavailable at compile time, there exists at compile time an appropriate interface or superclass by which to refer to the class (Item 52). If this is the case, you can create instances reflectively and access them normally via their interface or superclass.
// Reflective instantiation with interface access
public static void main(String[] args) {
// Translate the class name into a Class object
Class<?> cl = null;
try {
cl = Class.forName(args[0]);
} catch(ClassNotFoundException e) {
System.err.println("Class not found.");
System.exit(1);
}
// Instantiate the class
Set<String> s = null;
try {
s = (Set<String>) cl.newInstance();
} catch(IllegalAccessException e) {
System.err.println("Class not accessible.");
System.exit(1);
} catch(InstantiationException e) {
System.err.println("Class not instantiable.");
System.exit(1);
}
// Exercise the set
s.addAll(Arrays.asList(args).subList(1, args.length));
System.out.println(s);
}

Item-54 - Use native methods judiciously
The Java Native Interface (JNI) allows Java applications to call native methods, which are special methods written in native programming languages such as C or
C++. Native methods can perform arbitrary computation in native languages before returning to the Java programming language.

Historically, native methods have had three main uses. They provided access to platform-specific facilities such as registries and file locks. They provided
access to libraries of legacy code, which could in turn provide access to legacy data. Finally, native methods were used to write performance-critical parts of
applications in native languages for improved performance.

It is legitimate to use native methods to access platform-specific facilities, but as the Java platform matures, it provides more and more features previously found only in host platforms. For example, java.util.prefs, added in release 1.4, offers the functionality of a registry.

It is rarely advisable to use native methods for improved performance. In early releases (prior to 1.3), it was often necessary, but JVM implementations have
gotten much faster.
The use of native methods has serious disadvantages. Because native languages are not safe (Item 39), applications using native methods are no longer
immune to memory corruption errors. Because native languages are platform dependent, applications using native methods are far less portable.

Item-55 - Optimize judiciously
Don’t sacrifice sound architectural principles for performance. Strive to write good programs rather than fast ones. If a good program is not fast enough, its
architecture will allow it to be optimized.

Consider the performance consequences of your API design decisions. Making a public type mutable may require a lot of needless defensive copying
(Item 39). Similarly, using inheritance in a public class where composition would have been appropriate ties the class forever to its superclass, which can place artificial limits on the performance of the subclass.

The effects of API design on performance are very real. Consider the getSize method in the java.awt.Component class. The decision that this performancecritical
method was to return a Dimension instance, coupled with the decision that Dimension instances are mutable, forces any implementation of this method to 
allocate a new Dimension instance on every invocation.

Profiling tools can help you decide where to focus your optimization efforts. Such tools give you runtime information, such as roughly how much time each
method is consuming and how many times it is invoked. In addition to focusing your tuning efforts, this can alert you to the need for algorithmic changes.

Item-56 - Adhere to generally accepted naming conventions
Loosely speaking, naming conventions fall into two categories: typographical and grammatical.
There are only a handful of typographical naming conventions, covering packages, classes, interfaces, methods, fields, and type variables. You should
rarely violate them and never without a very good reason. If an API violates these conventions, it may be difficult to use. If an implementation violates them, it may be difficult to maintain.

Package names should be hierarchical with the components separated by periods. Components should consist of lowercase alphabetic characters and, rarely,
digits. for example, edu.cmu, com.sun, gov.nsa. Meaningful abbreviations are encouraged, for example, util rather than utilities. Acronyms are acceptable, for example, awt. 

Class and interface names, including enum and annotation type names, should consist of one or more words, with the first letter of each word capitalized.
Which class name would you rather see, HTTPURL or HttpUrl?

Grammatical naming conventions are more flexible and more controversial than typographical conventions. There are no grammatical naming conventions to
speak of for packages. Classes, including enum types, are generally named with a singular noun or noun phrase, for example, Timer, BufferedWriter.

Methods that perform some action are generally named with a verb or verb phrase (including object), for example, append or drawImage. Methods that return
a boolean value usually have names that begin with the word is or, less commonly, has, followed by a noun, noun phrase, or any word or phrase that functions
as an adjective, for example, isDigit, isProbablePrime, isEmpty, isEnabled, or hasSiblings.

A few method names deserve special mention. Methods that convert the type of an object, returning an independent object of a different type, are often called
toType, for example, toString, toArray. Methods that return a view (Item 5) whose type differs from that of the receiving object are often called asType, for
example, asList. Methods that return a primitive with the same value as the object on which they’re invoked are often called typeValue, for example, 
intValue. Common names for static factories are valueOf, of, getInstance, newInstance, getType, and newType.
==================================================================================================================

======================================Exceptions============================================================================
Item-57 - Use exceptions only for exceptional conditions
// Horrible abuse of exceptions. Don't ever do this!
try {
int i = 0;
while(true)
range[i++].climb();
} catch(ArrayIndexOutOfBoundsException e) {
}
The infinite loop terminates by throwing, catching, and ignoring an ArrayIndexOutOfBoundsException when it attempts to access the first array element outside the bounds of the array.
for (Mountain m : range)
m.climb();

exceptions are, as their name implies, to be used only for exceptional conditions; they should never be used for ordinary control flow.

A well-designed API must not force its clients to use exceptions for ordinary control flow. A class with a “state-dependent” method that can be invoked only under certain unpredictable conditions should generally have a separate “state-testing” method indicating whether it is appropriate to invoke the state-dependent method. For example, the Iterator interface has the state-dependent method next and the corresponding state-testing method hasNext. This enables the standard idiom for iterating over a collection with a traditional for loop (as well as the for-each loop, where the has-Next method is used internally):
for (Iterator<Foo> i = collection.iterator(); i.hasNext(); ) {
Foo foo = i.next();
...
}
If Iterator lacked the hasNext method, clients would be forced to do this instead:
// Do not use this hideous code for iteration over a collection!
try {
Iterator<Foo> i = collection.iterator();
while(true) {
Foo foo = i.next();
...
}
} catch (NoSuchElementException e) {
}

An alternative to providing a separate state-testing method is to have the statedependent method return a distinguished value such as null if it is invoked with
the object in an inappropriate state. This technique would not be appropriate for Iterator, as null is a legitimate return value for the next method.

Here are some guidelines to help you choose between a state-testing method and a distinguished return value. If an object is to be accessed concurrently without
external synchronization or is subject to externally induced state transitions, you must use a distinguished return value, as the object’s state could change in the interval between the invocation of a state-testing method and its state-dependent method. Performance concerns may dictate that a distinguished return value be used if a separate state-testing method would duplicate the work of the statedependent method. All other things being equal, a state-testing method is mildly preferable to a distinguished return value. It offers slightly better readability, and incorrect use may be easier to detect: if you forget to call a state-testing method, the state-dependent method will throw an exception, making the bug obvious; if you forget to check for a distinguished return value, the bug may be subtle.

Item-58 - Use checked exceptions for recoverable conditions and runtime exceptions for programming errors
The Java programming language provides three kinds of throwables: checked exceptions, runtime exceptions, and errors.

use checked exceptions for conditions from which the caller can reasonably be expected to recover. By throwing a checked exception, you
force the caller to handle the exception in a catch clause or to propagate it outward. Each checked exception that a method is declared to throw is therefore a
potent indication to the API user that the associated condition is a possible outcome of invoking the method.

If a program throws an unchecked exception or an error, it is generally the case that recovery is impossible and continued execution would
do more harm than good. If a program does not catch such a throwable, it will cause the current thread to halt with an appropriate error message.

Use runtime exceptions to indicate programming errors. The great majority of runtime exceptions indicate precondition violations. A precondition violation
is simply a failure by the client of an API to adhere to the contract established by the API specification. For example, the contract for array access specifies that the array index must be between zero and the array length minus one. ArrayIndexOutOfBoundsException indicates that this precondition was violated.

While the Java Language Specification does not require it, there is a strong convention that errors are reserved for use by the JVM to indicate resource deficiencies, invariant failures, or other conditions that make it impossible to continue execution.

It is possible to define a throwable that is not a subclass of Exception, RuntimeException, or Error. The JLS does not address such throwables directly but
specifies implicitly that they are behaviorally identical to ordinary checked exceptions (which are subclasses of Exception but not RuntimeException).
It has no benefits over an ordinary checked exception and would merely serve to confuse the user of your API.

To summarize, use checked exceptions for recoverable conditions and runtime exceptions for programming errors. Of course, the situation is not always black
and white. For example, consider the case of resource exhaustion, which can be caused by a programming error such as allocating an unreasonably large array or
by a genuine shortage of resources. If resource exhaustion is caused by a temporary shortage or by temporarily heightened demand, the condition may well be
recoverable. It is a matter of judgment on the part of the API designer whether a given instance of resource exhaustion is likely to allow for recovery. If you believe a condition is likely to allow for recovery, use a checked exception; if not, use a runtime exception.

Because checked exceptions generally indicate recoverable conditions, it’s especially important for such exceptions to provide methods that furnish information
that could help the caller to recover. For example, suppose a checked exception is thrown when an attempt to make a purchase with a gift card fails because
the card doesn’t have enough money left on it. The exception should provide an accessor method to query the amount of the shortfall, so the amount can be
relayed to the shopper.

Item-59 - Avoid unnecessary use of checked exceptions
Checked exceptions are a wonderful feature of the Java programming language. Unlike return codes, they force the programmer to deal with exceptional conditions,
greatly enhancing reliability. That said, overuse of checked exceptions can make an API far less pleasant to use.

As a litmus test, ask yourself how the programmer will handle the exception. Is this the best that can be done?
} catch(TheCheckedException e) {
throw new AssertionError(); // Can't happen!
}
How about this?
} catch(TheCheckedException e) {
e.printStackTrace(); // Oh well, we lose.
System.exit(1);
}

If the programmer using the API can do no better, an unchecked exception would be more appropriate. One example of an exception that fails this test is
CloneNotSupportedException. It is thrown by Object.clone, which should be invoked only on objects that implement Cloneable (Item 11). In practice, the
catch block almost always has the character of an assertion failure. The checked nature of the exception provides no benefit to the programmer.

One technique for turning a checked exception into an unchecked exception is to break the method that throws the exception into two methods, the first of which
returns a boolean that indicates whether the exception would be thrown. This API refactoring transforms the calling sequence from this:
// Invocation with checked exception
try {
obj.action(args);
} catch(TheCheckedException e) {
// Handle exceptional condition
...
}
to this:
// Invocation with state-testing method and unchecked exception
if (obj.actionPermitted(args)) {
obj.action(args);
} else {
// Handle exceptional condition
...
}

The API resulting from this refactoring is essentially identical to the state-testing method API in Item 57 and the same caveats apply: if an object is to be accessed concurrently without external synchronization or it is subject to externally induced state transitions, this refactoring is inappropriate, 
as the object’s state may change between the invocations of actionPermitted and action.

Item-60 - Favor the use of standard exceptions
IllegalArgumentException - Non-null parameter value is inappropriate
IllegalStateException -  Object state is inappropriate for method invocation
NullPointerException - Parameter value is null where prohibited
IndexOutOfBoundsException - Index parameter value is out of range
ConcurrentModificationException - Concurrent modification of an object has been detected where it is prohibited
UnsupportedOperationException - Object does not support method

Item-61 - Throw exceptions appropriate to the abstraction
It is disconcerting when a method throws an exception that has no apparent connection to the task that it performs. This often happens when a method propagates an exception thrown by a lower-level abstraction. Not only is this disconcerting, but it pollutes the API of the higher layer with implementation details. If the implementation of the higher layer changes in a subsequent release, the exceptions that it throws will change too, potentially breaking existing client programs.

To avoid this problem, higher layers should catch lower-level exceptions and, in their place, throw exceptions that can be explained in terms of the
higher-level abstraction. This idiom is known as exception translation:
// Exception Translation
try {
// Use lower-level abstraction to do our bidding
...
} catch(LowerLevelException e) {
throw new HigherLevelException(...);
}

A special form of exception translation called exception chaining is appropriate in cases where the lower-level exception might be helpful to someone debugging
the problem that caused the higher-level exception. The lower-level exception (the cause) is passed to the higher-level exception, which provides an
accessor method (Throwable.getCause) to retrieve the lower-level exception: Most standard exceptions have chaining-aware constructors. For exceptions that
don’t, you can set the cause using Throwable’s initCause method
// Exception Chaining
try {
... // Use lower-level abstraction to do our bidding
} catch (LowerLevelException cause) {
throw new HigherLevelException(cause);
}

While exception translation is superior to mindless propagation of exceptions from lower layers, it should not be overused.
If it is impossible to prevent exceptions from lower layers, the next best thing is to have the higher layer silently work around these exceptions, insulating the caller of the higher-level method from lower-level problems. Under these circumstances, it may be appropriate to log the exception using some appropriate logging facility such as java.util.logging. This allows an administrator to investigate the problem, while insulating the client code and the end user from it.

Item-62 - Document all exceptions thrown by each method
Always declare checked exceptions individually, and document precisely the conditions under which each one is thrown using the Javadoc @throws tag.
As an extreme example, never declare that a method “throws Exception” or, worse yet, “throws Throwable.”

Use the Javadoc @throws tag to document each unchecked exception that a method can throw, but do not use the throws keyword to include unchecked exceptions in the method declaration.

It should be noted that documenting all of the unchecked exceptions that each method can throw is an ideal, not always achievable in the real world. When a
class undergoes revision, it is not a violation of source or binary compatibility if an exported method is modified to throw additional unchecked exceptions.

If an exception is thrown by many methods in a class for the same reason, it is acceptable to document the exception in the class’s documentation comment
rather than documenting it individually for each method.

Item-63 - Include failure-capture information in detail messages
When a program fails due to an uncaught exception, the system automatically prints out the exception’s stack trace. The stack trace contains the exception’s string representation, the result of invoking its toString method. This typically consists of the exception’s class name followed by its detail message. Frequently this is the only information that programmers or field service personnel will have when investigating a software failure . If the failure is not easily reproducible, it may be difficult or impossible to get any more information. Therefore, it is critically important that the exception’s toString method return as much information as possible concerning the cause of the failure.

To capture the failure, the detail message of an exception should contain the values of all parameters and fields that “contributed to the exception.”
For example, the detail message of an IndexOutOfBoundsException should contain the lower bound, the upper bound, and the index value that failed to lie
between the bounds. This information tells a lot about the failure. Any or all of the three values could be wrong.

One way to ensure that exceptions contain adequate failure-capture information in their detail messages is to require this information in their constructors
instead of a string detail message. The detail message can then be generated auto matically to include the information. For example, instead of a String constructor, IndexOutOfBoundsException could have had a constructor that looks like this:

public IndexOutOfBoundsException(int lowerBound, int upperBound,
int index) {
// Generate a detail message that captures the failure
super("Lower bound: " + lowerBound +
", Upper bound: " + upperBound +
", Index: " + index);
// Save failure information for programmatic access
this.lowerBound = lowerBound;
this.upperBound = upperBound;
this.index = index;
}

Unfortunately, the Java platform libraries do not make heavy use of this idiom, but it is highly recommended.

As suggested in Item 58, it may be appropriate for an exception to provide accessor methods for its failure-capture information (lowerBound, upperBound,
and index in the above example). It is more important to provide such accessor methods on checked exceptions than on unchecked exceptions, because the failure-
capture information could be useful in recovering from the failur.

Item-64 - Strive for failure atomicity
After an object throws an exception, it is generally desirable that the object still be in  a well-defined, usable state, even if the failure occurred in the midst of performing an operation. This is especially true for checked exceptions, from which the caller is expected to recover. Generally speaking, a failed method invocation should leave the object in the state that it was in prior to the invocation. A method with this property is said to be failure atomic.

There are several ways to achieve this effect. The simplest is to design immutable objects (Item 15). If an object is immutable, failure atomicity is free. If an operation fails, it may prevent a new object from getting created, but it will never leave an existing object in an inconsistent state, because the state of each object is consistent when it is created and can’t be modified thereafter.

For methods that operate on mutable objects, the most common way to achieve failure atomicity is to check parameters for validity before performing the
operation (Item 38).
public Object pop() {
if (size == 0)
throw new EmptyStackException();
Object result = elements[--size];
elements[size] = null; // Eliminate obsolete reference
return result;
}
If the initial size check were eliminated, the method would still throw an exception when it attempted to pop an element from an empty stack. It would,
however, leave the size field in an inconsistent (negative) state.

A closely related approach to achieving failure atomicity is to order the computation so that any part that may fail takes place before any part that modifies the object.

A third and far less common approach to achieving failure atomicity is to write recovery code that intercepts a failure that occurs in the midst of an operation
and causes the object to roll back its state to the point before the operation began. This approach is used mainly for durable (disk-based) data structures.

A final approach to achieving failure atomicity is to perform the operation on a temporary copy of the object and to replace the contents of the object with the
temporary copy once the operation is complete. This approach occurs naturally when the computation can be performed more quickly once the data has been
stored in a temporary data structure. For example, Collections.sort dumps its input list into an array prior to sorting to reduce the cost of accessing elements in the inner loop of the sort. This is done for performance, but as an added benefit, it ensures that the input list will be untouched if the sort fails.

Item-65 - Don’t ignore exceptions
// Empty catch block ignores exception - Highly suspect!
try {
...
} catch (SomeException e) {
}
An empty catch block defeats the purpose of exceptions, which is to force you to handle exceptional conditions. Ignoring an exception is analogous to ignoring
a fire alarm  

At the very least, the catch block should contain a comment explaining why it is appropriate to ignore the exception.
An example of the sort of situation where it might be appropriate to ignore an exception is when closing a FileInputStream. You haven’t changed the state of
the file, so there’s no need to perform any recovery action, and you’ve already read the information that you need from the file, so there’s no reason to abort the operation in progress. Even in this case, it is wise to log the exception, so that you can investigate the matter if these exceptions happen often.

==================================================================================================================

=====================================Concurrency=============================================================================
Item-66 - Synchronize access to shared mutable data
The synchronized keyword ensures that only a single thread can execute a method or block at one time. Many programmers think of synchronization solely as a
means of mutual exclusion, to prevent an object from being observed in an inconsistent state while it’s being modified by another thread. In this view, an object is created in a consistent state (Item 15) and locked by the methods that access it. These methods observe the state and optionally cause a state transition, transforming the object from one consistent state to another. Proper use of synchronization guarantees that no method will ever observe the object in an inconsistent state.

The language specification guarantees that reading or writing a variable is atomic unless the variable is of type long or double [JLS, 17.4.7]. In other words,
reading a variable other than a long or double is guaranteed to return a value that was stored into that variable by some thread, even if multiple threads modify the variable concurrently and without synchronization. 

You may hear it said that to improve performance, you should avoid synchronization when reading or writing atomic data. This advice is dangerously wrong.
While the language specification guarantees that a thread will not see an arbitrary value when reading a field, it does not guarantee that a value written by one thread will be visible to another. Synchronization is required for reliable communication between threads as well as for mutual exclusion.

The consequences of failing to synchronize access to shared mutable data can be dire even if the data is atomically readable and writable. Consider the task of
stopping one thread from another. The libraries provide the Thread.stop method, but this method was deprecated long ago because it is inherently unsafe—its use
can result in data corruption. Do not use Thread.stop. A recommended way to stop one thread from another is to have the first thread poll a boolean field that is initially false but can be set to true by the second thread to indicate that the first thread is to stop itself. Because reading and writing a boolean field is atomic, some programmers dispense with synchronization when accessing the field:
// Broken! - How long would you expect this program to run?
public class StopThread {
private static boolean stopRequested;
public static void main(String[] args)
throws InterruptedException {
Thread backgroundThread = new Thread(new Runnable() {
public void run() {
int i = 0;
while (!stopRequested)
i++;
}
});
backgroundThread.start();
TimeUnit.SECONDS.sleep(1);
stopRequested = true;
}
}
The problem is that in the absence of synchronization, there is no guarantee as to when, if ever, the background thread will see the change in the value of stop- Requested that was made by the main thread

One way to fix the problem is to synchronize access to the stopRequested field. This program terminates in about one second, as expected:
// Properly synchronized cooperative thread termination
public class StopThread {
private static boolean stopRequested;
private static synchronized void requestStop() {
stopRequested = true;
}
private static synchronized boolean stopRequested() {
return stopRequested;
}
public static void main(String[] args)
throws InterruptedException {
Thread backgroundThread = new Thread(new Runnable() {
public void run() {
int i = 0;
while (!stopRequested())
i++;
}
});
backgroundThread.start();
TimeUnit.SECONDS.sleep(1);
requestStop();
}
} 
synchronization has no effect unless both read and write operations are synchronized.
the synchronization on these methods is used solely for its communication effects, not for mutual exclusion. While the cost of synchronizing on each iteration of the loop is small, there is a correct alternative that is less verbose and whose performance is likely to be better. The locking in the second version of StopThread can be omitted if stopRequested is declared volatile. While the volatile modifier performs no mutual exclusion, it guarantees that any thread that reads the field will see the most recently written value:
// Cooperative thread termination with a volatile field
public class StopThread {
private static volatile boolean stopRequested;
public static void main(String[] args)
throws InterruptedException {
Thread backgroundThread = new Thread(new Runnable() {
public void run() {
int i = 0;
while (!stopRequested)
i++;
}
});
backgroundThread.start();
TimeUnit.SECONDS.sleep(1);
stopRequested = true;
}
}

You do have to be careful when using volatile. Consider the following method, which is supposed to generate serial numbers:
// Broken - requires synchronization!
private static volatile int nextSerialNumber = 0;
public static int generateSerialNumber() {
return nextSerialNumber++;
} // the method won’t work properly without synchronization
The problem is that the increment operator (++) is not atomic. It performs two operations on the nextSerialNumber field: first it reads the value, then it writes back a new value, equal to the old value plus one. If a second thread reads the field between the time a thread reads the old value and writes back a new one, the second thread will see the same value as the first and return the same serial number. This is a safety failure: the program computes the wrong results.
One way to fix the generateSerialNumber method is to add the synchronized modifier to its declaration. This ensures that multiple invocations won’t be
interleaved, and that each invocation will see the effects of all previous invocations. 

Better still, follow the advice in Item 47 and use the class AtomicLong, which is part of java.util.concurrent.atomic.
private static final AtomicLong nextSerialNum = new AtomicLong();
public static long generateSerialNumber() {
return nextSerialNum.getAndIncrement();
}

Item-67 - Avoid excessive synchronization
To avoid liveness and safety failures, never cede control to the client within a synchronized method or block. In other words, inside a synchronized
region, do not invoke a method that is designed to be overridden, or one provided by a client in the form of a function object (Item 21). From the perspective of the class with the synchronized region, such methods are alien. 

To make this concrete, consider the following class, which implements an observable set wrapper. It allows clients to subscribe to notifications when elements
are added to the set. This is the Observer pattern.
// Broken - invokes alien method from synchronized block!
public class ObservableSet<E> extends ForwardingSet<E> {
public ObservableSet(Set<E> set) { super(set); }
private final List<SetObserver<E>> observers =
new ArrayList<SetObserver<E>>();
public void addObserver(SetObserver<E> observer) {
synchronized(observers) {
observers.add(observer);
}
}
public boolean removeObserver(SetObserver<E> observer) {
synchronized(observers) {
return observers.remove(observer);
}
}
private void notifyElementAdded(E element) {
synchronized(observers) {
for (SetObserver<E> observer : observers)
observer.added(this, element);
}
}
@Override public boolean add(E element) {
boolean added = super.add(element);
if (added)
notifyElementAdded(element);
return added;
}
@Override public boolean addAll(Collection<? extends E> c) {
boolean result = false;
for (E element : c)
result |= add(element); // calls notifyElementAdded
return result;
}
}

//program prints the numbers from 0 through 99:
public static void main(String[] args) {
ObservableSet<Integer> set =
new ObservableSet<Integer>(new HashSet<Integer>());
set.addObserver(new SetObserver<Integer>() {
public void added(ObservableSet<Integer> s, Integer e) {
System.out.println(e);
}
});
for (int i = 0; i < 100; i++)
set.add(i);
}

Now let’s try something a bit fancier
set.addObserver(new SetObserver<Integer>() {
public void added(ObservableSet<Integer> s, Integer e) {
System.out.println(e);
if (e == 23) s.removeObserver(this);
}
});

You might expect the program to print the numbers 0 through 23, What actually happens is that it prints the numbers 0 through 23, and then throws a ConcurrentModificationException. The problem is that notifyElementAdded is in the process of iterating over the observers list when it invokes the observer’s
added method. The added method calls the observable set’s removeObserver method, which in turn calls observers.remove. Now we are in trouble. We are
trying to remove an element from a list in the midst of iterating over it, which is illegal.

Now let’s try something odd: let’s write an observer that attempts to unsubscribe,
but instead of calling removeObserver directly, it engages the services of
another thread to do the deed.

// Observer that uses a background thread needlessly
set.addObserver(new SetObserver<Integer>() {
public void added(final ObservableSet<Integer> s, Integer e) {
System.out.println(e);
if (e == 23) {
ExecutorService executor =
Executors.newSingleThreadExecutor();
final SetObserver<Integer> observer = this;
try {
executor.submit(new Runnable() {
public void run() {
s.removeObserver(observer);
}
}).get(); // get blocks the executer spawned thread 
} catch (ExecutionException ex) {
throw new AssertionError(ex.getCause());
} catch (InterruptedException ex) {
throw new AssertionError(ex.getCause());
} finally {
executor.shutdown();
}
}
}
});

This time we don’t get an exception; we get a deadlock. The background thread calls s.removeObserver, which attempts to lock observers, but it can’t acquire
the lock, because the main thread already has the lock. All the while, the main thread is waiting for the background thread to finish removing the observer, which explains the deadlock.

In both of the previous examples (the exception and the deadlock) we were lucky. The resource that was guarded by the synchronized region (observers)
was in a consistent state when the alien method (added) was invoked. Suppose you were to invoke an alien method from within a synchronized region while the
invariant protected by the synchronized region was temporarily invalid. Because locks in the Java programming language are reentrant, such calls won’t deadlock.
As in the first example, which resulted in an exception, the calling thread already holds the lock, so the thread will succeed when it tries to reacquire the lock, even though another conceptually unrelated operation is in progress on the data guarded by the lock. The consequences of such a failure can be catastrophic. In essence, the lock has failed to do its job. Reentrant locks simplify the construction of multithreaded object-oriented programs, but they can turn liveness failures into safety failures.

Luckily, it is usually not too hard to fix this sort of problem by moving alien method invocations out of synchronized blocks. For the notifyElementAdded
method, this involves taking a “snapshot” of the observers list that can then be safely traversed without a lock.

// Alien method moved outside of synchronized block - open calls
private void notifyElementAdded(E element) {
List<SetObserver<E>> snapshot = null;
synchronized(observers) {
snapshot = new ArrayList<SetObserver<E>>(observers);
}
for (SetObserver<E> observer : snapshot)
observer.added(this, element);
}

In fact, there’s a better way to move the alien method invocations out of the synchronized block. Since release 1.5, the Java libraries have provided a concurrent collection (Item 69) known as CopyOnWriteArrayList, which is tailor-made for this purpose. It is a variant of ArrayList in which all write operations are implemented by making a fresh copy of the entire underlying array. Because the internal array is never modified, iteration requires no locking and is very fast.

// Thread-safe observable set with CopyOnWriteArrayList
private final List<SetObserver<E>> observers =
new CopyOnWriteArrayList<SetObserver<E>>();
public void addObserver(SetObserver<E> observer) {
observers.add(observer);
}
public boolean removeObserver(SetObserver<E> observer) {
return observers.remove(observer);
}
private void notifyElementAdded(E element) {
for (SetObserver<E> observer : observers)
observer.added(this, element);
}

An alien method invoked outside of a synchronized region is known as an open call [Lea00 2.4.1.3]. Besides preventing failures, open calls can greatly
increase concurrency. An alien method might run for an arbitrarily long period. If the alien method were invoked from a synchronized region, other threads would
be denied access to the protected resource unnecessarily. As a rule, you should do as little work as possible inside synchronized regions. Obtain the lock, examine the shared data, transform it as necessary, and drop the lock.

The first part of this item was about correctness. Now let’s take a brief look at performance. While the cost of synchronization has plummeted since the early
days of Java, it is more important than ever not to oversynchronize. In a multicore world, the real cost of excessive synchronization is not the CPU time spent obtaining locks; it is the lost opportunities for parallelism and the delays imposed by the need to ensure that every core has a consistent view of memory. Another hidden cost of oversynchronization is that it can limit the VM’s ability to optimize code execution.

You should make a mutable class thread-safe (Item 70) if it is intended for concurrent use and you can achieve significantly higher concurrency by synchronizing
internally than you could by locking the entire object externally. Otherwise, don’t synchronize internally. Let the client synchronize externally where it is
appropriate. 

If you do synchronize your class internally, you can use various techniques to achieve high concurrency, such as lock splitting, lock striping, and nonblocking
concurrency control. These techniques are beyond the scope of this book, but they are discussed elsewhere.

Item-68 - Prefer executors and tasks to threads
In release 1.5, java.util.concurrent was added to the Java platform. This package contains an Executor Framework, which is a flexible interface-based task
execution facility
ExecutorService executor = Executors.newSingleThreadExecutor();
executor.execute(runnable);
executor.shutdown();

You can do many more things with an executor service. For example, you can wait for a particular task to complete (as in the “background thread SetObserver”
in Item 67, page 267), you can wait for any or all of a collection of tasks to complete (using the invokeAny or invokeAll methods), you can wait for the executor service’s graceful termination to complete (using the awaitTermination method), you can retrieve the results of tasks one by one as they complete (using an ExecutorCompletionService), and so on.

Choosing the executor service for a particular application can be tricky. If you’re writing a small program, or a lightly loaded server, using Executors.new-
CachedThreadPool is generally a good choice, as it demands no configuration and generally “does the right thing.” But a cached thread pool is not a good choice
for a heavily loaded production server! In a cached thread pool, submitted tasks are not queued but immediately handed off to a thread for execution. If no threads are available, a new one is created. If a server is so heavily loaded that all of its CPUs are fully utilized, and more tasks arrive, more threads will be created, which will only make matters worse. Therefore, in a heavily loaded production server, you are much better off using Executors.newFixedThreadPool, which gives you a pool with a fixed number of threads, or using the ThreadPoolExecutor class directly, for maximum control.

Not only should you refrain from writing your own work queues, but you should generally refrain from working directly with threads. The key abstraction
is no longer Thread, which served as both the unit of work and the mechanism for executing it. Now the unit of work and mechanism are separate. The key abstraction is the unit of work, which is called a task. There are two kinds of tasks: Runnable and its close cousin, Callable (which is like Runnable, except that it returns a value).  In essence, the Executor Framework does for execution what the Collections Framework did for aggregation.

The Executor Framework also has a replacement for java.util.Timer, which is ScheduledThreadPoolExecutor. While it is easier to use a timer, a
scheduled thread pool executor is much more flexible. A timer uses only a single thread for task execution, which can hurt timing accuracy in the presence of longrunning tasks. If a timer’s sole thread throws an uncaught exception, the timer ceases to operate. A scheduled thread pool executor supports multiple threads and recovers gracefully from tasks that throw unchecked exceptions.
Read Java Concurrency in Practice for complete understanding.

Item-69 - Prefer concurrency utilities to wait and notify
As of release 1.5, the Java platform provides higher-level concurrency utilities that do the sorts of things you formerly had to hand-code atop wait and notify. Given the difficulty of using wait and notify correctly, you should use the higher-level concurrency utilities instead.

The higher-level utilities in java.util.concurrent fall into three categories: the Executor Framework, concurrent collections and synchronizers

The concurrent collections provide high-performance concurrent implementations of standard collection interfaces such as List, Queue, and Map. To provide
high concurrency, these implementations manage their own synchronization internally.
clients can’t atomically compose method invocations on concurrent collections. Some of the collection interfaces have therefore been extended with state-dependent modify operations, which combine several primitives into a single atomic operation.

Some of the collection interfaces have been extended with blocking operations, which wait (or block) until they can be successfully performed. For example,
BlockingQueue extends Queue and adds several methods, including take, which removes and returns the head element from the queue, waiting if the queue
is empty. This allows blocking queues to be used for work queues (also known as producer-consumer queues), to which one or more producer threads enqueue
work items and from which one or more consumer threads dequeue and process items as they become available. As you’d expect, most ExecutorService implementations, including ThreadPoolExecutor, use a BlockingQueue.

Synchronizers are objects that enable threads to wait for one another, allowing them to coordinate their activities. The most commonly used synchronizers are
CountDownLatch and Semaphore. Less commonly used are CyclicBarrier and Exchanger.
Countdown latches are single-use barriers that allow one or more threads to wait for one or more other threads to do something. The sole constructor for
CountDownLatch takes an int that is the number of times the countDown method must be invoked on the latch before all waiting threads are allowed to proceed.

// Simple framework for timing concurrent execution
public static long time(Executor executor, int concurrency, final Runnable action) throws InterruptedException {
final CountDownLatch ready = new CountDownLatch(concurrency);
final CountDownLatch start = new CountDownLatch(1);
final CountDownLatch done = new CountDownLatch(concurrency);
for (int i = 0; i < concurrency; i++) {
executor.execute(new Runnable() {
public void run() {
ready.countDown(); // Tell timer we're ready
try {
start.await(); // Wait till peers are ready
action.run();
} catch (InterruptedException e) {
Thread.currentThread().interrupt();
} finally {
done.countDown(); // Tell timer we're done
}
}
});
}
ready.await(); // Wait for all workers to be ready
long startNanos = System.nanoTime();
start.countDown(); // And they're off!
done.await(); // Wait for all workers to finish
return System.nanoTime() - startNanos;
}

For interval timing, always use System.nanoTime in preference to System.currentTime- Millis. System.nanoTime is both more accurate and more precise.

// The standard idiom for using the wait method
synchronized (obj) {
while (<condition does not hold>)
obj.wait(); // (Releases lock, and reacquires on wakeup)
... // Perform action appropriate to condition
}

Always use the wait loop idiom to invoke the wait method; never invoke it outside of a loop. The loop serves to test the condition before and after waiting.

Item-70 - Document thread safety
How a class behaves when its instances or static methods are subjected to concurrent use is an important part of the contract the class makes with its clients. If you don’t document this facet of a class’s behavior, programmers who use the class will be forced to make assumptions.
You might hear it said that you can tell if a method is thread-safe by looking for the synchronized modifier in its documentation. This is wrong on several
counts. In normal operation, Javadoc does not include the synchronized modifier in its output.

In fact, there are several levels of thread safety. To enable safe concurrent use, a class must clearly document what level of thread safety it supports.
immutable—Instances of this class appear constant. No external synchronization is necessary. Examples include String, Long, and BigInteger
unconditionally thread-safe—Instances of this class are mutable, but the class has sufficient internal synchronization that its instances can be used
concurrently without the need for any external synchronization. Examples include Random and ConcurrentHashMap.
conditionally thread-safe—Like unconditionally thread-safe, except that some methods require external synchronization for safe concurrent use. Examples include the collections returned by the Collections.synchronized wrappers, whose iterators require external synchronization
not thread-safe—Instances of this class are mutable. To use them concurrently, clients must surround each method invocation.
thread-hostile—This class is not safe for concurrent use even if all method invocations are surrounded by external synchronization. The System.runFinalizersOnExit method is thread-hostile and has been deprecated

Item-71 - Use lazy initialization judiciously
Lazy initialization is the act of delaying the initialization of a field until its value is needed. If the value is never needed, the field is never initialized. This technique is applicable to both static and instance fields.

Under most circumstances, normal initialization is preferable to lazy initialization.

If you use lazy initialization to break an initialization circularity, use a synchronized accessor, as it is the simplest, clearest alternative:
// Lazy initialization of instance field - synchronized accessor
private FieldType field;
synchronized FieldType getField() {
if (field == null)
field = computeFieldValue();
return field;
}

If you need to use lazy initialization for performance on a static field, use the lazy initialization holder class idiom. This idiom (also known as the initializeon- demand holder class idiom) exploits the guarantee that a class will not be initialized until it is used

private static class FieldHolder {
static final FieldType field = computeFieldValue();
}
static FieldType getField() { return FieldHolder.field; }

If you need to use lazy initialization for performance on an instance field, use the double-check idiom. This idiom avoids the cost of locking when accessing
the field after it has been initialized.
// Double-check idiom for lazy initialization of instance fields
private volatile FieldType field;
FieldType getField() {
FieldType result = field;
if (result == null) { // First check (no locking)
synchronized(this) {
result = field;
if (result == null) // Second check (with locking)
field = result = computeFieldValue();
}
}
return result;
}

Occasionally, you may need to lazily initialize an instance field that can tolerate repeated initialization. If you find yourself in this situation, you can use a variant of the double-check idiom that dispenses with the second check. It is, not surprisingly, known as the singlecheck idiom
// Single-check idiom - can cause repeated initialization!
private volatile FieldType field;
private FieldType getField() {
FieldType result = field;
if (result == null)
field = result = computeFieldValue();
return result;
}

Item-72 - Don’t depend on the thread scheduler
When many threads are runnable, the thread scheduler determines which ones get to run, and for how long. Any reasonable operating system will try to make this
determination fairly, but the policy can vary. Therefore, well-written programs shouldn’t depend on the details of this policy. Any program that relies on the
thread scheduler for correctness or performance is likely to be nonportable.

Note that the number of runnable threads isn’t the same as the total number of threads, which can be much higher. Threads that are waiting are not runnable.
The best way to write a robust, responsive, portable program is to ensure that the average number of runnable threads is not significantly greater than the number of processors. This leaves the thread scheduler with little choice: it simply runs the runnable threads till they’re no longer runnable.

The main technique for keeping the number of runnable threads down is to have each thread do some useful work and then wait for more. Threads should
not run if they aren’t doing useful work. In terms of the Executor Framework (Item 68), this means sizing your thread pools appropriately.

Threads should not busy-wait, repeatedly checking a shared object waiting for something to happen. Besides making the program vulnerable to the vagaries of
the scheduler, busy-waiting greatly increases the load on the processor, reducing the amount of useful work that others can accomplish.
// Awful CountDownLatch implementation - busy-waits incessantly!
public class SlowCountDownLatch {
private int count;
public SlowCountDownLatch(int count) {
if (count < 0)
throw new IllegalArgumentException(count + " < 0");
this.count = count;
}
public void await() {
while (true) {
synchronized(this) {
if (count == 0) return;
}
}
}
public synchronized void countDown() {
if (count != 0)
count--;
}
}

When faced with a program that barely works because some threads aren’t getting enough CPU time relative to others, resist the temptation to “fix” the
program by putting in calls to Thread.yield. You may succeed in getting the program to work after a fashion, but it will not be portable.

A related technique, to which similar caveats apply, is adjusting thread priorities. Thread priorities are among the least portable features of the Java platform.

Item-73 - Avoid thread groups
Along with threads, locks, and monitors, a basic abstraction offered by the threading system is thread groups. Thread groups were originally envisioned as a mechanism for isolating applets for security purposes. They never really fulfilled this.

In an ironic twist, the ThreadGroup API is weak from a thread safety standpoint. To get a list of the active threads in a thread group, you must invoke
the enumerate method, which takes as a parameter an array large enough to hold all the active threads. The activeCount method returns the number of active
threads in a thread group, but there is no guarantee that this count will still be accurate once an array has been allocated and passed to the enumerate method. If the thread count has increased and the array is too small, the enumerate method silently ignores any threads for which there is no room in the array.

If you design a class that deals with logical groups of threads, you should probably use thread pool executors as thread groups are obsolete.
==================================================================================================================

======================================Serialization============================================================================
Item-74 - Implement Serializable judiciously
While the immediate cost to make a class serializable can be negligible, the long-term costs are often substantial.

A major cost of implementing Serializable is that it decreases the flexibility to change a class’s implementation once it has been released.
When a class implements Serializable, its byte-stream encoding (or serialized form) becomes part of its exported API. Once you distribute a class widely, you are generally required to support the serialized form forever, just as you are required to support all other parts of the exported API.

If you accept the default serialized form and later change the class’s internal representation, an incompatible change in the serialized form might result. Clients attempting to serialize an instance using an old version of the class and deserialize it using the new version will experience program failures.
A simple example of the constraints on evolution that accompany serializability concerns stream unique identifiers, more commonly known as serial version 
UIDs.

A second cost of implementing Serializable is that it increases the likelihood of bugs and security holes. Normally, objects are created using constructors;
serialization is an extralinguistic mechanism for creating objects.

A third cost of implementing Serializable is that it increases the testing burden associated with releasing a new version of a class. When a serializable
class is revised, it is important to check that it is possible to serialize an instance in the new release and deserialize it in old releases, and vice versa.

Classes designed for inheritance (Item 17) should rarely implement Serializable, and interfaces should rarely extend it.
Classes designed for inheritance that do implement Serializable include Throwable, Component, and HttpServlet. Throwable implements Serializable
so exceptions from remote method invocation (RMI) can be passed from server to client. Component implements Serializable so GUIs can be sent, saved, and
restored. HttpServlet implements Serializable so session state can be cached.

There is one caveat regarding the decision not to implement Serializable. If a class that is designed for inheritance is not serializable, it may be impossible to write a serializable subclass. Specifically, it will be impossible if the superclass does not provide an accessible parameterless constructor. Therefore, you should consider providing a parameterless constructor on nonserializable classes designed for inheritance.

Here is a way to add a parameterless constructor to a nonserializable extendable class that avoids these deficiencies.
// Nonserializable stateful class allowing serializable subclass
public abstract class AbstractFoo {
private int x, y; // Our state
// This enum and field are used to track initialization
private enum State { NEW, INITIALIZING, INITIALIZED };
private final AtomicReference<State> init = new AtomicReference<State>(State.NEW);
public AbstractFoo(int x, int y) { initialize(x, y); }
// This constructor and the following method allow
// subclass's readObject method to initialize our state.
protected AbstractFoo() { }

protected final void initialize(int x, int y) {
if (!init.compareAndSet(State.NEW, State.INITIALIZING))
throw new IllegalStateException(
"Already initialized");
this.x = x;
this.y = y;
... // Do anything else the original constructor did
init.set(State.INITIALIZED);
}
// These methods provide access to internal state so it can be manually serialized by subclass's writeObject method.
protected final int getX() { checkInit(); return x; }
protected final int getY() { checkInit(); return y; }
// Must call from all public and protected instance methods
private void checkInit() {
if (init.get() != State.INITIALIZED)
throw new IllegalStateException("Uninitialized");
}
... // Remainder omitted
}

All public and protected instance methods in AbstractFoo must invoke checkInit before doing anything else. This ensures that method invocations fail
quickly and cleanly if a poorly written subclass fails to initialize an instance. Note that the initialized field is an atomic reference (java.util.concurrent.
atomic.AtomicReference). This is necessary to ensure object integrity in the face of a determined adversary. In the absence of this precaution, if one thread
were to invoke initialize on an instance while a second thread attempted to use it, the second thread might see the instance in an inconsistent state.

// Serializable subclass of nonserializable stateful class
public class Foo extends AbstractFoo implements Serializable {
private void readObject(ObjectInputStream s)
throws IOException, ClassNotFoundException {
s.defaultReadObject();
// Manually deserialize and initialize superclass state
int x = s.readInt();
int y = s.readInt();
initialize(x, y);
}
private void writeObject(ObjectOutputStream s)
throws IOException {
s.defaultWriteObject();
// Manually serialize superclass state
s.writeInt(getX());
s.writeInt(getY());
}
// Constructor does not use the fancy mechanism
public Foo(int x, int y) { super(x, y); }
private static final long serialVersionUID = 1856835860954L;
}

Inner classes (Item 22) should not implement Serializable. They use compiler-generated synthetic fields to store references to enclosing instances and
to store values of local variables from enclosing scopes. How these fields correspond to the class definition is unspecified, as are the names of anonymous and
local classes. Therefore, the default serialized form of an inner class is illdefined. A static member class can, however, implement Serializable.

Item-75 - Consider using a custom serialized form
Do not accept the default serialized form without first considering whether it is appropriate. Generally speaking, you should accept the default
serialized form only if it is largely identical to the encoding that you would choose if you were designing a custom serialized form.
The default serialized form of an object is a reasonably efficient encoding of the physical representation of the object graph rooted at the object. In other words, it describes the data contained in the object and in every object that is reachable from this object.

The default serialized form is likely to be appropriate if an object’s physical representation is identical to its logical content. For example, the default
serialized form would be reasonable for the following class, which simplistically represents a person’s name:
// Good candidate for default serialized form
public class Name implements Serializable {

private final String lastName;
private final String firstName;
private final String middleName;
... // Remainder omitted
}

Even if you decide that the default serialized form is appropriate, you often must provide a readObject method to ensure invariants and security. In
the case of Name, the readObject method must ensure that lastName and first- Name are non-null.

consider the following class, which represents a list of strings (ignoring for the moment that you’d be better off using one of the standard List implementations):
// Awful candidate for default serialized form
public final class StringList implements Serializable {
private int size = 0;
private Entry head = null;
private static class Entry implements Serializable {
String data;
Entry next;
Entry previous;
}
... // Remainder omitted
}

Logically speaking, this class represents a sequence of strings. Physically, it represents the sequence as a doubly linked list. If you accept the default serialized form, the serialized form will painstakingly mirror every entry in the linked list and all the links between the entries, in both directions.

Using the default serialized form when an object’s physical representation differs substantially from its logical data content has four disadvantages:
1.It permanently ties the exported API to the current internal representation. In the above example, the private StringList.Entry class becomes part
of the public API. If the representation is changed in a future release, the StringList class will still need to accept the linked list representation on input
and generate it on output.
2.It can consume excessive space
3. It can consume excessive time. The serialization logic has no knowledge of the topology of the object graph, so it must go through an expensive graph
traversal.
4.It can cause stack overflows. The default serialization procedure performs a recursive traversal of the object graph, which can cause stack overflows

A reasonable serialized form for StringList is simply the number of strings in the list, followed by the strings themselves. This constitutes the logical data
represented by a StringList, stripped of the details of its physical representation 

// StringList with a reasonable custom serialized form
public final class StringList implements Serializable {
private transient int size = 0;
private transient Entry head = null;

// No longer Serializable!
private static class Entry {
String data;
Entry next;
Entry previous;
}
// Appends the specified string to the list
public final void add(String s) { ... }

private void writeObject(ObjectOutputStream s)
throws IOException {
s.defaultWriteObject();
s.writeInt(size);
// Write out all elements in the proper order.
for (Entry e = head; e != null; e = e.next)
s.writeObject(e.data);
}
private void readObject(ObjectInputStream s)
throws IOException, ClassNotFoundException {
s.defaultReadObject();
int numElements = s.readInt();
// Read in all elements and insert them in list
for (int i = 0; i < numElements; i++)
add((String) s.readObject());
}
}

Note that the first thing writeObject does is to invoke defaultWriteObject, and the first thing readObject does is to invoke defaultReadObject, even though
all of StringList’s fields are transient. If all instance fields are transient, it is technically permissible to dispense with invoking defaultWriteObject and
defaultReadObject, but it is not recommended. Even if all instance fields are transient, invoking defaultWriteObject affects the serialized form, resulting in
greatly enhanced flexibility. The resulting serialized form makes it possible to add nontransient instance fields in a later release while preserving backward and forward compatibility. If an instance is serialized in a later version and deserialized in an earlier version, the added fields will be ignored. Had the earlier version’s readObject method failed to invoke defaultReadObject, the deserialization would fail with a StreamCorruptedException.

For example, consider the case of a hash table. The physical representation is a sequence of hash buckets containing key-value entries. The bucket that an entry
resides in is a function of the hash code of its key, which is not, in general, guaranteed to be the same from JVM implementation to JVM implementation. In fact, it isn’t even guaranteed to be the same from run to run. Therefore, accepting the default serialized form for a hash table would constitute a serious bug.

Whether or not you use the default serialized form, every instance field that is not labeled transient will be serialized when the defaultWriteObject method
is invoked. Therefore, every instance field that can be made transient should be made so.

Whether or not you use the default serialized form, you must impose any synchronization on object serialization that you would impose on any other
method that reads the entire state of the object.
// writeObject for synchronized class with default serialized form
private synchronized void writeObject(ObjectOutputStream s)
throws IOException {
s.defaultWriteObject();
}

Regardless of what serialized form you choose, declare an explicit serial version UID in every serializable class you write. This eliminates the serial version
UID as a potential source of incompatibility.

Item-76 - Write readObject methods defensively
The problem is that the readObject method is effectively another public constructor, and it demands all of the same care as any other constructor. Just as a
constructor must check its arguments for validity (Item 38) and make defensive copies of parameters where appropriate (Item 39), so must a readObject method.
If a readObject method fails to do either of these things, it is a relatively simple matter for an attacker to violate the class’s invariants.

Loosely speaking, readObject is a constructor that takes a byte stream as its sole parameter. In normal use, the byte stream is generated by serializing a normally constructed instance. The problem arises when readObject is presented with a byte stream that is artificially constructed to generate an object that violates the invariants of its class.

// readObject method with validity checking
private void readObject(ObjectInputStream s)
throws IOException, ClassNotFoundException {
s.defaultReadObject();
// Check that our invariants are satisfied
if (start.compareTo(end) > 0)
throw new InvalidObjectException(start +" after "+ end);
}

When an object is deserialized, it is critical to defensively copy any field containing an object reference that a client must not possess. Therefore, every serializable immutable class containing private mutable components must defensively copy these components in its readObject method.
private void readObject(ObjectInputStream s)
throws IOException, ClassNotFoundException {
s.defaultReadObject();
// Defensively copy our mutable components
start = new Date(start.getTime());
end = new Date(end.getTime());
// Check that our invariants are satisfied
if (start.compareTo(end) > 0)
throw new InvalidObjectException(start +" after "+ end);
}

To summarize, anytime you write a readObject method, adopt the mind-set that you are writing a public constructor that must produce a valid instance regardless
of what byte stream it is given. Here, in summary form, are the guidelines for writing a bulletproof readObject method:
•For classes with object reference fields that must remain private, defensively copy each object in such a field. Mutable components of immutable classes fall
into this category.
• Check any invariants and throw an InvalidObjectException if a check fails. The checks should follow any defensive copying. 
• If an entire object graph must be validated after it is deserialized, use the ObjectInputValidation interface [JavaSE6, Serialization].
• Do not invoke any overridable methods in the class, directly or indirectly

Item-77 - For instance control, prefer enum types to readResolve
Item 3 describes the Singleton pattern and gives the following example of a singleton class. This class restricts access to its constructor to ensure that only a single instance is ever created:
public class Elvis {
public static final Elvis INSTANCE = new Elvis();
private Elvis() { ... }
public void leaveTheBuilding() { ... }
}
As noted in Item 3, this class would no longer be a singleton if the words “implements Serializable” were added to its declaration. It doesn’t matter
whether the class uses the default serialized form or a custom serialized form (Item 75), nor does it matter whether the class provides an explicit readObject
method (Item 76). Any readObject method, whether explicit or default, returns a newly created instance, which will not be the same instance that was created at
class initialization time. 

The readResolve feature allows you to substitute another instance for the one created by readObject [Serialization, 3.7]. If the class of an object being deserialized defines a readResolve method with the proper declaration, this method is invoked on the newly created object after it is deserialized.

If the Elvis class is made to implement Serializable, the following readResolve method suffices to guarantee the singleton property:
// readResolve for instance control - you can do better!
private Object readResolve() {
// Return the one true Elvis and let the garbage collector
// take care of the Elvis impersonator.
return INSTANCE;
}

In fact, if you depend on readResolve for instance control, all instance fields with object reference types must be declared transient. Otherwise, it is possible for a determined attacker to secure a reference to the deserialized object before its readResolve method is run 

If instead you write your serializable instance-controlled class as an enum, you get an ironclad guarantee that there can be no instances besides the declared
constants. The JVM makes this guarantee, and you can depend on it. It requires no special care on your part
// Enum singleton - the preferred approach
public enum Elvis {
INSTANCE;
private String[] favoriteSongs =
{ "Hound Dog", "Heartbreak Hotel" };
public void printFavorites() {
System.out.println(Arrays.toString(favoriteSongs));
}
}

The accessibility of readResolve is significant. If you place a readResolve method on a final class, it should be private. If you place a readResolve method
on a nonfinal class, you must carefully consider its accessibility. If it is private, it will not apply to any subclasses. If it is package-private, it will apply only to subclasses in the same package. If it is protected or public, it will apply to all subclasses that do not override it. If a readResolve method is protected or public and a subclass does not override it, deserializing a serialized subclass instance will produce a superclass instance, which is likely to cause a ClassCastException.

Item-78 - Consider serialization proxies instead of serialized instances
As mentioned in Item 74 and discussed throughout this chapter, the decision to implement Serializable increases the likelihood of bugs and security problems,
because it causes instances to be created using an extralinguistic mechanism in place of ordinary constructors. There is, however, a technique that greatly reduces these risks. This technique is known as the serialization proxy pattern.

The serialization proxy pattern is reasonably straightforward. First, design a private static nested class of the serializable class that concisely represents the logical state of an instance of the enclosing class. This nested class, known as the serialization proxy, should have a single constructor, whose parameter type is the enclosing class. This constructor merely copies the data from its argument: it need not do any consistency checking or defensive copying. By design, the default serialized form of the serialization proxy is the perfect serialized form of the enclosing class. Both the enclosing class and its serialization proxy must be declared to implement Serializable.

// Serialization proxy for Period class
private static class SerializationProxy implements Serializable {
private final Date start;
private final Date end;
SerializationProxy(Period p) {
this.start = p.start;
this.end = p.end;
}
private static final long serialVersionUID =
234098243823485285L; // Any number will do (Item 75)
}

Next, add the following writeReplace method to the enclosing class
// writeReplace method for the serialization proxy pattern
private Object writeReplace() {
return new SerializationProxy(this);
}

The presence of this method causes the serialization system to emit a SerializationProxy instance instead of an instance of the enclosing class. In other words,
the writeReplace method translates an instance of the enclosing class to its serialization proxy prior to serialization.

With this writeReplace method in place, the serialization system will never generate a serialized instance of the enclosing class, but an attacker might fabricate one in an attempt to violate the class’s invariants. To guarantee that such an attack would fail, merely add this readObject method to the enclosing class: 
// readObject method for the serialization proxy pattern
private void readObject(ObjectInputStream stream)
throws InvalidObjectException {
throw new InvalidObjectException("Proxy required");
}

Finally, provide a readResolve method on the SerializationProxy class that returns a logically equivalent instance of the enclosing class. The presence of
this method causes the serialization system to translate the serialization proxy back into an instance of the enclosing class upon deserialization.

This readResolve method creates an instance of the enclosing class using only its public API, and therein lies the beauty of the pattern. It largely eliminates
the extralinguistic character of serialization, because the deserialized instance is created using the same constructors, static factories, and methods as any other instance.

// readResolve method for Period.SerializationProxy
private Object readResolve() {
return new Period(start, end); // Uses public constructor
}

The serialization proxy pattern has two limitations. It is not compatible with classes that are extendable by their clients (Item 17). Also, it is not compatible with some classes whose object graphs contain circularities:
==================================================================================================================



C:\Program Files\Java\jdk1.8.0_111\bin;%USERPROFILE%\AppData\Local\Microsoft\WindowsApps;
